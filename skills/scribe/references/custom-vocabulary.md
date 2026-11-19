# Scribe Custom Vocabulary and Vocabulary Assets

Custom vocabulary improves recognition of names, acronyms, organizations, product names, and other domain-specific terms. Scribe supports two forms, and a request must use one form or the other, not both:

- `config.vocabulary.phrases` supplies an inline list for one Fast request, Batch job, or Live session.
- `config.vocabulary_id` references a persistent Vocabulary Asset.

Both forms accept at most 100 phrases.

## Inline Vocabulary

```json
{
  "config": {
    "language": "en-US",
    "vocabulary": {
      "phrases": ["Jane Chen", "Project Atlas", "Zoom AI Companion"]
    }
  }
}
```

The scope is the current Fast request, Batch job and all of its input files, or Live session. During a Live session, send another `session.update` to replace the active inline vocabulary for subsequent recognition:

```json
{
  "type": "session.update",
  "config": {
    "vocabulary": {
      "phrases": ["Project Apollo", "Jane Chen"]
    }
  }
}
```

## Create a Vocabulary Asset

Create an asset when the same phrase list must be reused across requests or sessions. The create body takes `phrases` at the top level; it does not accept the nested `vocabulary: {"phrases":[...]}` request shape, which belongs only to transcription `config`.

```bash
curl -X POST https://api.zoom.us/v2/aiservices/assets/vocabularies \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme English Vocabulary",
    "language": "en-US",
    "phrases": [
      "Project Atlas",
      "ServiceNow",
      "Zoom AI Companion"
    ]
  }'
```

Save the server-generated `vocabulary_id`. The registered language must be compatible with the recognition language of every request that uses the asset. Create and get responses can also include `updated_at`.

## Apply a Vocabulary Asset

Set `config.vocabulary_id` in Fast, Batch, or Live:

```json
{
  "file": "https://example.com/audio.wav",
  "config": {
    "language": "en-US",
    "vocabulary_id": "acct_vocab_abc123"
  }
}
```

## Manage Vocabulary Assets

| Action | Endpoint |
|---|---|
| Create | `POST /aiservices/assets/vocabularies` |
| List | `GET /aiservices/assets/vocabularies` |
| Get | `GET /aiservices/assets/vocabularies/{vocabularyId}` |
| Delete | `DELETE /aiservices/assets/vocabularies/{vocabularyId}` |

List results are paginated with `page_size` and `next_page_token`. Vocabulary Assets are account-scoped and immutable; to change a saved list, create a new asset and use its new ID.

The how-to documentation states that deletion returns HTTP `200` with an empty JSON object and that a later get returns `404`. The 2026-11-19 OpenAPI snapshot instead declares `201` for create and `204` for delete; treat the OpenAPI status codes as the machine-readable contract and verify observed behavior when the difference matters.

## Sources

- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/scribe/custom-vocabulary.md`
- OpenAPI snapshot 2026-11-19: `developers.zoom.us/docs/api/ai-services/openapi-ai-services.json`
