# CUSTODIAL_WALLET

The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.  
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.  

> BASE_URL https://api.dev.wbdevel.net

## 0) Wallet base data

### Step 0.1 Get available assets
**POST** `/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`

**Headers** `x-api-key`

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

### Step 0.2 Get current balance operations
**GET** `/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

Required request params:
- `clientId`

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
| `txHash` | `string \| null` | No | Blockchain transaction hash when known. |
| `createdAt` | `string` | Yes | Operation creation date/time. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Merchant does not have access to the client or endpoint. |
| `400 CLIENT_NOT_FOUND` | Business error | No | Client id is invalid or client is not linked to the merchant. |

### Step 0.3 Get enhanced merchant account balances
**GET** `/api/v2/accounting/merchant/account/enhanced`

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
    },
    {
      "currency": "USDT",
      "type": "USER_BALANCE",
      "amount": 40.24507500,
      "usdRate": 1,
      "usdAmount": 40.25,
      "creationDate": 1732799992688,
      "modificationDate": 1777534606246,
      "fiat": false
    },
    {
      "currency": "USD",
      "type": "USER_BALANCE",
      "amount": 85.00,
      "usdRate": 1,
      "usdAmount": 85.00,
      "creationDate": 1732799992688,
      "modificationDate": 1732800042763,
      "fiat": true
    },
    {
      "currency": "RUB",
      "type": "USER_BALANCE",
      "amount": 4109.49,
      "usdRate": 0.012,
      "usdAmount": 49.31,
      "creationDate": 1732799992688,
      "modificationDate": 1776797970394,
      "fiat": true
    }
  ],
  "totalFiatUsdAmount": 134.31,
  "totalCryptoUsdAmount": 93.48
}
```

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

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `401 Unauthorized` | HTTP error | Yes | `x-api-key` is missing or invalid. |
| `403 Forbidden` | HTTP error | No | Merchant is not allowed to access account balances. |
| `500 Internal Server Error` | HTTP error | No | Accounting service or balance provider error. |

## 1) Deposit (`deposit`)

### Step 1.1 Create crypto deposit
**POST** `/api/v2/exchange/merchant/balance/crypto/deposit`

Required body fields:
- `clientId`
- `accountType`
- `asset.code`
- `asset.network`
- `asset.amount`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
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

### Step 1.2 Get fiat payment methods
**POST** `/api/v2/exchange/merchant/payment/method`

Required body fields:
- `clientId`

Optional filters:
- `fiatAsset`
- `orderType`
- `destination`
- `providers`
- `isCrypto`
- `countryGroup`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
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

### Step 1.3 Create fiat deposit
**POST** `/api/v2/exchange/merchant/balance/fiat/deposit`

Required body fields:
- `clientId`
- `accountType`
- `fiatProviderType`
- `asset.code`
- `asset.amount`
- one of `paymentToken` or `internalToken`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
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

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatPaymentLink` | `string` | No | Payment URL that client should open to complete fiat deposit. |
| `creationDate` | `string` | Yes | Deposit creation date/time. |
| `expirationMinutes` | `number` | No | Payment link lifetime in minutes. |
| `paymentDetails` | `object` | No | Provider-specific payment data. |
| `paymentDetails.paymentLink` | `string` | No | Provider payment URL. |
| `paymentDetails.notificationPhoneNumber` | `string \| null` | No | Phone number used for provider notifications when available. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | `paymentToken`/`internalToken` is missing, invalid, or unavailable. |
| `400 INVALID_FIAT_PROVIDER` | Business error | No | Provider is unsupported for this currency or flow. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot perform fiat deposit. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

## 2) Send (`withdrawal`)

### Step 2.1 Calculate crypto withdrawal
**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

Required body fields:
- `clientId`
- `asset.code`
- `asset.network`
- `asset.amount`
- `toAddress`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "asset":{
        "amount":10,
        "code":"TRX",
        "network":"Tron"
    },
    "toAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"  // destination crypto withdrawal address
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

### Step 2.2 Create crypto withdrawal
**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal`

