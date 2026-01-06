# Order Requests Documentation

This document describes how to publish **Place Order**, **Modify Order**, and **Cancel Order**
requests to the Adapter (Zerodha) via Redis.

---

## Redis Publishing Overview

All requests are published to a Redis channel as JSON.

- **Channel**: `adapter.channel`
- **Message format**: JSON
- **Publisher**: Blitz
- **Consumer**: Broker Adapter (Zerodha)

---

## Common Request Structure

| Field | Type | Description | Notes |
|------|------|------------|------|
| request_id | string | Unique request identifier | Must be unique per request |
| action | string | Action name | Determines request type |
| data | object | Payload for the action | Mandatory |

---

## 1. PLACE ORDER

### Action
PLACE_ORDER

### Data Fields

| Field | Type | Description | Notes |
|------|------|------------|------|
| symbol | string | Exchange + symbol | NSE|RELIANCE |
| quantity | int | Total order quantity | Required |
| product | string | Product type | MIS / CNC / NRML |
| tif | string | Time in force | DAY / IOC |
| price | float | Order price | Required for LIMIT |
| orderType | string | Order type | LIMIT / MARKET / SL / SL-M |
| orderSide | string | Order side | BUY / SELL |
| stopPrice | float | Trigger price | 0 if not applicable |

### Example

```json
{
  "request_id": "place_001",
  "action": "PLACE_ORDER",
  "data": {
    "BOID": "BOID123",
    "instrumentId": 23455666,
    "symbol": "NSE|RELIANCE",
    "quantity": 1,
    "product": "MIS",
    "tif": "DAY",
    "price": 1560,
    "orderType": "LIMIT",
    "orderSide": "BUY",
    "stopPrice": 0
  }
}
```

---

## 2. MODIFY ORDER

### Action
MODIFY_ORDER

### Data Fields

| Field | Type | Description | Notes |
|------|------|------------|------|
| order_id | string | Broker order ID | Mandatory |
| quantity | int | Modified quantity | Optional |
| orderType | string | Order type | LIMIT / MARKET |
| validity | string | Order validity | DAY / IOC |

### Example

```json
{
  "request_id": "modify_001",
  "action": "MODIFY_ORDER",
  "data": {
    "BOID": "BOID123",
    "quantity": 2,
    "orderType": "LIMIT",
    "validity": "DAY"
  }
}
```

---

## 3. CANCEL ORDER

### Action
CANCEL_ORDER

### Data Fields

| Field | Type | Description | Notes |
|------|------|------------|------|
| order_id | string | Broker order ID | Mandatory |

### Example

```json
{
  "request_id": "cancel_001",
  "action": "CANCEL_ORDER",
  "data": {
    "BOID": "BOID123",
  }
}
```

---

## Notes

- All enum values must be **UPPERCASE**
- `order_id` refers to **ExchangeOrderId** from broker
- Adapter maps broker response into `OrderLog`
- Errors / rejects are published back asynchronously
