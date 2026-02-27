# Distributed Meeting Creation and Event Processing with Fallbacks

Use this architecture for high-volume meeting creation with resilient event processing.

## Core Architectural Considerations

1. **Separation of planes**
- Command plane: REST meeting creation/update APIs.
- Event plane: webhook ingestion and async projection.

2. **Idempotency and dedupe**
- Require caller-provided idempotency key per create request.
- Dedupe webhook events by stable event key.

3. **Token isolation**
- Central token broker with distributed lock (Redis/Postgres advisory lock).

4. **Backpressure and queueing**
- Queue all webhook events and meeting commands.
- Use DLQ for poison messages.

5. **Fallback mechanisms**
- Retry with exponential backoff + jitter for retriable failures (`429/5xx/network`).
- Circuit breaker around Zoom API dependency.
- Reconciliation poller when webhook delivery is delayed/missed.

## Reference Topology

```text
API Gateway
  -> Meeting Command Service
      -> Idempotency Store (Redis/Postgres)
      -> Token Broker
      -> Zoom REST API
      -> Outbox/Event Bus

Webhook Ingress
  -> Signature Verify + URL Validation
  -> Queue (Kafka/SQS/Rabbit)
  -> Projection Workers
  -> Meeting State Store

Recovery Services
  -> Retry Worker
  -> Reconciliation Poller (REST pull)
  -> Dead Letter Reprocessor
```

## Retry + Circuit Breaker Example (TypeScript)

```ts
type RetryOptions = {
  retries: number;
  baseMs: number;
  maxMs: number;
};

function sleep(ms: number) {
  return new Promise((r) => setTimeout(r, ms));
}

function backoff(attempt: number, baseMs: number, maxMs: number) {
  const exp = Math.min(maxMs, baseMs * 2 ** attempt);
  const jitter = Math.floor(Math.random() * Math.min(250, exp / 4));
  return exp + jitter;
}

export async function retry<T>(fn: () => Promise<T>, opts: RetryOptions, isRetriable: (e: any) => boolean): Promise<T> {
  let lastErr: any;
  for (let i = 0; i <= opts.retries; i += 1) {
    try {
      return await fn();
    } catch (e) {
      lastErr = e;
      if (i === opts.retries || !isRetriable(e)) break;
      await sleep(backoff(i, opts.baseMs, opts.maxMs));
    }
  }
  throw lastErr;
}

export class CircuitBreaker {
  private failures = 0;
  private openUntil = 0;

  constructor(private threshold = 5, private coolDownMs = 15_000) {}

  canCall() {
    return Date.now() > this.openUntil;
  }

  recordSuccess() {
    this.failures = 0;
  }

  recordFailure() {
    this.failures += 1;
    if (this.failures >= this.threshold) {
      this.openUntil = Date.now() + this.coolDownMs;
    }
  }
}
```

## Distributed State Rules

- Meeting state is event-sourced or projection-based, not only request-response based.
- Persist `last_seen_event_ts` and status transitions to handle out-of-order events.
- Add monotonic transition guards (e.g., do not move `ended -> in_progress`).

## Fallback Matrix

| Failure | Primary response | Fallback |
|---|---|---|
| Token refresh failure | Retry token exchange | Fail fast + alert + pause new create requests |
| REST `429` / `5xx` | Retry w/ backoff | Queue command for delayed retry |
| Webhook verification failure | Reject `401` | Alert security pipeline |
| Webhook processor down | Buffer in queue | DLQ + replay job |
| Missing webhook event | Detect via reconciliation lag | REST poll and repair projection |
| Dependency outage | Open circuit breaker | Serve degraded status + queued commands |