Required body fields:
- `clientId`
- `calculationId`
- `accountType`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "accountType":"WALLET",
    "calculationId":"ea40bbbf-16a2-4fa2-aada-f55121c45eac",
    "comment":""  // MEMO/comment/TAG field, used as destination memo for networks like TON
}
```

**Response**
```json
{
  "transactionId": "crypto-withdrawal-transaction-id"
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

### Step 2.3 Calculate fiat withdrawal
**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

Required body fields:
- `clientId`
- `fiatProviderType`
- `asset.code`
- `asset.amount`

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

### Step 2.4 Create fiat withdrawal
**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal`

Required body fields:
- `clientId`
- `accountType`
- `fiatProviderType`
- `asset.code`
- `asset.amount`
- one of `paymentToken` or `internalToken`

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

## 3) Buy (`buy`) — merchant V3 flow

### Step 3.1 Create quote
**POST** `/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
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
| `input.type` | `string` | Yes | Source operation type. For buy use `FIAT_PROVIDER`. |
| `input.asset` | `string` | Yes | Source fiat asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `input.provider` | `string` | Yes | Fiat provider used for payment. |
| `input.token` | `string` | Conditional | Payment token used for provider payment. |
| `output.type` | `string` | Yes | Destination operation type. For custodial wallet use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Crypto asset that will be credited to internal balance. |
| `output.amount` | `number` | Conditional | Target amount. At least one of `input.amount` or `output.amount` is required. |

#### Response

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

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote request cannot be calculated with provided assets, amount, or payment details. |
| `400 INVALID_CLIENT_STATUS` | Business error | No | Client cannot create quote for this operation. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Payment token is invalid or restricted. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 3.2 Create buy order
**POST** `/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":"47b2985a-2fe3-427c-9a18-6b16736c460e"
}

// exchange operation is processed immediately
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
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "PROCESSING",
    "failureMessage": null,
    "completionDate": null,
    "creationDate": "2026-04-30T11:29:29+0000",
    "sessionId": null,
    "input": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "50",
        "transactionAmount": "50",
        "feeAmount": "3.35",
        "status": "PROCESSING",
        "failureMessage": null,
        "expirationDate": null,
        "provider": "ASSIST",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK",
        "clientBank": null,
        "fromToken": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "toToken": "97fe9aa7-7805-438f-8c5e-aea24b4f9dc4",
        "link": "https://payments.t.paysecure.ru/pay/p2p/cc2mc.cfm?merchant_id=...&orderNumber=...&customerNumber=...&orderCurrency=BYN&orderAmount=50.0&checkValue=...&signature=...&tokenFrom=...&tokenTo=...",
        "processorTransactionId": "c4e50d3e83bd4fccb8bf8b742470475f",
        "post": null,
        "paymentSystem": null
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "47.757985",
        "transactionAmount": "47.757985",
        "feeAmount": "0",
        "status": "NEW",
        "failureMessage": null,
        "expirationDate": null
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
| `quoteId` | `string` | Yes | Quote id returned by quote creation endpoint. |

#### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full order calculation details. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Order status, for example `PROCESSING`, `COMPLETED`, `FAILED`. |
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

## 4) Sell (`sell`) — merchant V3 flow

### Step 4.1 Create quote
**POST** `/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"INTERNAL_BALANCE",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"TRX",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":100                  // amount
    },
    "output":{
        "type":"FIAT_PROVIDER",
        "asset":"BYN",
        "provider": "ASSIST",                                // provider
        "token": "{{payment_token}}"      // payment token id
    }
}

