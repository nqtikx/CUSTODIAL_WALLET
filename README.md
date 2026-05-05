# CUSTODIAL_WALLET

<<<<<<< Updated upstream
The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.  
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.  

> BASE_URL https://api.dev.wbdevel.net
=======
The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.

All examples use `{{URL}}` format. Merchant API requests must include `x-api-key`.

---
>>>>>>> Stashed changes

## Common Request Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key used to authenticate server-to-server requests. |
| `Content-Type` | `string` | Conditional | Use `application/json` for requests with JSON body. Not required for GET requests without body. |

## Common Error Shape

Most errors are returned as JSON object with a business status/code and human-readable message.

```json
{
  "message": "error message",
  "code": 400,
  "status": "ERROR_CODE"
}
```

---

## 0) Wallet Base Data

Base data endpoints are used before any wallet operation. They help the merchant show available assets, current operations, and account balances to the client.

### Step 0.1 Get Available Assets

Use this endpoint to get the list of fiat and crypto assets available for custodial wallet flows.

<<<<<<< Updated upstream
### Step 0.1 Get available assets

Use this endpoint to retrieve all fiat and crypto assets available for custodial wallet operations. Use the response to build asset selectors and validate supported routes before any operation.
**POST** `/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`

### Headers
- `x-api-key: {{x-api-key}}`
=======
**POST** `{{URL}}/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`
>>>>>>> Stashed changes

**Response**

```json
{
  "fiatAssets": [
    { "id": "BYN", "code": "BYN" },
    { "id": "RUB", "code": "RUB" },
    { "id": "USD", "code": "USD" },
    { "id": "EUR", "code": "EUR" }
  ],
  "cryptoAssets": [
    { "id": "BTC", "code": "BTC", "network": "Bitcoin" },
    { "id": "ETH", "code": "ETH", "network": "Ethereum" },
    { "id": "TRX", "code": "TRX", "network": "Tron" },
    { "id": "USDT_TRC", "code": "USDT", "network": "Tron", "protocol": "TRC-20" }
  ]
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `destination` | `string` | No | Query parameter that filters assets for a specific flow. For custodial wallet use `SDK_ACCOUNTING`. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatAssets` | `array of objects` | Yes | List of fiat assets that can be shown to the client as available wallet currencies for this merchant flow. |
| `fiatAssets[].id` | `string` | Yes | Internal asset identifier used in API requests and routing logic. |
| `fiatAssets[].code` | `string` | Yes | Currency code that can be displayed to the client in UI. |
| `cryptoAssets` | `array of objects` | Yes | List of crypto assets/networks that can be used in deposit, withdrawal, buy, sell, or conversion flows. |
| `cryptoAssets[].id` | `string` | Yes | Internal crypto asset identifier used in API requests; may include network-specific suffixes such as `USDT_TRC`. |
| `cryptoAssets[].code` | `string` | Yes | Asset ticker displayed to the client; can differ from `id` when asset is network-specific. |
| `cryptoAssets[].network` | `string` | No | Blockchain network that must be used for deposits/withdrawals of this asset. |
| `cryptoAssets[].protocol` | `string` | No | Token protocol shown to prevent sending funds through the wrong network. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_DESTINATION` | Business error | No | `destination` value cannot be mapped to supported enum for merchant assets. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant is authenticated but does not have `ASSET_API` permission. |
| `429 Too Many Requests` | HTTP error | No | Rate limit is exceeded for this endpoint. |


### Step 0.2 Get current balance operations

Use this endpoint to fetch the client's current fiat and crypto wallet operations and statuses. Use the response to show live operation state in UI and support dashboards.
**GET** `/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | No | No JSON body is sent. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `destination` | `string` | No | Query parameter that filters assets for a specific flow. For custodial wallet use `SDK_ACCOUNTING`. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatAssets` | `array<object>` | Yes | Fiat currencies available for the merchant flow. |
| `fiatAssets[].id` | `string` | Yes | Internal fiat asset identifier. |
| `fiatAssets[].code` | `string` | Yes | Display currency code. |
| `cryptoAssets` | `array<object>` | Yes | Crypto assets available for the merchant flow. |
| `cryptoAssets[].id` | `string` | Yes | Internal crypto asset identifier, including network-specific asset id when applicable. |
| `cryptoAssets[].code` | `string` | Yes | Display crypto currency code. |
| `cryptoAssets[].network` | `string` | No | Blockchain network used for the asset. |
| `cryptoAssets[].protocol` | `string` | No | Token standard or protocol, for example `TRC-20`. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Merchant does not have access to this endpoint. |
| `400 Bad Request` | HTTP error | No | Invalid or unsupported query parameter value. |

### Step 0.2 Get Current Balance Operations

Use this endpoint to show the client's active wallet operations, such as pending deposits and withdrawals.

**GET** `{{URL}}/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

>>>>>>> Stashed changes
**Response**

