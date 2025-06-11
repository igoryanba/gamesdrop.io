# GamesDrop.io API – Merchant Guide

## Table of Contents
1. [Authorization](#authorization)
   - [Obtaining a Token](#obtaining-a-token)
   - [Token Security](#token-security)
2. [Core Concepts](#core-concepts)
   - [Order Statuses](#order-statuses)
   - [Error Handling](#possible-errors)
3. [API Endpoints](#api-endpoints)
   - [Balance Check](#balance-check)
   - [Product Management](#get-product-information)
   - [Order Management](#create-order)
   - [Player Validation](#player-validation)
4. [API Testing](#api-testing)
   - [Test Product](#test-product)
5. [Integration Guidelines](#integration-guidelines)
6. [Support](#support)

## Authorization

### Obtaining a Token

#### Process
1. Log in to your merchant dashboard.
2. Create a new shop.
3. After the shop is created you will receive a unique token.
4. Token format example: `abcdef1234567890abcdef1234567890`

### Token Security
- 🔒 The token is confidential information.
- ⚠️ Do NOT share the token with third parties.
- 📝 The token cannot be restored once generated.
- 🔄 You can generate a new token if needed (the old one will be invalidated).

## Core Concepts

### Order Statuses

| Status | Description | Actions |
|--------|-------------|---------|
| `SUBMITTED` | Order created, awaiting processing | Wait until it moves to PROCESSING |
| `PROCESSING` | Order is being processed | Wait for completion |
| `COMPLETED` | Order processed successfully | Retrieve key/product |
| `CANCELED` | Order canceled due to an error | Create a new order |
| `REFUND` | Order refunded | Wait for funds to be returned |

### Possible Errors

| Error Code | Description | Resolution |
|------------|-------------|-----------|
| `INVALID_TOKEN` | Invalid authorization token | Verify or obtain a new token |
| `OFFER_NOT_FOUND` | Product not found or access denied | Check product ID and permissions |
| `TRANSACTION_DUPLICATE` | Duplicate transaction | Use a new `transaction_id` |
| `WRONG_PRICE` | Incorrect product price | Update price information |
| `ORDER_NOT_FOUND` | Order not found | Verify order ID |
| `ORDER_NOT_PROCESSING` | Order not yet in processing | Wait until status is PROCESSING |
| `ORDER_NOT_COMPLETED` | Order not yet completed | Wait for completion |
| `ORDER_ALREADY_CANCELED` | Order already canceled | Create a new order |
| `ORDER_ALREADY_REFUNDED` | Order already refunded | Create a new order |
| `SERVICE_UNAVAILABLE` | Service temporarily unavailable | Try again later |
| `INVALID_REQUEST_BODY` | Invalid request format | Check request structure |

## API Endpoints

### Balance Check

```http
POST /api/v1/balance/check
Authorization: {{token}}
```

**Response:**
```json
{
  "currency": "RUB",
  "balance": 689240.32
}
```

**Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | Defined on registration |
| `balance` | `null` \| `number` | - |

**Note:** This feature is under development and might be temporarily unavailable.

### Get Product Information

```http
POST /api/v1/offers/find-one
Authorization: {{token}}

{
  "offerId": 1001
}
```

**Response:**
```json
{
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "isReturnDataForCustomer": true
}
```

**Request Fields:**
| Key | Value | Note |
|-----|-------|------|
| `offerId` | `number` | Must be a number, not a string |

**Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `count` | `number` | Quantity |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `isReturnDataForCustomer` | `boolean` | Whether activation key is returned |

### Create Order

```http
POST /api/v1/offers/create-order
Authorization: {{token}}

{
  "offerId": 1001,
  "price": 560.10,
  "transactionId": "test112321124214",
  "customer": {
    "email": "user@gmail.com",
    "gameUserId": "52357322414"
  }
}
```

**Request Fields:**
| Key | Value | Note |
|-----|-------|------|
| `offerId` | `number` | Must be a number, not a string |
| `price` | `number` | - |
| `transactionId` | `string` | Unique transaction identifier in your system |
| `customer` | `undefined` \| `object` | - |
| `email` | `undefined` \| `string` | Optional field for tracking |
| `gameUserId` | `undefined` \| `string` | Required for certain product types |

**Response:**
```json
{
  "orderId": 10222502,
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "status": "COMPLETED",
  "isReturnDataForCustomer": true,
  "key": "001434249936",
  "createdAt": "2024-05-28 10:08:04.296+00"
}
```

**Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `orderId` | `number` | - |
| `count` | `number` | Quantity |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `status` | `string` | Processing of some denominations may take time |
| `isReturnDataForCustomer` | `boolean` | - |
| `key` | `undefined` \| `string` | Returned only when `COMPLETED` and `isReturnDataForCustomer=true` |
| `createdAt` | `string` | UTC +0 |

### Check Order Status

```http
POST /api/v1/order/find-one
Authorization: {{token}}

{
  "orderId": 10222502
}
```

**Request Fields:**
| Key | Value | Note |
|-----|-------|------|
| `orderId` | `number` | - |

**Response:**
```json
{
  "orderId": 10222502,
  "count": 1,
  "price": 560.10,
  "currency": "RUB",
  "offerId": 1001,
  "productName": "PUBG MOBILE GIFT",
  "offerName": "60 UC",
  "status": "COMPLETED",
  "isReturnDataForCustomer": true,
  "key": "001434249936",
  "createdAt": "2024-05-28 10:08:04.296+00"
}
```

**Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `orderId` | `number` | - |
| `count` | `number` | Quantity |
| `price` | `number` | - |
| `currency` | `KZT` \| `USD` \| `EUR` \| `RUB` | - |
| `offerId` | `number` | - |
| `productName` | `string` | - |
| `offerName` | `string` | - |
| `status` | `string` | Current order status |
| `isReturnDataForCustomer` | `boolean` | - |
| `key` | `undefined` \| `string` | Returned only when `COMPLETED` and `isReturnDataForCustomer=true` |
| `createdAt` | `string` | UTC +0 |

### Player Validation

```http
POST /api/v1/offers/check-game-data
Authorization: {{token}}

{
  "offerId": 1001,
  "price": 560.10,
  "transactionId": "112321124214",
  "customer": {
    "email": "user@gmail.com",
    "gameUserId": "52357322414"
  }
}
```

**Request Fields:**
| Key | Value | Note |
|-----|-------|------|
| `offerId` | `number` | Must be a number, not a string |
| `price` | `number` | - |
| `transactionId` | `string` | Unique transaction identifier |
| `customer` | `object` | - |
| `email` | `undefined` \| `string` | Optional field |
| `gameUserId` | `string` | Player identifier |
| `gameServerId` | `undefined` \| `string` | Server identifier (if required) |

**Successful Response:**
```json
{
  "status": "VALID",
  "gameUserLogin": "JJJ"
}
```

**Successful Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `status` | `"VALID"` | Player is valid |
| `gameUserLogin` | `string` | Player login |

**Error Response:**
```json
{
  "status": "INVALID"
}
```

**Error Response Fields:**
| Key | Value | Note |
|-----|-------|------|
| `status` | `"INVALID"` | Player is not valid |

## API Testing

### Test Product

A special product is available for testing your API integration:

- **Product ID:** 999
- **Product Name:** Steam US
- **Offer Name:** TEST OFFER GROUP
- **Price:** 23.09 KZT (use the exact value)
- **Returns Key:** Yes

**Example request to get test product info:**
```http
POST /api/v1/offers/find-one
Authorization: {{token}}

{
  "offerId": 999
}
```

**Example request to create a test order:**
```http
POST /api/v1/offers/create-order
Authorization: {{token}}

{
  "offerId": 999,
  "price": 23.09,
  "transactionId": "test_123456",
  "customer": {
    "email": "test@example.com",
    "gameUserId": "123456789"
  }
}
```

**Test Product Characteristics:**
- Always returns a successful response with correct parameters.
- Generates a test activation key.
- Creates an order with the `COMPLETED` status.
- Player validation always returns `VALID` for the test product.
- Works in test mode without real charges.

## Integration Guidelines

### 🔍 Before Creating an Order
- Fetch up-to-date product information.
- Validate the player (for direct top-ups).

### 📝 When Creating an Order
- Use a unique `transactionId`.
- Specify the current price.
- Provide `offerId` as a number, not a string.
- Fill in all required fields for the given product type.

### ✅ After Creating an Order
- Save the `orderId`.
- Check the order status.
- Retrieve the key/product when status is `COMPLETED`.

### ⚠️ In Case of Errors
- Verify the token.
- Ensure data correctness and request format.
- Create a new order if necessary.

## Support

If you have any questions, please contact technical support:
- 📧 Email: support@gamesdrop.io
- 💬 Telegram: @igoryan34

### Error Handling Guidelines

#### 🔍 Pre-Request Checks
- Token validation.
- Product ID check.
- Price accuracy.
- `transaction_id` uniqueness.
- Data format correctness (especially `offerId` as a number).

#### 🛠 Response Handling
- Handle all error codes.
- Log errors.
- Implement retry mechanisms.

#### 📊 Working with Orders
- Save order IDs.
- Monitor statuses.
- Keep data up to date. 