// amount can be provided in input or output
// systemRateValue   - rate without fee
// exchangeRateValue - rate with fee
// actualRateValue   - currently not used in UI logic
// feeAmount         - fee amount
// expirationDate    - quote lifetime
```

**Response**
```json
{
    "id": "a95bf590-c029-47b2-bf95-adbcf50a11bb",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9765",
    "exchangeRateValue": "0.9765",
    "actualRateValue": "0.9208",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
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

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom sell quote is created. |
| `input.type` | `string` | Yes | Source operation type. For custodial wallet sell use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Crypto asset being sold. |
| `input.amount` | `number` | Conditional | Source amount. At least one of `input.amount` or `output.amount` is required. |
| `output.type` | `string` | Yes | Destination operation type. For fiat payout use `FIAT_PROVIDER`. |
| `output.asset` | `string` | Yes | Fiat asset to receive. |
| `output.provider` | `string` | Yes | Fiat provider. |
| `output.token` | `string` | Conditional | Payment token for receiving fiat. |

#### Response

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

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be calculated. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Client wallet balance is not enough. |
| `400 INVALID_PAYMENT_TOKEN` | Business error | No | Fiat payment token is invalid or restricted. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 4.2 Create sell order
**POST** `/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":"a95bf590-c029-47b2-bf95-adbcf50a11bb"
}

// exchange operation is processed immediately
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
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
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

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full quote/order calculation details. |
| `clientId` | `string` | Yes | Client id for this order. |
| `status` | `string` | Yes | Current order status. |
| `failureMessage` | `string \| null` | No | Failure reason when order fails. |
| `input` | `object` | Yes | Source crypto/internal balance operation. |
| `output` | `object` | Yes | Destination fiat provider operation. |
| `output.provider` | `string` | No | Fiat provider used for payout. |
| `output.processingBank` | `string` | No | Processing bank selected by fiat provider route. |
| `processorTransactionId` | `string \| null` | No | Provider transaction id when available. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or not found. |
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be used for order creation. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

## 5) Operation history/details

### Step 5.1 Get order history/details
**POST** `/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`

**Request**
```json
{
    "clientIds": [
        "{{clientId}}"
    ]
}
// "operationTypes": [], // FIAT_PROVIDER / CRYPTO_TRANSFER / INTERNAL_BALANCE
// "statuses": [],       // PROCESSING / EXPIRED / COMPLETED / FAILED
// "assets": [],         // assets: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
// "completionDateFrame":{    // completion date range
//     "start":"2024-08-25T00:00:00+0300",
//     "end":"2026-09-02T00:00:00+0300"
// },
// "creationDateFrame":{      // creation date range
//     "start":"2024-08-25T00:00:00+0300",
//     "end":"2026-09-02T00:00:00+0300"
// }
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
      "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
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
  ],
  "totalElements": 1,
  "totalPages": 1,
  "number": 0,
  "size": 20
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

## 6) Conversion (`conversion`)

For custodial wallet, conversion goes through internal balance (`USER_BALANCE` / `INTERNAL_BALANCE`).


### Step 6.1 Check limits
**POST** `/api/v3/exchange/merchant/limit`

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

### Step 6.2 Create quote
**POST** `/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"INTERNAL_BALANCE",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"USDT_TRC",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":5                  // amount
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
}

// amount can be provided in input or output
// systemRateValue   - rate without fee
// exchangeRateValue - rate with fee
// actualRateValue   - currently not used in UI logic
// feeAmount         - fee amount
// expirationDate    - quote lifetime
```

**Response**
```json
{
    "id": "601b24b6-c7c3-4205-8396-79903f76f25e",
    "rate": "TRX/USDT_TRC",
    "systemRateValue": "0.3255",
    "exchangeRateValue": "0.3255",
    "actualRateValue": "0.3305",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
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

#### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Merchant API key. |
| `Content-Type` | `string` | Yes | Must be `application/json`. |

#### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client for whom quote is created. |
| `input.type` | `string` | Yes | Source type. For conversion use `INTERNAL_BALANCE`. |
| `input.asset` | `string` | Yes | Source asset. |
| `input.amount` | `number` | Conditional | Source amount. At least one side amount is required. |
| `output.type` | `string` | Yes | Destination type. For conversion use `INTERNAL_BALANCE`. |
| `output.asset` | `string` | Yes | Destination asset. |
| `output.amount` | `number` | Conditional | Destination amount. At least one side amount is required. |

#### Response

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

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be calculated. |
| `400 AMOUNT_OUT_OF_LIMIT` | Business error | No | Amount is outside allowed min/max. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

### Step 6.3 Create swap operation
**POST** `/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":""
}
// exchange operation is processed immediately
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
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
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

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Conversion order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Full conversion calculation details. |
| `clientId` | `string` | Yes | Client id. |
| `status` | `string` | Yes | Conversion order status. Usually `COMPLETED` when internal swap succeeds. |
| `failureMessage` | `string \| null` | No | Failure reason when conversion fails. |
| `input` | `object` | Yes | Source internal balance operation. |
| `output` | `object` | Yes | Destination internal balance operation. |
| `input.status` / `output.status` | `string` | Yes | Status of each operation leg. |

#### Errors

| Name | Type | Required | Description |
|---|---|---:|---|
| `400 QUOTE_NOT_FOUND` | Business error | No | Quote id is missing, expired, or not found. |
| `400 INVALID_QUOTE` | Business error | No | Quote cannot be used for conversion. |
| `400 INSUFFICIENT_BALANCE` | Business error | No | Source internal balance is not enough. |
| `401 Unauthorized` | HTTP error | Yes | Invalid or missing `x-api-key`. |

