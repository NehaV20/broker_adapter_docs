# Blitz Order Message Structure Document

## 1. Overview

Blitz uses a standardized `OrderLog` object to store and communicate order events, regardless of the broker. Adapter modules map broker-specific events into this standardized format.

There are two main flows:

- **From Broker → Adapter → Blitz**: Raw broker events are converted to Blitz `OrderLog`.
- **From Blitz → Adapter → Broker**: Order requests from Blitz are sent to broker-specific formats via adapters.

This document focuses on the message structure used in these flows.

---

## 2. Blitz Standard OrderLog

All order messages in Blitz follow this structure:

| Field | Type | Description | Notes |
|-------|------|-------------|-------|
| ExchangeOrderId | str | Broker-provided order ID | Primary key per exchange |
| ExecutionId | str | Internal/execution ID | Typically same as `order_id` in broker data |
| Account | str | Account identifier | Broker account ID |
| InstrumentId | int | Unique instrument token | Broker-specific numeric ID |
| InstrumentName | str | Tradingsymbol | e.g., RELIANCE |
| OrderQuantity | int | Total order quantity | |
| OrderPrice | float | Order price | |
| OrderSide | str | BUY / SELL | Always uppercase |
| OrderType | str | LIMIT / MARKET | Always uppercase |
| OrderStatus | str | NEW / FILLED / CANCELLED / REJECTED | Standardized across brokers |
| LeavesQuantity | int | Quantity remaining to fill | |
| CumulativeQuantity | int | Quantity already executed | |
| OrderTriggerPrice | float | Trigger price for stop orders | 0 if not applicable |
| CancelledQuantity | int | Quantity cancelled | |
| AverageTradedPrice | float | Average executed price | 0 if not yet traded |
| OrderGeneratedDateTime | int | Epoch timestamp in ms | When order was generated in Blitz |
| ExchangeTransactTime | int | Epoch timestamp in ms | Broker execution timestamp |
| UserText | dict | Raw broker JSON | Reference for debugging |

### Example JSON

```json
{
  "ExchangeOrderId": "ABC123",
  "ExecutionId": "12345",
  "Account": "ACC001",
  "InstrumentId": 256265,
  "InstrumentName": "RELIANCE",
  "OrderQuantity": 100,
  "OrderPrice": 2500.5,
  "OrderSide": "BUY",
  "OrderType": "LIMIT",
  "OrderStatus": "NEW",
  "LeavesQuantity": 100,
  "CumulativeQuantity": 0,
  "OrderTriggerPrice": 0,
  "CancelledQuantity": 0,
  "AverageTradedPrice": 0,
  "OrderGeneratedDateTime": 1701638400000,
  "ExchangeTransactTime": 1701638400000,
  "UserText": {
    "exchange_order_id": "ABC123",
    "order_id": "12345",
    "quantity": 100,
    "price": 2500.5,
    "status": "OPEN"
  }
}
```

---

## 3. Message Flow: Broker → Adapter → Blitz

**Steps:**

1. Adapter receives raw broker order JSON.
2. Adapter converts it into `OrderLog` using broker-specific mapping logic.
3. `UserText` stores the original broker message for reference.
4. Blitz consumes the standardized `OrderLog`.

### Example (Zerodha)

**Broker Raw Data:**

```json
{
  "order_id": "12345",
  "exchange_order_id": "ABC123",
  "tradingsymbol": "RELIANCE",
  "instrument_token": 256265,
  "quantity": 100,
  "price": 2500.5,
  "transaction_type": "BUY",
  "order_type": "LIMIT",
  "status": "OPEN",
  "average_price": 0.0,
  "order_timestamp": "2026-01-02T13:00:00",
  "exchange_timestamp": "2026-01-02T13:00:00"
}
```

**Adapter → Blitz:**

```json
{
  "ExchangeOrderId": "ABC123",
  "ExecutionId": "12345",
  "OrderSide": "BUY",
  "OrderType": "LIMIT",
  "OrderStatus": "NEW",
  "UserText": { ...original broker JSON... }
}
```

---

## 4. Message Flow: Blitz → Adapter → Broker

When Blitz generates an order request:

1. Blitz sends an OrderLog-like request to the adapter.
2. Adapter converts Blitz fields to broker-specific API fields.
3. Adapter sends request to broker API.

**Mapping Example (Blitz → Zerodha):**

| Blitz Field | Zerodha API Field |
|-------------|-----------------|
| OrderSide   | transaction_type |
| OrderType   | order_type      |
| OrderQuantity | quantity       |
| OrderPrice  | price           |
| InstrumentId | instrument_token |
| Account     | account_id      |

---

## 5. Status Mapping

Blitz normalizes broker statuses:

| Broker Status | Blitz Status |
|---------------|-------------|
| OPEN          | NEW         |
| COMPLETE      | FILLED      |
| CANCELLED     | CANCELLED   |
| REJECTED      | REJECTED    |

Other statuses pass through as-is if unrecognized.

---

## 6. Timestamps

All timestamps in Blitz are stored as **epoch milliseconds**:

- `OrderGeneratedDateTime`: Order creation time
- `ExchangeTransactTime`: Broker execution time

Adapter must convert ISO format strings to epoch ms.

---

## 7. Notes for Adapters

- Always populate `UserText` with raw broker JSON.
- OrderLog fields should be fully filled where possible.
- Any missing numeric field defaults to `0`.
- Any missing string field defaults to `""` or `None`.

