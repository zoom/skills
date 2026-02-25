# Zoom Number Management API

Phone number provisioning and management API.

## Overview

The Number Management API enables provisioning, porting, and management of phone numbers for Zoom Phone. It covers number acquisition, assignment, porting requests, and number inventory management.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with phone/number management scopes:
- `phone:read:admin` - Read phone numbers
- `phone:write:admin` - Manage phone numbers
- `phone_number:read:admin` - Read number inventory
- `phone_number:write:admin` - Provision numbers

## Key Endpoint Categories

### Phone Numbers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/numbers` | List all phone numbers |
| GET | `/phone/numbers/{numberId}` | Get number details |
| POST | `/phone/numbers` | Add/provision number |
| PATCH | `/phone/numbers/{numberId}` | Update number |
| DELETE | `/phone/numbers/{numberId}` | Release number |

### Number Search & Provisioning

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/numbers/available` | Search available numbers |
| POST | `/phone/numbers/provision` | Provision new numbers |
| GET | `/phone/numbers/inventory` | Get number inventory |

### Number Assignment

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/phone/numbers/{numberId}/assign` | Assign to user/resource |
| DELETE | `/phone/numbers/{numberId}/assign` | Unassign number |
| GET | `/phone/numbers/unassigned` | List unassigned numbers |

### Porting

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/porting/orders` | List porting orders |
| POST | `/phone/porting/orders` | Create port-in request |
| GET | `/phone/porting/orders/{orderId}` | Get porting order status |
| PATCH | `/phone/porting/orders/{orderId}` | Update porting order |
| DELETE | `/phone/porting/orders/{orderId}` | Cancel porting order |
| POST | `/phone/porting/validate` | Validate portability |

### Number Blocks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/number_blocks` | List number blocks |
| POST | `/phone/number_blocks` | Create number block |
| DELETE | `/phone/number_blocks/{blockId}` | Release number block |

### Toll-Free Numbers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/numbers/toll_free` | List toll-free numbers |
| GET | `/phone/numbers/toll_free/available` | Search toll-free numbers |
| POST | `/phone/numbers/toll_free` | Provision toll-free number |

## Example: Search Available Numbers

```bash
curl -X GET "https://api.zoom.us/v2/phone/numbers/available?country=US&state=CA&area_code=415&quantity=10" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "available_numbers": [
    {
      "phone_number": "+14155551001",
      "country": "US",
      "state": "CA",
      "city": "San Francisco",
      "type": "local",
      "capabilities": ["voice", "sms"],
      "monthly_cost": 1.00
    },
    {
      "phone_number": "+14155551002",
      "country": "US",
      "state": "CA",
      "city": "San Francisco",
      "type": "local",
      "capabilities": ["voice", "sms"],
      "monthly_cost": 1.00
    }
  ]
}
```

## Example: Provision Numbers

```bash
curl -X POST "https://api.zoom.us/v2/phone/numbers/provision" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "phone_numbers": ["+14155551001", "+14155551002"],
    "site_id": "site_abc"
  }'
```

### Response

```json
{
  "provisioned_numbers": [
    {
      "id": "num_xyz1",
      "phone_number": "+14155551001",
      "status": "active"
    },
    {
      "id": "num_xyz2",
      "phone_number": "+14155551002",
      "status": "active"
    }
  ]
}
```

## Example: Create Port-In Request

```bash
curl -X POST "https://api.zoom.us/v2/phone/porting/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "phone_numbers": ["+14155559999"],
    "carrier_name": "Previous Carrier",
    "account_number": "123456789",
    "account_pin": "1234",
    "authorized_person": "John Doe",
    "billing_address": {
      "address": "123 Main St",
      "city": "San Francisco",
      "state": "CA",
      "zip": "94105"
    },
    "requested_port_date": "2024-02-01"
  }'
```

### Response

```json
{
  "order_id": "port_abc123",
  "status": "submitted",
  "phone_numbers": ["+14155559999"],
  "submitted_at": "2024-01-15T10:00:00Z",
  "estimated_completion": "2024-02-01"
}
```

## Example: Assign Number

```bash
curl -X POST "https://api.zoom.us/v2/phone/numbers/{numberId}/assign" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "assignee_type": "user",
    "assignee_id": "user_abc123"
  }'
```

## Number Types

| Type | Description |
|------|-------------|
| `local` | Local geographic number |
| `toll_free` | Toll-free number |
| `national` | National number |
| `mobile` | Mobile number |

## Number Capabilities

| Capability | Description |
|------------|-------------|
| `voice` | Voice calling |
| `sms` | SMS messaging |
| `mms` | MMS messaging |
| `fax` | Fax capability |

## Assignee Types

| Type | Description |
|------|-------------|
| `user` | Individual user |
| `call_queue` | Call queue |
| `auto_receptionist` | Auto receptionist |
| `common_area` | Common area phone |
| `zoom_room` | Zoom Room |

## Porting Status

| Status | Description |
|--------|-------------|
| `submitted` | Order submitted |
| `pending` | Awaiting carrier |
| `scheduled` | Port date scheduled |
| `completed` | Port successful |
| `rejected` | Port rejected |
| `cancelled` | Order cancelled |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/phone/methods/#tag/Phone-Numbers
- **Number Management Guide**: https://developers.zoom.us/docs/zoom-phone/number-management/
- **Porting Guide**: https://support.zoom.us/hc/en-us/articles/360028869972
