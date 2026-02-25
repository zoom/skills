# Commerce API

App monetization and entitlement management for Zoom Marketplace apps.

## Overview

The Commerce API enables Zoom App developers to manage in-app purchases, subscriptions, and user entitlements. It powers the monetization features for apps distributed through the Zoom App Marketplace.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with app-specific scopes:
- `app:read:entitlements` - Read entitlements
- `app:write:entitlements` - Manage entitlements

## Key Endpoints

### Entitlements

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/apps/{appId}/entitlements` | List all entitlements |
| POST | `/apps/{appId}/entitlements/check` | Check user entitlement |
| GET | `/apps/{appId}/users/{userId}/entitlements` | Get user's entitlements |

### Subscriptions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/apps/{appId}/subscriptions` | List subscriptions |
| GET | `/apps/{appId}/subscriptions/{subscriptionId}` | Get subscription details |
| POST | `/apps/{appId}/subscriptions/{subscriptionId}/cancel` | Cancel subscription |

### Products

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/apps/{appId}/products` | List available products |
| GET | `/apps/{appId}/products/{productId}` | Get product details |

### Purchases

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/apps/{appId}/purchases` | List purchases |
| GET | `/apps/{appId}/purchases/{purchaseId}` | Get purchase details |

## Example: Check Entitlement

```bash
curl -X POST "https://api.zoom.us/v2/apps/{appId}/entitlements/check" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_abc",
    "product_id": "premium_feature",
    "account_id": "account_xyz"
  }'
```

### Response

```json
{
  "entitled": true,
  "entitlement": {
    "id": "ent_123",
    "product_id": "premium_feature",
    "type": "subscription",
    "status": "active",
    "valid_until": "2024-12-31T23:59:59Z",
    "quantity": 1
  }
}
```

## Example: List User Entitlements

```bash
curl "https://api.zoom.us/v2/apps/{appId}/users/{userId}/entitlements" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "entitlements": [
    {
      "id": "ent_123",
      "product_id": "premium_plan",
      "product_name": "Premium Plan",
      "type": "subscription",
      "status": "active",
      "created_at": "2024-01-15T10:00:00Z",
      "valid_until": "2024-02-15T10:00:00Z",
      "auto_renew": true
    },
    {
      "id": "ent_456",
      "product_id": "addon_storage",
      "product_name": "Extra Storage",
      "type": "one_time",
      "status": "active",
      "created_at": "2024-01-20T14:30:00Z",
      "valid_until": null
    }
  ]
}
```

## Subscription Object

```json
{
  "id": "sub_abc123",
  "product_id": "premium_monthly",
  "user_id": "user_xyz",
  "account_id": "acc_123",
  "status": "active",
  "plan": {
    "id": "plan_monthly",
    "name": "Premium Monthly",
    "interval": "month",
    "price": 9.99,
    "currency": "USD"
  },
  "current_period": {
    "start": "2024-01-15T00:00:00Z",
    "end": "2024-02-15T00:00:00Z"
  },
  "cancel_at_period_end": false,
  "created_at": "2024-01-15T10:00:00Z"
}
```

## Entitlement Types

| Type | Description |
|------|-------------|
| `subscription` | Recurring billing entitlement |
| `one_time` | Single purchase, perpetual |
| `trial` | Free trial period |
| `promotional` | Promotional/gifted access |

## Entitlement Status

| Status | Description |
|--------|-------------|
| `active` | Currently valid |
| `expired` | Past valid_until date |
| `cancelled` | User cancelled |
| `suspended` | Payment issue |
| `pending` | Awaiting activation |

## Webhook Events

Handle these events in your app:

| Event | Description |
|-------|-------------|
| `app.entitlement_granted` | New entitlement created |
| `app.entitlement_revoked` | Entitlement removed |
| `app.subscription_renewed` | Subscription renewed |
| `app.subscription_cancelled` | User cancelled |
| `app.trial_started` | Free trial began |
| `app.trial_ended` | Free trial expired |
| `app.payment_failed` | Payment processing failed |

## Implementation Pattern

```javascript
// Middleware to check entitlement
async function requireEntitlement(productId) {
  return async (req, res, next) => {
    const { entitled } = await checkEntitlement(
      req.app.id,
      req.user.id,
      productId
    );
    
    if (!entitled) {
      return res.status(403).json({
        error: 'subscription_required',
        message: 'This feature requires a premium subscription',
        upgrade_url: `/upgrade?product=${productId}`
      });
    }
    
    next();
  };
}

// Usage
app.get('/api/premium-feature', 
  requireEntitlement('premium_plan'),
  (req, res) => {
    // Premium feature logic
  }
);
```

## Client-Side Purchase Flow

```javascript
// Initiate purchase from Zoom App
import zoomSdk from '@zoom/appssdk';

async function initiatePurchase(productId) {
  try {
    await zoomSdk.openPurchaseFlow({
      product_id: productId,
      success_url: 'https://yourapp.com/purchase/success',
      cancel_url: 'https://yourapp.com/purchase/cancel'
    });
  } catch (error) {
    console.error('Purchase flow error:', error);
  }
}
```

## Revenue Share

| Party | Percentage |
|-------|------------|
| Developer | 85% |
| Zoom | 15% |

## Resources

- **Monetization Guide**: https://developers.zoom.us/docs/distribute/monetization/
- **Marketplace Distribution**: https://developers.zoom.us/docs/distribute/
- **Webhook Events**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/events/