```json
{
  "fiatOperations": [
    {
      "number": 9210086,
      "accountType": "WALLET",
      "operationType": "DEPOSIT",
      "amount": 100,
      "transactionId": "fiat-transaction-id",
      "asset": "BYN",
      "status": "PROCESSING",
      "fiatProvider": "ASSIST",
      "orderIdentity": "order-id",
      "createdAt": "2026-04-30T10:00:00"
    }
  ],
  "cryptoOperations": [
    {
      "number": 9210087,
      "accountType": "WALLET",
      "operationType": "DEPOSIT",
      "submitTimeout": "DEFAULT",
      "transactionId": "crypto-transaction-id",
      "status": "PENDING",
      "depositCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
      "amount": 100,
      "asset": "USDT_TRC",
      "network": "Tron",
      "txHash": null,
      "createdAt": "2026-04-30T10:00:00"
    }
  ]
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to scope the request to one merchant client and return only that client's wallet data. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatOperations` | `array of objects` | Yes | Current fiat wallet operations. |
| `cryptoOperations` | `array of objects` | Yes | Current crypto wallet operations. |
=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | No | No JSON body is sent. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client identifier whose active balance operations must be returned. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatOperations` | `array<object>` | Yes | Current fiat wallet operations. |
| `cryptoOperations` | `array<object>` | Yes | Current crypto wallet operations. |
>>>>>>> Stashed changes
| `number` | `number` | Yes | Human-readable operation number. |
| `accountType` | `string` | Yes | Balance account type. For custodial wallet use `WALLET`. |
| `operationType` | `string` | Yes | Operation direction/type, for example `DEPOSIT` or `WITHDRAWAL`. |
| `amount` | `number` | Yes | Operation amount in the asset currency. |
| `transactionId` | `string` | Yes | Internal fiat or crypto transaction id. |
| `asset` | `string` | Yes | Asset used by the operation. |
| `status` | `string` | Yes | Current processing status of the operation. |
| `fiatProvider` | `string` | No | Fiat provider used by fiat operation. |
| `orderIdentity` | `string` | No | Provider/order reference. |
| `submitTimeout` | `string` | No | Crypto deposit timeout mode. |
| `depositCryptoAddress` | `string` | No | Address where the user must send crypto for deposit. |
| `network` | `string` | No | Blockchain network. |
<<<<<<< Updated upstream
| `txHash` | `string/null` | No | Blockchain transaction hash after the crypto transfer is detected; use for explorer links and reconciliation. |
| `createdAt` | `string` | Yes | Operation creation date/time. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to the merchant in access validation. |
| `400 Bad Request` | HTTP error | No | Request parameters are invalid or cannot be parsed. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for this client or endpoint. |


### Step 0.3 Get enhanced client account balances

Use this endpoint to get enhanced client account balances grouped by currency type. Use the response to populate balance widgets and reconciliation summaries.
**POST** `/api/v2/accounting/client/account/enhanced`

### Headers
- `Authorization: Bearer {{accessToken}}`

**Request**
```json
{
  "clientId": "{{cleintId}}"
}
```
=======
| `txHash` | `string \| null` | No | Blockchain transaction hash when known. |
| `createdAt` | `string` | Yes | Operation creation date/time. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Merchant does not have access to the client or endpoint. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or client is not linked to the merchant. |

### Step 0.3 Get Enhanced Merchant Account Balances

Use this endpoint to show merchant account balances grouped by fiat and crypto assets.

**GET** `{{URL}}/api/v2/accounting/merchant/account/enhanced`
>>>>>>> Stashed changes

**Response**

```json
{
  "balances": [
    {
      "currency": "BTC",
      "type": "USER_BALANCE",
      "amount": 0.00005088,
      "usdRate": 77622.51,
      "usdAmount": 3.95,
      "creationDate": 1732799992688,
      "modificationDate": 1777534608348,
      "fiat": false
    }
  ],
  "totalFiatUsdAmount": 134.31,
  "totalCryptoUsdAmount": 93.48
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `Authorization` | `string` | Yes | Bearer access token used to authenticate the client user context. Format: `Bearer <accessToken>`. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| Body | `object` | Yes | `AccountFilterRequest` payload used to resolve the target client balance scope. |
| `clientId` | `string` | No | Optional client identifier. If omitted, backend resolves client by authenticated user context. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `balances` | `array of objects` | Yes | List of client account balances. |
=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | No | No JSON body is sent. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| Body | `object` | No | No request body is required. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `balances` | `array<object>` | Yes | List of merchant account balances. |
>>>>>>> Stashed changes
| `balances[].currency` | `string` | Yes | Currency or asset code. |
| `balances[].type` | `string` | Yes | Balance type, for example `USER_BALANCE`. |
| `balances[].amount` | `number` | Yes | Available balance amount. |
| `balances[].usdRate` | `number` | No | Current USD conversion rate used for display/summary. |
| `balances[].usdAmount` | `number` | No | Balance value converted to USD. |
| `balances[].creationDate` | `number` | No | Balance creation timestamp in milliseconds. |
| `balances[].modificationDate` | `number` | No | Last update timestamp in milliseconds. |
| `balances[].fiat` | `boolean` | Yes | `true` for fiat currency, `false` for crypto asset. |
| `totalFiatUsdAmount` | `number` | Yes | Total fiat balances converted to USD. |
| `totalCryptoUsdAmount` | `number` | Yes | Total crypto balances converted to USD. |

<<<<<<< Updated upstream
### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 Bad Request` | HTTP error | No | Request is malformed or unsupported by accounting endpoint. |
| `401 Unauthorized` | HTTP error | Yes | Authorization header is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Authenticated user does not have required `USER` authority. |
| `500 Internal Server Error` | HTTP error | No | Unexpected accounting/balance aggregation failure. |


## 1) Deposit (`deposit`)

### Step 1.1 Create crypto deposit

Use this endpoint to create a crypto deposit operation and generate a destination address. Use the response to provide deposit instructions and track the operation by transaction id.
**POST** `/api/v2/exchange/merchant/balance/crypto/deposit`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Merchant is not allowed to access account balances. |
| `500 Internal Server Error` | HTTP error | No | Accounting service or balance provider error. |

---

## 1) Deposit (`deposit`)

Deposit endpoints are used when the client adds funds to the custodial wallet. Crypto deposit returns a blockchain address, and fiat deposit returns payment data for provider processing.

### Step 1.1 Create Crypto Deposit

Use this endpoint to create a crypto deposit operation and receive the address where the client must send crypto funds.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/deposit`

>>>>>>> Stashed changes
**Request**

```json
{
  "clientId": "{{cleintId}}",
  "accountType": "WALLET",
  "asset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  }
}
```

**Response**

```json
{
  "transactionId": "e9b08950-ed34-4e78-88ca-5e74b22a125c",
  "depositCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the deposit operation is created. |
| `accountType` | `string` | Yes | Determines which internal balance account is affected. For custodial wallet operations use `WALLET`. |
| `asset.code` | `string` | Yes | Asset identifier used to create the operation; must match one of the assets returned by the assets endpoint. |
| `asset.network` | `string` | Yes | Blockchain network for address generation; prevents creating a deposit address for the wrong network. |
| `asset.amount` | `number` | Yes | Amount expected from the client; used for limits, display, and operation tracking. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | WhiteBird transaction identifier for tracking operation status, support cases, and reconciliation. |
| `depositCryptoAddress` | `string` | Yes | Blockchain address that must be shown to the client as the destination for crypto deposit. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 ACTIVE_DEPOSIT_REQUEST_FOUND` | Business error | No | An uncompleted deposit already exists for this client/asset. |
| `400 INVALID_AMOUNT` | Business error | No | Provided amount is invalid for deposit constraints. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to the merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for this operation. |


### Step 1.2 Get fiat payment methods

Use this endpoint to retrieve available fiat payment methods for the selected client and flow. Use the response to select a valid payment token for deposit or withdrawal requests.
**POST** `/api/v2/exchange/merchant/payment/method`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who creates the wallet deposit. |
| `accountType` | `string` | Yes | Target account type. For custodial wallet use `WALLET`. |
| `asset.code` | `string` | Yes | Asset to deposit, for example `USDT_TRC`. |
| `asset.network` | `string` | Yes | Blockchain network used for deposit address generation. |
| `asset.amount` | `number` | Yes | Expected deposit amount. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Crypto transaction id created in WhiteBird. Use it for tracking. |
| `depositCryptoAddress` | `string` | Yes | Blockchain address where the client sends crypto funds. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_ASSET` | Business error | No | Asset code/network is missing or unsupported. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client status does not allow wallet deposit. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client is not found or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 1.2 Get Fiat Payment Methods

Use this endpoint before fiat deposit or withdrawal to show the client available fiat payment instruments and providers.

**POST** `{{URL}}/api/v2/exchange/merchant/payment/method`

>>>>>>> Stashed changes
**Request**

```json
{
  "clientId": "{{cleintId}}",
  "fiatAsset": "BYN",
  "orderType": "BUY",
  "destination": "SDK_ACCOUNTING"
}
```

**Response**

```json
[
  {
    "id": "payment-token",
    "number": "**** **** **** 1111",
    "brand": "VISA",
    "providerId": "ASSIST",
    "providerType": "ASSIST",
    "status": "ENABLED",
    "isRestricted": false,
    "isCrypto": false,
    "country": "Belarus",
    "currency": "BYN",
    "supportedCurrencies": ["BYN"]
  }
]
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to return only payment methods available to this client. |
| `fiatAsset` | `string` | No | Fiat currency filter, for example `BYN`. |
| `orderType` | `string` | No | Operation type filter, for example `BUY` for fiat input. |
| `destination` | `string` | No | Flow filter. For custodial wallet use `SDK_ACCOUNTING`. |
| `providers` | `array of strings` | No | Optional list of allowed fiat providers. |
| `isCrypto` | `boolean` | No | Optional filter for crypto-related payment methods. |
| `countryGroup` | `string` | No | Optional country group filter. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Payment method token. Pass this value as `paymentToken` in fiat deposit/withdrawal or fiat-provider quote requests. |
| `number` | `string` | No | Masked payment method number shown to client. |
| `brand` | `string` | No | Payment method brand, for example `VISA`. |
| `providerId` | `string` | Yes | Provider identifier. |
| `providerType` | `string` | Yes | Provider type, for example `ASSIST`. |
| `status` | `string` | Yes | Payment method status. Use enabled methods only. |
| `isRestricted` | `boolean` | Yes | Shows whether this payment method is restricted. |
| `isCrypto` | `boolean` | Yes | Shows whether method is crypto-related. |
| `country` | `string` | No | Payment method country. |
| `currency` | `string` | No | Primary fiat currency. |
| `supportedCurrencies` | `array of strings` | No | Fiat currencies supported by this payment method. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to the merchant. |
| `400 INVALID_ORDER_TYPE` | Business error | No | `orderType` value is unsupported for payment method resolution. |
| `400 INVALID_FIAT_ASSET` | Business error | No | `fiatAsset` value is unsupported for the selected flow. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no `PAYMENT_API` permission for this client. |
| `429 Too Many Requests` | HTTP error | No | Rate limit is exceeded for payment methods endpoint. |


### Step 1.3 Create fiat deposit

Use this endpoint to initiate a fiat deposit through a selected payment provider. Use the response to redirect the client to provider payment flow or render payment details.
**POST** `/api/v2/exchange/merchant/balance/fiat/deposit`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client whose payment methods must be returned. |
| `fiatAsset` | `string` | No | Fiat currency filter, for example `BYN`. |
| `orderType` | `string` | No | Operation type filter, for example `BUY` for fiat input. |
| `destination` | `string` | No | Flow filter. For custodial wallet use `SDK_ACCOUNTING`. |
| `providers` | `array<string>` | No | Optional list of allowed fiat providers. |
| `isCrypto` | `boolean` | No | Optional filter for crypto-related payment methods. |
| `countryGroup` | `string` | No | Optional country group filter. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Payment token used later as `paymentToken`. |
| `number` | `string` | No | Masked payment method number shown to client. |
| `brand` | `string` | No | Payment method brand, for example `VISA`. |
| `providerId` | `string` | Yes | Provider identifier. |
| `providerType` | `string` | Yes | Provider type, for example `ASSIST`. |
| `status` | `string` | Yes | Payment method status. Use enabled methods only. |
| `isRestricted` | `boolean` | Yes | Shows whether this payment method is restricted. |
| `isCrypto` | `boolean` | Yes | Shows whether method is crypto-related. |
| `country` | `string` | No | Payment method country. |
| `currency` | `string` | No | Primary fiat currency. |
| `supportedCurrencies` | `array<string>` | No | Fiat currencies supported by this payment method. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 CLIENT_NOT_FOUND` | Business error | No | Client is not found or not linked to merchant. |
| `400 INVALID_PAYMENT_PROVIDER` | Business error | No | Provider filter is unsupported. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 1.3 Create Fiat Deposit

Use this endpoint to start a fiat deposit to the custodial wallet through a fiat provider.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/deposit`

>>>>>>> Stashed changes
**Request**

```json
{
  "clientId": "{{cleintId}}",
  "accountType": "WALLET",
  "fiatProviderType": "ASSIST",
  "paymentToken": "payment-token",
  "asset": {
    "code": "BYN",
    "amount": 100
  }
}
```

**Response**

```json
{
  "fiatPaymentLink": "https://payments.t.paysecure.ru/pay/p2p",
  "creationDate": "2026-04-30T09:45:15+0000",
  "expirationMinutes": 15,
  "paymentDetails": {
    "paymentLink": "https://payments.t.paysecure.ru/pay/p2p",
    "notificationPhoneNumber": null
  }
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the fiat wallet deposit is created. |
| `accountType` | `string` | Yes | Determines which internal balance account is affected. For custodial wallet operations use `WALLET`. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payment processing, for example `ASSIST`. |
| `paymentToken` | `string` | Conditional | Token of the client saved payment method. Use it to route fiat payment/payout through a selected card or payment instrument. Required if `internalToken` is not used. |
| `internalToken` | `string` | Conditional | Internal token alternative used when the payment instrument is represented by internal provider data instead of `paymentToken`. Required if `paymentToken` is not used. |
| `asset.code` | `string` | Yes | Fiat currency to deposit. |
| `asset.amount` | `number` | Yes | Fiat amount the client should deposit to the wallet. |

### Response
=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who deposits fiat to wallet. |
| `accountType` | `string` | Yes | Target account type. For custodial wallet use `WALLET`. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payment processing, for example `ASSIST`. |
| `paymentToken` | `string` | Conditional | Saved payment method token. Required if `internalToken` is not used. |
| `internalToken` | `string` | Conditional | Internal payment token. Required if `paymentToken` is not used. |
| `asset.code` | `string` | Yes | Fiat currency to deposit. |
| `asset.amount` | `number` | Yes | Deposit amount. |

#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatPaymentLink` | `string` | No | Payment URL that client should open to complete fiat deposit. |
| `creationDate` | `string` | Yes | Deposit creation date/time. |
| `expirationMinutes` | `number` | No | Payment link lifetime in minutes. |
| `paymentDetails` | `object` | No | Provider-specific payment data. |
| `paymentDetails.paymentLink` | `string` | No | Provider payment URL. |
<<<<<<< Updated upstream
| `paymentDetails.notificationPhoneNumber` | `string/null` | No | Phone number returned by provider when the payment scenario requires notification or additional confirmation. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | Business error | No | Fiat provider operation cannot be started or processed. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | `paymentToken`/`internalToken` is invalid, restricted, or missing. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for this operation. |


## 2) Send (`withdrawal`)

### Step 2.1 Calculate crypto withdrawal

Use this endpoint to calculate crypto withdrawal fees and net payout before submission. Use the response to show final amounts and keep the calculation id for withdrawal creation.
**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

### Headers
- `x-api-key: {{x-api-key}}`

=======
| `paymentDetails.notificationPhoneNumber` | `string \| null` | No | Phone number used for provider notifications when available. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | `paymentToken`/`internalToken` is missing, invalid, or unavailable. |
| `400 INVALID_FIAT_PROVIDER` | Business error | No | Provider is unsupported for this currency or flow. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot perform fiat deposit. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

---

## 2) Send (`withdrawal`)

Withdrawal endpoints are used when the client sends funds out of the custodial wallet. Crypto withdrawal requires calculation before creation, and fiat withdrawal uses a fiat provider/payment token.

### Step 2.1 Calculate Crypto Withdrawal

Use this endpoint to calculate fees and received amount before creating a crypto withdrawal.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "asset":{
        "amount":10,
        "code":"TRX",
        "network":"Tron"
    },
    "toAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"  // destination crypto withdrawal address
=======
  "clientId": "{{clientId}}",
  "asset": {
    "amount": 10,
    "code": "TRX",
    "network": "Tron"
  },
  "toAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
>>>>>>> Stashed changes
}
```

**Response**

```json
{
  "id": "92db12c2-bf7a-402e-995b-3c43f1e4eb77",
  "withdrawalAmount": "10",
  "commissionAmount": "0.263",
  "receivedAmount": "9.737",
  "expirationDate": "2026-04-30T09:51:19+0000"
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier whose wallet balance will be used for crypto withdrawal. |
| `asset.amount` | `number` | Yes | Amount to withdraw before commission. |
| `asset.code` | `string` | Yes | Crypto asset code. |
| `asset.network` | `string` | Yes | Blockchain network. |
| `toAddress` | `string` | Yes | Destination crypto address. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Calculation id used to create withdrawal. |
| `withdrawalAmount` | `string` | Yes | Original withdrawal amount. |
| `commissionAmount` | `string` | Yes | Network/service commission amount. |
| `receivedAmount` | `string` | Yes | Amount expected to be received after commission. |
| `expirationDate` | `string` | No | Date/time when this calculation expires. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_ADDRESS` | Business error | No | Destination address is invalid for selected network or blocked as internal address. |
| `400 INVALID_AMOUNT` | Business error | No | Amount is invalid (including fee greater than withdrawal amount). |
| `400 ACTIVE_WITHDRAWAL_REQUEST_FOUND` | Business error | No | An uncompleted withdrawal already exists for this client/asset. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for this operation. |


### Step 2.2 Create crypto withdrawal

Use this endpoint to create a crypto withdrawal using a valid calculation context. Use the response to store transaction id and monitor withdrawal lifecycle.
**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who withdraws crypto from wallet. |
| `asset.amount` | `number` | Yes | Amount to withdraw before commission. |
| `asset.code` | `string` | Yes | Crypto asset code. |
| `asset.network` | `string` | Yes | Blockchain network. |
| `toAddress` | `string` | Yes | Destination crypto address. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Calculation id used to create withdrawal. |
| `withdrawalAmount` | `string` | Yes | Original withdrawal amount. |
| `commissionAmount` | `string` | Yes | Network/service commission amount. |
| `receivedAmount` | `string` | Yes | Amount expected to be received after commission. |
| `expirationDate` | `string` | No | Date/time when this calculation expires. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_ADDRESS` | Business error | No | Destination address is invalid for the selected network. |
| `400 INVALID_AMOUNT` | Business error | No | Amount is below/above allowed limits or insufficient. |
| `400 INVALID_ASSET` | Business error | No | Asset or network is unsupported. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 2.2 Create Crypto Withdrawal

Use this endpoint after calculation to create the actual crypto withdrawal from wallet.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "accountType":"WALLET",
    "calculationId":"ea40bbbf-16a2-4fa2-aada-f55121c45eac",
    "comment":""  // MEMO/comment/TAG field, used as destination memo for networks like TON
=======
  "clientId": "{{clientId}}",
  "accountType": "WALLET",
  "calculationId": "ea40bbbf-16a2-4fa2-aada-f55121c45eac",
  "comment": ""
>>>>>>> Stashed changes
}
```

**Response**

```json
{
  "transactionId": "crypto-withdrawal-transaction-id"
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the withdrawal operation is created. |
| `accountType` | `string` | Yes | Determines from which internal balance account funds are debited. For custodial wallet use `WALLET`. |
| `calculationId` | `string` | Yes | Calculation id returned by withdrawal calculation endpoint. |
| `comment` | `string` | No | Optional memo/comment/tag for networks that require additional destination data. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Created crypto withdrawal transaction identifier used for tracking status and support. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_CALCULATION` | Business error | No | `calculationId` is not found or expired. |
| `400 INVALID_STATUS` | Business error | No | Withdrawal operation status does not allow execution. |
| `400 ACTIVE_WITHDRAWAL_REQUEST_FOUND` | Business error | No | Another uncompleted withdrawal blocks this operation. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Operation is forbidden for this merchant (`AccessDeniedException`). |


### Step 2.3 Calculate fiat withdrawal

Use this endpoint to calculate fiat withdrawal commission and expected payout amount. Use the response to confirm final withdrawal values with the client.
**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who creates withdrawal. |
| `accountType` | `string` | Yes | Source account type. For custodial wallet use `WALLET`. |
| `calculationId` | `string` | Yes | Calculation id returned by withdrawal calculation endpoint. |
| `comment` | `string` | No | Optional memo/comment/tag for networks that require additional destination data. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Created crypto withdrawal transaction id. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 CALCULATION_NOT_FOUND` | Business error | No | Calculation id is missing, expired, or not found. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot perform withdrawal. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Wallet balance is not enough for withdrawal. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 2.3 Calculate Fiat Withdrawal

Use this endpoint to calculate fiat withdrawal fee and received amount before creating payout.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

>>>>>>> Stashed changes
**Request**

```json
{
  "clientId": "{{cleintId}}",
  "fiatProviderType": "ASSIST",
  "paymentToken": "{{payment_token}}",
  "asset": {
    "code": "BYN",
    "amount": 100
  }
}
```

**Response**

```json
{
  "id": null,
  "withdrawalAmount": "100",
  "commissionAmount": "1.5",
  "receivedAmount": "98.5",
  "expirationDate": null
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier whose wallet fiat balance will be used for payout calculation. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payout. |
| `paymentToken` | `string` | Conditional | Payment method token for fiat withdrawal. |
| `internalToken` | `string` | Conditional | Internal token alternative. |
| `asset.code` | `string` | Yes | Fiat currency. |
| `asset.amount` | `number` | Yes | Fiat amount requested from the client wallet before provider fees are applied. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string/null` | No | Calculation identifier when provider creates a reusable calculation. Can be `null` when the fiat calculation is direct and no follow-up calculation id is required. |
| `withdrawalAmount` | `string` | Yes | Amount requested for withdrawal. |
| `commissionAmount` | `string` | Yes | Fiat withdrawal commission. |
| `receivedAmount` | `string` | Yes | Amount expected after commission. |
| `expirationDate` | `string/null` | No | Date/time until which the calculated fees and amounts are valid. `null` means the provider did not return an expiration time. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | Business error | No | Fiat withdrawal calculation cannot be produced by provider/flow. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token is invalid, unavailable, or unsupported. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for this operation. |


### Step 2.4 Create fiat withdrawal

Use this endpoint to create a fiat withdrawal from custodial wallet balance. Use the response to persist transaction id and track provider payout status.
**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who withdraws fiat. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payout. |
| `paymentToken` | `string` | Conditional | Payment method token for fiat withdrawal. |
| `internalToken` | `string` | Conditional | Internal token alternative. |
| `asset.code` | `string` | Yes | Fiat currency. |
| `asset.amount` | `number` | Yes | Fiat withdrawal amount. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string \| null` | No | Calculation id when provider requires it. Can be `null` for direct calculation flows. |
| `withdrawalAmount` | `string` | Yes | Amount requested for withdrawal. |
| `commissionAmount` | `string` | Yes | Fiat withdrawal commission. |
| `receivedAmount` | `string` | Yes | Amount expected after commission. |
| `expirationDate` | `string \| null` | No | Calculation expiration date when applicable. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token is missing or unavailable. |
| `400 INVALID_AMOUNT` | Business error | No | Amount is invalid or outside limits. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Wallet fiat balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 2.4 Create Fiat Withdrawal

Use this endpoint to create fiat withdrawal from wallet to selected fiat payment method.

**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "accountType": "WALLET",
    "fiatProviderType": "ASSIST",
    "paymentToken": "{{payment_token}}",
    "asset": {
        "code": "BYN",
        "amount": 100
    }
=======
  "clientId": "{{clientId}}",
  "accountType": "WALLET",
  "fiatProviderType": "ASSIST",
  "paymentToken": "{{payment_token}}",
  "asset": {
    "code": "BYN",
    "amount": 100
  }
>>>>>>> Stashed changes
}
```

**Response**

```json
{
  "transactionId": "fiat-withdrawal-transaction-id"
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the fiat withdrawal operation is created. |
| `accountType` | `string` | Yes | Determines from which internal balance account funds are debited. For custodial wallet use `WALLET`. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payout. |
| `paymentToken` | `string` | Conditional | Token of the client saved payment method. Use it to route fiat payment/payout through a selected card or payment instrument. Required if `internalToken` is not used. |
| `internalToken` | `string` | Conditional | Internal token alternative used when the payment instrument is represented by internal provider data instead of `paymentToken`. Required if `paymentToken` is not used. |
| `asset.code` | `string` | Yes | Fiat currency. |
| `asset.amount` | `number` | Yes | Withdrawal amount. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Created fiat withdrawal transaction identifier used for tracking payout status and reconciliation. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | Business error | No | Fiat withdrawal cannot be started or provider rejected operation. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token/internal token is invalid or restricted. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Operation is forbidden for this merchant (`AccessDeniedException`). |

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client who creates fiat withdrawal. |
| `accountType` | `string` | Yes | Source account type. For custodial wallet use `WALLET`. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payout. |
| `paymentToken` | `string` | Conditional | Saved payment method token. Required if `internalToken` is not used. |
| `internalToken` | `string` | Conditional | Internal payment token. Required if `paymentToken` is not used. |
| `asset.code` | `string` | Yes | Fiat currency. |
| `asset.amount` | `number` | Yes | Withdrawal amount. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Created fiat withdrawal transaction id. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token is missing, invalid, or restricted. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Wallet balance is not enough for withdrawal. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot perform fiat withdrawal. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

---
>>>>>>> Stashed changes

## 3) Buy (`buy`) - Merchant V3 Flow

Buy flow is used when the client pays fiat through a provider and receives crypto to internal wallet balance. The flow is quote first, then order creation.

### Step 3.1 Create Quote

Use this endpoint to calculate a buy quote before order creation. Quote fixes the rate, fees, input amount, output amount, and expiration time.

<<<<<<< Updated upstream
### Step 3.1 Create quote

Use this endpoint to create a buy quote and lock rate/amounts for a short time. Use the response to display final buy terms and pass quote id to order creation.
**POST** `/api/v3/exchange/merchant/quote`

### Headers
- `x-api-key: {{x-api-key}}`

=======
**POST** `{{URL}}/api/v3/exchange/merchant/quote`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "input":{
        "type":"FIAT_PROVIDER",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"BYN",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":50,                // amount
        "provider": "ASSIST",       // provider
        "token": "{{payment_token}}"       // payment token id
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
=======
  "clientId": "{{clientId}}",
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": 50,
    "provider": "ASSIST",
    "token": "{{payment_token}}"
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX"
  }
>>>>>>> Stashed changes
}
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "3cf9f5b7-1013-4769-b396-9eb28e6b408d",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9768",
    "exchangeRateValue": "0.9768",
    "actualRateValue": "1.0469",
    "clientId": "{{cleintId}}",
    "creationDate": "2026-04-30T11:28:17+0000",
    "expirationDate": "2026-04-30T11:28:47+0000",
    "input": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "50",
        "feeAmount": "3.35",
        "provider": "ASSIST",
        "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK"
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "47.757985",
        "feeAmount": "0"
    }
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to calculate quote limits, fees, and eligibility for this client. |
=======
  "id": "3cf9f5b7-1013-4769-b396-9eb28e6b408d",
  "rate": "TRX/BYN",
  "systemRateValue": "0.9768",
  "exchangeRateValue": "0.9768",
  "actualRateValue": "1.0469",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "creationDate": "2026-04-30T11:28:17+0000",
  "expirationDate": "2026-04-30T11:28:47+0000",
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": "50",
    "feeAmount": "3.35",
    "provider": "ASSIST",
    "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
    "paymentType": "P2P",
    "processingBank": "BELARUSBANK"
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "47.757985",
    "feeAmount": "0"
  }
}
```

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom quote is created. |
>>>>>>> Stashed changes
| `input.type` | `string` | Yes | Source operation type. For buy use `FIAT_PROVIDER`. |
| `input.asset` | `string` | Yes | Source fiat asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `input.provider` | `string` | Yes | Fiat provider used for payment. |
| `input.token` | `string` | Conditional | Payment token used for provider payment. |
| `output.type` | `string` | Yes | Destination operation type. For custodial wallet use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Crypto asset that will be credited to internal balance. |
| `output.amount` | `number` | Conditional | Target amount. At least one of `input.amount` or `output.amount` is required. |

<<<<<<< Updated upstream
### Response
=======
#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used to create order. |
| `rate` | `string` | Yes | Currency pair displayed for the quote. |
| `systemRateValue` | `string` | Yes | Base system rate before merchant/customer fee effects. |
| `exchangeRateValue` | `string` | Yes | Exchange rate applied to the quote. |
| `actualRateValue` | `string` | Yes | Actual resulting rate for displayed amounts. |
| `clientId` | `string` | Yes | Client id for the quote. |
| `creationDate` | `string` | Yes | Quote creation date/time. |
| `expirationDate` | `string` | Yes | Quote expiration date/time. |
| `input` | `object` | Yes | Calculated source payment details. |
| `output` | `object` | Yes | Calculated destination payment details. |
| `input.feeAmount` / `output.feeAmount` | `string` | Yes | Fee amount for each side of the operation. |
| `input.paymentType` | `string` | No | Fiat payment type selected by provider configuration. |
| `input.processingBank` | `string` | No | Processing bank selected for fiat provider route. |

<<<<<<< Updated upstream
### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote input is inconsistent or cannot be calculated for provided payment details. |
| `400 CURRENCY_NOT_FOUND` | Business error | No | One of input/output assets is unknown. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Provided client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no `ORDER_V2_API` permission. |
| `429 Too Many Requests` | HTTP error | No | Rate limit is exceeded for quote creation endpoint. |


### Step 3.2 Create buy order

Use this endpoint to create a buy order from a valid non-expired quote. Use the response to track order execution and render provider/payment metadata.
**POST** `/api/v3/exchange/merchant/order`

### Headers
- `x-api-key: {{x-api-key}}`
=======
#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote request cannot be calculated with provided assets, amount, or payment details. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot create quote for this operation. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token is invalid or restricted. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 3.2 Create Buy Order

Use this endpoint to create a buy order from a valid non-expired quote.

**POST** `{{URL}}/api/v3/exchange/merchant/order`
>>>>>>> Stashed changes

**Request**

```json
{
  "quoteId": "47b2985a-2fe3-427c-9a18-6b16736c460e"
}
<<<<<<< Updated upstream
// exchange operation is processed immediately
=======
>>>>>>> Stashed changes
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "d938165d-2158-4f4e-8bf1-9ef6c5806fdc",
    "number": 721000004148,
    "conditions": {
        "fromAsset": "BYN",
        "toAsset": "TRX",
        "fromGrossAmount": "50",
        "fromNetAmount": "46.65",
        "fromFeeAmount": "3.35",
        "toGrossAmount": "47.757985",
        "toNetAmount": "47.757985",
        "toFeeAmount": "0",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9768",
        "exchangeRateValue": "0.9768",
        "actualRateValue": "1.0469"
    },
    "recalculationReason": null,
    "clientId": "{{cleintId}}",
=======
  "id": "d938165d-2158-4f4e-8bf1-9ef6c5806fdc",
  "number": 721000004148,
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "status": "PROCESSING",
  "failureMessage": null,
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": "50",
>>>>>>> Stashed changes
    "status": "PROCESSING",
    "provider": "ASSIST",
    "paymentType": "P2P",
    "processingBank": "BELARUSBANK",
    "link": "https://payments.t.paysecure.ru/pay/p2p/..."
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "47.757985",
    "status": "NEW"
  }
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Quote identifier returned by quote creation. It fixes the calculated amounts/rates and must be used before quote expiration. |

### Response
=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Quote id returned by quote creation endpoint. |

#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full order calculation details. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Order status, for example `PROCESSING`, `COMPLETED`, `FAILED`. |
<<<<<<< Updated upstream
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `input` | `object` | Yes | Source operation details. |
| `output` | `object` | Yes | Destination operation details. |
| `input.link` | `string/null` | No | Fiat provider payment URL that should be opened by the client when the payment flow requires redirect/link confirmation. |
| `processorTransactionId` | `string/null` | No | External provider transaction identifier used to reconcile WhiteBird operation with fiat provider processing. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for order creation. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client status/checks do not allow order creation (for example testing not completed). |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this order. |

=======
| `failureMessage` | `string \| null` | No | Error reason when order fails. |
| `input` | `object` | Yes | Source operation details. |
| `output` | `object` | Yes | Destination operation details. |
| `input.link` | `string \| null` | No | Fiat provider payment link when payment requires redirect/link. |
| `processorTransactionId` | `string \| null` | No | Provider transaction id when available. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or not found. |
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be used to create order. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client is not allowed to create order. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

---
>>>>>>> Stashed changes

## 4) Sell (`sell`) - Merchant V3 Flow

Sell flow is used when the client sells crypto from internal wallet balance and receives fiat through a provider.

### Step 4.1 Create Quote

Use this endpoint to calculate a sell quote before creating the order.

<<<<<<< Updated upstream
### Step 4.1 Create quote

Use this endpoint to create a sell quote and lock rate/amounts for sell flow. Use the response to show sell terms and pass quote id to order creation.
**POST** `/api/v3/exchange/merchant/quote`

### Headers
- `x-api-key: {{x-api-key}}`

=======
**POST** `{{URL}}/api/v3/exchange/merchant/quote`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "input":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX",
        "amount":100
    },
    "output":{
        "type":"FIAT_PROVIDER",
        "asset":"BYN",
        "provider": "ASSIST",
        "token": "{{payment_token}}"
    }
=======
  "clientId": "{{clientId}}",
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": 100
  },
  "output": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "provider": "ASSIST",
    "token": "{{payment_token}}"
  }
>>>>>>> Stashed changes
}
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "a95bf590-c029-47b2-bf95-adbcf50a11bb",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9765",
    "exchangeRateValue": "0.9765",
    "actualRateValue": "0.9208",
    "clientId": "{{cleintId}}",
    "creationDate": "2026-04-30T11:38:25+0000",
    "expirationDate": "2026-04-30T11:38:55+0000",
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "feeAmount": "0"
    },
    "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "92.08",
        "feeAmount": "5.57",
        "provider": "ASSIST",
        "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK"
    }
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to calculate sell quote, fees, and wallet-balance eligibility. |
=======
  "id": "a95bf590-c029-47b2-bf95-adbcf50a11bb",
  "rate": "TRX/BYN",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "expirationDate": "2026-04-30T11:38:55+0000",
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "100",
    "feeAmount": "0"
  },
  "output": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": "92.08",
    "feeAmount": "5.57",
    "provider": "ASSIST",
    "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
    "paymentType": "P2P",
    "processingBank": "BELARUSBANK"
  }
}
```

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom sell quote is created. |
>>>>>>> Stashed changes
| `input.type` | `string` | Yes | Source operation type. For custodial wallet sell use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Crypto asset being sold. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `output.type` | `string` | Yes | Destination operation type. For fiat payout use `FIAT_PROVIDER`. |
| `output.asset` | `string` | Yes | Fiat asset to receive. |
| `output.provider` | `string` | Yes | Fiat provider. |
| `output.token` | `string` | Conditional | Payment token for receiving fiat. |

<<<<<<< Updated upstream
### Response
=======
#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used for order creation. |
| `rate` | `string` | Yes | Currency pair. |
| `clientId` | `string` | Yes | Client id for the quote. |
| `expirationDate` | `string` | Yes | Quote expiration date/time. |
| `input` | `object` | Yes | Calculated source operation details. |
| `output` | `object` | Yes | Calculated destination operation details. |
| `output.amount` | `string` | Yes | Fiat amount expected before/after fee according to response details. |
| `output.feeAmount` | `string` | Yes | Fiat provider/exchange fee amount. |
| `output.paymentType` | `string` | No | Fiat payment type. |
| `output.processingBank` | `string` | No | Processing bank selected by provider route. |

<<<<<<< Updated upstream
### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | Business error | No | Input/output asset is unknown. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no `ORDER_V2_API` permission. |


### Step 4.2 Create sell order

Use this endpoint to create a sell order from a valid non-expired quote. Use the response to track order progress and payout operation details.
**POST** `/api/v3/exchange/merchant/order`

### Headers
- `x-api-key: {{x-api-key}}`
=======
#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be calculated. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Client wallet balance is not enough. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Fiat payment token is invalid or restricted. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 4.2 Create Sell Order

Use this endpoint to create a sell order from a valid quote.

**POST** `{{URL}}/api/v3/exchange/merchant/order`
>>>>>>> Stashed changes

**Request**

```json
{
  "quoteId": "a95bf590-c029-47b2-bf95-adbcf50a11bb"
}
<<<<<<< Updated upstream
// exchange operation is processed immediately
=======
>>>>>>> Stashed changes
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "2bf54839-b540-452b-9014-3ba9d32a1e93",
    "number": 161000004149,
    "conditions": {
        "fromAsset": "TRX",
        "toAsset": "BYN",
        "fromGrossAmount": "100",
        "fromNetAmount": "100",
        "fromFeeAmount": "0",
        "toGrossAmount": "97.65",
        "toNetAmount": "92.08",
        "toFeeAmount": "5.57",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9765",
        "exchangeRateValue": "0.9765",
        "actualRateValue": "0.9208"
    },
    "recalculationReason": null,
    "clientId": "{{cleintId}}",
    "status": "PROCESSING",
    "failureMessage": null,
    "completionDate": null,
    "creationDate": "2026-04-30T11:40:23+0000",
    "sessionId": null,
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "transactionAmount": "100",
        "feeAmount": "0",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    },
    "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "97.65",
        "transactionAmount": "92.08",
        "feeAmount": "5.57",
        "status": "NEW",
        "failureMessage": null,
        "expirationDate": null,
        "provider": "ASSIST",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK",
        "clientBank": null,
        "fromToken": "97fe9aa7-7805-438f-8c5e-aea24b4f9dc4",
        "toToken": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "link": null,
        "processorTransactionId": "f42bb8b78d814be48f0256a96c2208ac",
        "post": null,
        "paymentSystem": null
    }
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Sell quote identifier returned by quote creation. It fixes the calculated sell amounts/rates and must be used before expiration. |

### Response
=======
  "id": "2bf54839-b540-452b-9014-3ba9d32a1e93",
  "number": 161000004149,
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "status": "PROCESSING",
  "failureMessage": null,
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "100",
    "status": "COMPLETED"
  },
  "output": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": "97.65",
    "status": "NEW",
    "provider": "ASSIST",
    "paymentType": "P2P",
    "processingBank": "BELARUSBANK"
  }
}
```

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Quote id returned by sell quote creation. |

#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full quote/order calculation details. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Current order status. |
<<<<<<< Updated upstream
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
=======
| `failureMessage` | `string \| null` | No | Failure reason when order fails. |
>>>>>>> Stashed changes
| `input` | `object` | Yes | Source crypto/internal balance operation. |
| `output` | `object` | Yes | Destination fiat provider operation. |
| `output.provider` | `string` | No | Fiat provider used for payout. |
| `output.processingBank` | `string` | No | Processing bank selected by fiat provider route. |
<<<<<<< Updated upstream
| `processorTransactionId` | `string/null` | No | External provider transaction identifier used to reconcile WhiteBird operation with fiat provider processing. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for sell order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Client internal balance is not enough for requested sell operation. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this order. |

=======
| `processorTransactionId` | `string \| null` | No | Provider transaction id when available. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or not found. |
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be used for order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

---
>>>>>>> Stashed changes

## 5) Operation History / Details

History endpoint is used to fetch created orders and their operation details for support, reconciliation, and client-facing history.

### Step 5.1 Get Order History / Details

Use this endpoint to search merchant orders by client, operation type, status, asset, or date range.

<<<<<<< Updated upstream
### Step 5.1 Get order history/details

Use this endpoint to fetch paged order history with optional filters and detailed operation data. Use the response to power history UI, reporting, and support investigations.
**POST** `/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`

### Headers
- `x-api-key: {{x-api-key}}`
=======
**POST** `{{URL}}/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`
>>>>>>> Stashed changes

**Request**

```json
{
  "clientIds": [
    "{{clientId}}"
  ]
}
```

**Response**

```json
{
  "content": [
    {
      "id": "2bf54839-b540-452b-9014-3ba9d32a1e93",
      "number": 161000004149,
<<<<<<< Updated upstream
      "conditions": {
        "fromAsset": "TRX",
        "toAsset": "BYN",
        "fromGrossAmount": "100",
        "fromNetAmount": "100",
        "fromFeeAmount": "0",
        "toGrossAmount": "97.65",
        "toNetAmount": "92.08",
        "toFeeAmount": "5.57",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9765",
        "exchangeRateValue": "0.9765",
        "actualRateValue": "0.9208"
      },
      "recalculationReason": null,
      "clientId": "{{cleintId}}",
=======
      "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
>>>>>>> Stashed changes
      "status": "PROCESSING",
      "failureMessage": null,
      "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "status": "COMPLETED"
      },
      "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "97.65",
        "status": "NEW",
        "provider": "ASSIST"
      }
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "number": 0,
  "size": 20
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `page` | `number` | No | Query parameter with page index. Default example uses `0`. |
| `size` | `number` | No | Query parameter with page size. Default example uses `20`. |
| `sort` | `string` | No | Query parameter for sorting, for example `creationDate,desc`. |
| `clientIds` | `array of strings` | No | Filter by one or more client ids. |
| `operationTypes` | `array of strings` | No | Filter by operation type: `FIAT_PROVIDER`, `CRYPTO_TRANSFER`, `INTERNAL_BALANCE`. |
| `statuses` | `array of strings` | No | Filter by order status: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `assets` | `array of strings` | No | Filter by assets. |
| `completionDateFrame.start` | `string` | No | Completion date range start. |
| `completionDateFrame.end` | `string` | No | Completion date range end. |
| `creationDateFrame.start` | `string` | No | Creation date range start. |
| `creationDateFrame.end` | `string` | No | Creation date range end. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `content` | `array of objects` | Yes | Page content with order objects. |
| `content[].id` | `string` | Yes | Order id. |
| `content[].number` | `number` | Yes | Human-readable order number. |
| `content[].conditions` | `object` | No | Full order calculation details. |
| `content[].clientId` | `string` | Yes | Client id. |
| `content[].status` | `string` | Yes | Current order status. |
| `content[].failureMessage` | `string/null` | No | Human-readable reason of failure for historical orders; useful for support and merchant-side audit. |
| `content[].input` | `object` | Yes | Source operation details. |
| `content[].output` | `object` | Yes | Destination operation details. |
| `totalElements` | `number` | Yes | Total number of matching orders. |
| `totalPages` | `number` | Yes | Total number of pages. |
| `number` | `number` | Yes | Current page number. |
| `size` | `number` | Yes | Current page size. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 Bad Request` | HTTP error | No | Filter body is invalid or missing required client filter (`Client id is required`). |
| `400 CLIENT_NOT_FOUND` | Business error | No | Provided client id/external client id is invalid for this merchant. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no permission for order history scope. |
| `429 Too Many Requests` | HTTP error | No | Rate limit is exceeded for history endpoint. |


## 6) Conversion (`conversion`)

For custodial wallet, conversion goes through internal balance (`USER_BALANCE` / `INTERNAL_BALANCE`).

### Step 6.1 Check limits

Use this endpoint to check conversion min/max limits for the selected asset pair. Use the response to validate entered amounts before quote creation.
**POST** `/api/v3/exchange/merchant/limit`

### Headers
- `x-api-key: {{x-api-key}}`
=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `page` | `number` | No | Query parameter with page index. Default example uses `0`. |
| `size` | `number` | No | Query parameter with page size. Default example uses `20`. |
| `sort` | `string` | No | Query parameter for sorting, for example `creationDate,desc`. |
| `clientIds` | `array<string>` | No | Filter by one or more client ids. |
| `operationTypes` | `array<string>` | No | Filter by operation type: `FIAT_PROVIDER`, `CRYPTO_TRANSFER`, `INTERNAL_BALANCE`. |
| `statuses` | `array<string>` | No | Filter by order status: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `assets` | `array<string>` | No | Filter by assets. |
| `completionDateFrame.start` | `string` | No | Completion date range start. |
| `completionDateFrame.end` | `string` | No | Completion date range end. |
| `creationDateFrame.start` | `string` | No | Creation date range start. |
| `creationDateFrame.end` | `string` | No | Creation date range end. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `content` | `array<object>` | Yes | Page content with order objects. |
| `content[].id` | `string` | Yes | Order id. |
| `content[].number` | `number` | Yes | Human-readable order number. |
| `content[].conditions` | `object` | No | Full order calculation details. |
| `content[].clientId` | `string` | Yes | Client id. |
| `content[].status` | `string` | Yes | Current order status. |
| `content[].failureMessage` | `string \| null` | No | Failure reason when order failed. |
| `content[].input` | `object` | Yes | Source operation details. |
| `content[].output` | `object` | Yes | Destination operation details. |
| `totalElements` | `number` | Yes | Total number of matching orders. |
| `totalPages` | `number` | Yes | Total number of pages. |
| `number` | `number` | Yes | Current page number. |
| `size` | `number` | Yes | Current page size. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_FILTER` | Business error | No | Filter value, date range, or pagination parameter is invalid. |
| `400 CLIENT_NOT_FOUND` | Business error | No | One of provided clients is not found or not linked to merchant. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

---

## 6) Conversion (`conversion`) - Merchant V3 Flow

Conversion uses internal balance on both sides. The client converts one custodial wallet asset into another through limit check, quote creation, and order creation.

### Step 6.1 Check Limits

Use this endpoint to check allowed min/max conversion amounts before creating quote.

**POST** `{{URL}}/api/v3/exchange/merchant/limit`
>>>>>>> Stashed changes

**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "fromAsset": "USDT_TRC",
    "fromPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    },
    "toAsset": "TRX",
    "toPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    }
=======
  "clientId": "{{clientId}}",
  "fromAsset": "USDT_TRC",
  "fromPaymentDetails": {
    "type": "INTERNAL_BALANCE"
  },
  "toAsset": "TRX",
  "toPaymentDetails": {
    "type": "INTERNAL_BALANCE"
  }
>>>>>>> Stashed changes
}
```

**Response**

```json
{
  "fromMinAmount": "0.00515231",
  "fromMaxAmount": "6.43319589",
  "toMinAmount": "947.37",
  "toMaxAmount": "1182894.74"
}
```

<<<<<<< Updated upstream
### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to apply client-specific limits before quote creation. |
| `fromAsset` | `string` | Yes | Source asset. |
| `fromPaymentDetails.type` | `string` | Yes | Source payment type. For conversion use `INTERNAL_BALANCE`. |
| `toAsset` | `string` | Yes | Destination asset. |
| `toPaymentDetails.type` | `string` | Yes | Destination payment type. For conversion use `INTERNAL_BALANCE`. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fromMinAmount` | `string` | Yes | Minimum allowed source amount. |
| `fromMaxAmount` | `string` | Yes | Maximum allowed source amount. |
| `toMinAmount` | `string` | Yes | Minimum allowed destination amount. |
| `toMaxAmount` | `string` | Yes | Maximum allowed destination amount. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_MERCHANT_ID` | Business error | No | No merchant identifier could be resolved for limit calculation context. |
| `400 INVALID_QUOTE` | Business error | No | Limit request contains invalid pair/payment details for calculation context. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no `LIMIT_V2_API` permission. |
| `429 Too Many Requests` | HTTP error | No | Rate limit is exceeded for limit endpoint. |


### Step 6.2 Create quote

Use this endpoint to create a conversion quote between internal balance assets. Use the response to show conversion terms and pass quote id to swap creation.
**POST** `/api/v3/exchange/merchant/quote`

### Headers
- `x-api-key: {{x-api-key}}`

=======
#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom limits are checked. |
| `fromAsset` | `string` | Yes | Source asset. |
| `fromPaymentDetails.type` | `string` | Yes | Source payment type. For conversion use `INTERNAL_BALANCE`. |
| `toAsset` | `string` | Yes | Destination asset. |
| `toPaymentDetails.type` | `string` | Yes | Destination payment type. For conversion use `INTERNAL_BALANCE`. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fromMinAmount` | `string` | Yes | Minimum allowed source amount. |
| `fromMaxAmount` | `string` | Yes | Maximum allowed source amount. |
| `toMinAmount` | `string` | Yes | Minimum allowed destination amount. |
| `toMaxAmount` | `string` | Yes | Maximum allowed destination amount. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 LIMIT_NOT_FOUND` | Business error | No | Limit configuration is missing for selected route. |
| `400 INVALID_CURRENCY_PAIR` | Business error | No | Asset pair is unsupported. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot perform conversion. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 6.2 Create Quote

Use this endpoint to calculate a conversion quote between two internal balance assets.

**POST** `{{URL}}/api/v3/exchange/merchant/quote`

>>>>>>> Stashed changes
**Request**

```json
{
<<<<<<< Updated upstream
    "clientId": "{{cleintId}}",
    "input":{
        "type":"INTERNAL_BALANCE",
        "asset":"USDT_TRC",
        "amount":5
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
=======
  "clientId": "{{clientId}}",
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "USDT_TRC",
    "amount": 5
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX"
  }
>>>>>>> Stashed changes
}
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "601b24b6-c7c3-4205-8396-79903f76f25e",
    "rate": "TRX/USDT_TRC",
    "systemRateValue": "0.3255",
    "exchangeRateValue": "0.3255",
    "actualRateValue": "0.3305",
    "clientId": "{{cleintId}}",
    "creationDate": "2026-04-30T11:05:37+0000",
    "expirationDate": "2026-04-30T11:06:07+0000",
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "USDT_TRC",
        "amount": "5",
        "feeAmount": "0"
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "15.130568",
        "feeAmount": "0.230415"
    }
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier used to calculate quote limits, fees, and eligibility for this client. |
=======
  "id": "601b24b6-c7c3-4205-8396-79903f76f25e",
  "rate": "TRX/USDT_TRC",
  "systemRateValue": "0.3255",
  "exchangeRateValue": "0.3255",
  "actualRateValue": "0.3305",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "expirationDate": "2026-04-30T11:06:07+0000",
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "USDT_TRC",
    "amount": "5",
    "feeAmount": "0"
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "15.130568",
    "feeAmount": "0.230415"
  }
}
```

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom quote is created. |
>>>>>>> Stashed changes
| `input.type` | `string` | Yes | Source type. For conversion use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Source asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one side amount is required. |
| `output.type` | `string` | Yes | Destination type. For conversion use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Destination asset. |
| `output.amount` | `number` | Conditional | Destination amount. At least one side amount is required. |

<<<<<<< Updated upstream
### Response
=======
#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used for conversion order. |
| `rate` | `string` | Yes | Conversion pair. |
| `systemRateValue` | `string` | Yes | Base system rate. |
| `exchangeRateValue` | `string` | Yes | Exchange rate applied to quote. |
| `actualRateValue` | `string` | Yes | Actual resulting rate. |
| `clientId` | `string` | Yes | Client id. |
| `expirationDate` | `string` | Yes | Quote expiration date/time. |
| `input` | `object` | Yes | Calculated source details. |
| `output` | `object` | Yes | Calculated destination details. |
| `feeAmount` | `string` | Yes | Fee amount on source/destination side. |

<<<<<<< Updated upstream
### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | Business error | No | Asset id is unknown for conversion pair. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client status/checks do not allow conversion quote creation. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no `ORDER_V2_API` permission. |


### Step 6.3 Create swap operation

Use this endpoint to create and execute a swap operation from a valid conversion quote. Use the response to persist order identifiers and final operation statuses.
**POST** `/api/v3/exchange/merchant/order`

### Headers
- `x-api-key: {{x-api-key}}`
=======
#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be calculated. |
| `400 AMOUNT_OUT_OF_LIMIT` | Business error | No | Amount is outside allowed min/max. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 6.3 Create Swap Operation

Use this endpoint to execute conversion from a valid quote. For internal-balance conversion, the order can be completed immediately when all checks pass.

**POST** `{{URL}}/api/v3/exchange/merchant/order`
>>>>>>> Stashed changes

**Request**

```json
{
  "quoteId": "601b24b6-c7c3-4205-8396-79903f76f25e"
}
```

**Response**

```json
{
<<<<<<< Updated upstream
    "id": "08d5b13c-5a5b-470b-b244-6a6becb7888b",
    "number": 821000004152,
    "conditions": {
        "fromAsset": "USDT_TRC",
        "toAsset": "TRX",
        "fromGrossAmount": "5",
        "fromNetAmount": "5",
        "fromFeeAmount": "0",
        "toGrossAmount": "15.346839",
        "toNetAmount": "15.116636",
        "toFeeAmount": "0.230203",
        "promoCode": null,
        "rate": "TRX/USDT_TRC",
        "systemRateValue": "0.3258",
        "exchangeRateValue": "0.3258",
        "actualRateValue": "0.3308"
    },
    "recalculationReason": null,
    "clientId": "{{cleintId}}",
    "status": "COMPLETED",
    "failureMessage": null,
    "completionDate": "2026-04-30T13:10:38+0000",
    "creationDate": "2026-04-30T13:10:35+0000",
    "sessionId": null,
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "USDT_TRC",
        "amount": "5",
        "transactionAmount": "5",
        "feeAmount": "0",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "15.116636",
        "transactionAmount": "15.116636",
        "feeAmount": "0.230203",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    }
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Conversion quote identifier returned by quote creation. It fixes the conversion rate and amounts until expiration. |

### Response
=======
  "id": "08d5b13c-5a5b-470b-b244-6a6becb7888b",
  "number": 821000004152,
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "status": "COMPLETED",
  "failureMessage": null,
  "input": {
    "type": "INTERNAL_BALANCE",
    "asset": "USDT_TRC",
    "amount": "5",
    "status": "COMPLETED"
  },
  "output": {
    "type": "INTERNAL_BALANCE",
    "asset": "TRX",
    "amount": "15.116636",
    "status": "COMPLETED"
  }
}
```

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Quote id returned by conversion quote endpoint. |

#### Response
>>>>>>> Stashed changes

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Conversion order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full conversion calculation details. |
| `clientId` | `string` | Yes | Client id. |
| `status` | `string` | Yes | Conversion order status. Usually `COMPLETED` when internal swap succeeds. |
<<<<<<< Updated upstream
| `failureMessage` | `string/null` | No | Human-readable reason of failure when conversion cannot be completed; use for support/debugging. |
=======
| `failureMessage` | `string \| null` | No | Failure reason when conversion fails. |
>>>>>>> Stashed changes
| `input` | `object` | Yes | Source internal balance operation. |
| `output` | `object` | Yes | Destination internal balance operation. |
| `input.status` / `output.status` | `string` | Yes | Status of each operation leg. |

<<<<<<< Updated upstream
### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for conversion order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Source internal balance is not enough to execute conversion. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this conversion. |


=======
#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or not found. |
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be used for conversion. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Source internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |
>>>>>>> Stashed changes
