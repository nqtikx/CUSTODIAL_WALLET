# CUSTODIAL_WALLET

The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.  
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.  

> BASE_URL https://api.dev.wbdevel.net

## 0) Wallet Base Data

Base data endpoints are used before any wallet operation. They help the merchant show available assets, current operations, and account balances to the client.

### Step 0.1 Get available assets

Use this endpoint to retrieve all fiat and crypto assets available for custodial wallet operations. Use the response to build asset selectors and validate supported routes before any operation.
**POST** `/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`

### Headers
- `x-api-key: {{x-api-key}}`

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
| `number` | `number` | Yes | Human-readable operation number. |
| `accountType` | `string` | Yes | Account scope for the operation. Allowed values: `WALLET`, `TRADING`, `SPOT`, `NONE`. Use `WALLET` for custodial wallet endpoints. |
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
  "clientId": "{{clientId}}"
}
```

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

**Request**

```json
{
  "clientId": "{{clientId}}",
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

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the deposit operation is created. |
| `accountType` | `string` | Yes | Account scope for balance mutation. Allowed values: `WALLET`, `TRADING`, `SPOT`, `NONE`. Use `WALLET` for custodial wallet operations. |
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

**Request**

```json
{
  "clientId": "{{clientId}}",
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

**Request**

```json
{
  "clientId": "{{clientId}}",
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

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the fiat wallet deposit is created. |
| `accountType` | `string` | Yes | Account scope for balance mutation. Allowed values: `WALLET`, `TRADING`, `SPOT`, `NONE`. Use `WALLET` for custodial wallet operations. |
| `fiatProviderType` | `string` | Yes | Fiat provider used for payment processing, for example `ASSIST`. |
| `paymentToken` | `string` | Conditional | Token of the client saved payment method. Use it to route fiat payment/payout through a selected card or payment instrument. Required if `internalToken` is not used. |
| `internalToken` | `string` | Conditional | Internal token alternative used when the payment instrument is represented by internal provider data instead of `paymentToken`. Required if `paymentToken` is not used. |
| `asset.code` | `string` | Yes | Fiat currency to deposit. |
| `asset.amount` | `number` | Yes | Fiat amount the client should deposit to the wallet. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatPaymentLink` | `string` | No | Payment URL that client should open to complete fiat deposit. |
| `creationDate` | `string` | Yes | Deposit creation date/time. |
| `expirationMinutes` | `number` | No | Payment link lifetime in minutes. |
| `paymentDetails` | `object` | No | Provider-specific payment data. |
| `paymentDetails.paymentLink` | `string` | No | Provider payment URL. |
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

**Request**

```json
{
    "clientId": "{{clientId}}",
    "asset":{
        "amount":10,
        "code":"TRX",
        "network":"Tron"
    },
    "toAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
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

**Request**

```json
{
    "clientId": "{{clientId}}",
    "accountType":"WALLET",
    "calculationId":"ea40bbbf-16a2-4fa2-aada-f55121c45eac",
    "comment":""
}
```

**Response**

```json
{
  "transactionId": "crypto-withdrawal-transaction-id"
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the withdrawal operation is created. |
| `accountType` | `string` | Yes | Source account scope for debit operation. Allowed values: `WALLET`, `TRADING`, `SPOT`, `NONE`. Use `WALLET` in custodial wallet flow. |
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

**Request**

```json
{
  "clientId": "{{clientId}}",
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

**Request**

```json
{
    "clientId": "{{clientId}}",
    "accountType": "WALLET",
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
  "transactionId": "fiat-withdrawal-transaction-id"
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | WhiteBird client identifier for whom the fiat withdrawal operation is created. |
| `accountType` | `string` | Yes | Source account scope for debit operation. Allowed values: `WALLET`, `TRADING`, `SPOT`, `NONE`. Use `WALLET` in custodial wallet flow. |
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


## 3) Buy (`buy`) - Merchant V3 Flow

Buy flow is used when the client pays fiat through a provider and receives crypto to internal wallet balance. The flow is quote first, then order creation.

### Step 3.1 Create Quote

Use this endpoint to calculate a buy quote before order creation. Quote fixes the rate, fees, input amount, output amount, and expiration time.

### Step 3.1 Create quote

Use this endpoint to create a buy quote and lock rate/amounts for a short time. Use the response to display final buy terms and pass quote id to order creation.
**POST** `/api/v3/exchange/merchant/quote`

### Headers
- `x-api-key: {{x-api-key}}`

**Request**

```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"FIAT_PROVIDER",
        "asset":"BYN",
        "amount":50,
        "provider": "ASSIST",
        "token": "{{payment_token}}"
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
}
```

**Response**

```json
{
    "id": "3cf9f5b7-1013-4769-b396-9eb28e6b408d",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9768",
    "exchangeRateValue": "0.9768",
    "actualRateValue": "1.0469",
    "clientId": "{{clientId}}",
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
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For buy, use `FIAT_PROVIDER`. |
| `input.asset` | `string` | Yes | Source fiat asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `input.provider` | `string` | Yes | Fiat provider used for payment. |
| `input.token` | `string` | Conditional | Payment token used for provider payment. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For buy destination, use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Crypto asset that will be credited to internal balance. |
| `output.amount` | `number` | Conditional | Target amount. At least one of `input.amount` or `output.amount` is required. |

### Response

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

**Request**

```json
{
  "quoteId": "47b2985a-2fe3-427c-9a18-6b16736c460e"
}
```

**Response**

```json
{
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
    "clientId": "{{clientId}}",
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

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `quoteId` | `string` | Yes | Quote identifier returned by quote creation. It fixes the calculated amounts/rates and must be used before quote expiration. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full order calculation details. |
| `conditions.fromAsset` | `string` | No | Source asset code from the quote used to create order. |
| `conditions.toAsset` | `string` | No | Destination asset code from the quote used to create order. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount after source-side fee application. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount included in final settlement. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount after destination-side fee application. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount included in final settlement. |
| `conditions.rate` | `string` | No | Rate pair used for conversion calculation. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate applied for this quote/order. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Current order lifecycle state. Typical values: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `input` | `object` | Yes | Source operation details. |
| `output` | `object` | Yes | Destination operation details. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Typical values: `NEW`, `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type (for example `P2P`, `SBP`, card payout type). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank used by fiat-provider route. |
| `input.clientBank` / `output.clientBank` | `string/null` | No | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string/null` | No | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string/null` | No | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |
| `input.post` / `output.post` | `string/null` | No | Additional provider payload or form-POST metadata when present. |
| `input.paymentSystem` / `output.paymentSystem` | `string/null` | No | Payment system metadata returned by provider integration. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for order creation. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client status/checks do not allow order creation (for example testing not completed). |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this order. |


## 4) Sell (`sell`) - Merchant V3 Flow

Sell flow is used when the client sells crypto from internal wallet balance and receives fiat through a provider.

### Step 4.1 Create Quote

Use this endpoint to calculate a sell quote before creating the order.

### Step 4.1 Create quote

Use this endpoint to create a sell quote and lock rate/amounts for sell flow. Use the response to show sell terms and pass quote id to order creation.
**POST** `/api/v3/exchange/merchant/quote`

### Headers
- `x-api-key: {{x-api-key}}`

**Request**

```json
{
    "clientId": "{{clientId}}",
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
}
```

**Response**

```json
{
    "id": "a95bf590-c029-47b2-bf95-adbcf50a11bb",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9765",
    "exchangeRateValue": "0.9765",
    "actualRateValue": "0.9208",
    "clientId": "{{clientId}}",
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
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For sell source, use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Crypto asset being sold. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For sell payout, use `FIAT_PROVIDER`. |
| `output.asset` | `string` | Yes | Fiat asset to receive. |
| `output.provider` | `string` | Yes | Fiat provider. |
| `output.token` | `string` | Conditional | Payment token for receiving fiat. |

### Response

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

**Request**

```json
{
  "quoteId": "a95bf590-c029-47b2-bf95-adbcf50a11bb"
}
```

**Response**

```json
{
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
    "clientId": "{{clientId}}",
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

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full quote/order calculation details. |
| `conditions.fromAsset` | `string` | No | Source asset code from the quote used to create order. |
| `conditions.toAsset` | `string` | No | Destination asset code from the quote used to create order. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount after source-side fee application. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount included in final settlement. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount after destination-side fee application. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount included in final settlement. |
| `conditions.rate` | `string` | No | Rate pair used for conversion calculation. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate applied for this quote/order. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Current order lifecycle state. Typical values: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `input` | `object` | Yes | Source crypto/internal balance operation. |
| `output` | `object` | Yes | Destination fiat provider operation. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Typical values: `NEW`, `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type (for example `P2P`, `SBP`, card payout type). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank used by fiat-provider route. |
| `input.clientBank` / `output.clientBank` | `string/null` | No | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string/null` | No | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string/null` | No | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |
| `input.post` / `output.post` | `string/null` | No | Additional provider payload or form-POST metadata when present. |
| `input.paymentSystem` / `output.paymentSystem` | `string/null` | No | Payment system metadata returned by provider integration. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for sell order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Client internal balance is not enough for requested sell operation. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this order. |


## 5) Operation History / Details

History endpoint is used to fetch created orders and their operation details for support, reconciliation, and client-facing history.

### Step 5.1 Get Order History / Details

Use this endpoint to search merchant orders by client, operation type, status, asset, or date range.

### Step 5.1 Get order history/details

Use this endpoint to fetch paged order history with optional filters and detailed operation data. Use the response to power history UI, reporting, and support investigations.
**POST** `/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`

### Headers
- `x-api-key: {{x-api-key}}`

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
      "clientId": "{{clientId}}",
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
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full order calculation details. |
| `conditions.fromAsset` | `string` | No | Source asset code from quote/order conditions. |
| `conditions.toAsset` | `string` | No | Destination asset code from quote/order conditions. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount after source-side fees. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount for the order. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount after destination-side fees. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount for the order. |
| `conditions.rate` | `string` | No | Asset pair used in order calculation. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate applied for order conditions. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate for historical order item. |
| `clientId` | `string` | Yes | Client id. |
| `status` | `string` | Yes | Current order lifecycle state. Typical values: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure for historical orders; useful for support and merchant-side audit. |
| `input` | `object` | Yes | Source operation details. |
| `output` | `object` | Yes | Destination operation details. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Typical values: `NEW`, `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for specific operation leg in history. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata. |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for fiat-provider leg. |
| `input.link` / `output.link` | `string/null` | No | Provider redirect/payment URL if present in operation leg. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |
| `totalElements` | `number` | Yes | Total number of matching orders. |
| `totalPages` | `number` | Yes | Total number of pages. |
| `number` | `number` | Yes | Current page number. |
| `size` | `number` | Yes | Current page size. |
| `first` | `boolean` | No | `true` when current page is first page. |
| `last` | `boolean` | No | `true` when current page is last page. |
| `numberOfElements` | `number` | No | Number of items on current page. |
| `empty` | `boolean` | No | `true` when `content` array is empty. |

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

**Request**

```json
{
    "clientId": "{{clientId}}",
    "fromAsset": "USDT_TRC",
    "fromPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    },
    "toAsset": "TRX",
    "toPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    }
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

**Request**

```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"INTERNAL_BALANCE",
        "asset":"USDT_TRC",
        "amount":5
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
}
```

**Response**

```json
{
    "id": "601b24b6-c7c3-4205-8396-79903f76f25e",
    "rate": "TRX/USDT_TRC",
    "systemRateValue": "0.3255",
    "exchangeRateValue": "0.3255",
    "actualRateValue": "0.3305",
    "clientId": "{{clientId}}",
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
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For conversion source, use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Source asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one side amount is required. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. For conversion destination, use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Destination asset. |
| `output.amount` | `number` | Conditional | Destination amount. At least one side amount is required. |

### Response

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

**Request**

```json
{
  "quoteId": "601b24b6-c7c3-4205-8396-79903f76f25e"
}
```

**Response**

```json
{
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
    "clientId": "{{clientId}}",
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

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Conversion order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full conversion calculation details. |
| `conditions.fromAsset` | `string` | No | Source asset code from the quote used to create conversion. |
| `conditions.toAsset` | `string` | No | Destination asset code from the quote used to create conversion. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount after source-side fee application. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount included in settlement. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount after destination-side fee application. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount included in settlement. |
| `conditions.rate` | `string` | No | Rate pair used for conversion calculation. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate applied for this conversion. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Yes | Client id. |
| `status` | `string` | Yes | Conversion order lifecycle state. Typical values: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`; most internal swaps end as `COMPLETED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when conversion cannot be completed; use for support/debugging. |
| `input` | `object` | Yes | Source internal balance operation. |
| `output` | `object` | Yes | Destination internal balance operation. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | Yes | Leg status. Typical values: `NEW`, `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code when operation leg uses fiat-provider route. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata for operation leg. |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for operation leg. |
| `input.link` / `output.link` | `string/null` | No | Provider redirect/payment URL if present in operation leg. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |

### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | Business error | No | Quote exists but cannot be used for conversion order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Source internal balance is not enough to execute conversion. |
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP error | No | Merchant has no access to quote/client used by this conversion. |


