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
| `destination` | `string` | No | Flow destination filter. Use `SDK_ACCOUNTING` for custodial wallet operations. |

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

| Name | Code | Description |
|---|---|---|
| `400 INVALID_DESTINATION` | BUSINESS | `destination` value cannot be mapped to supported enum for merchant assets. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant is authenticated but does not have `ASSET_API` permission. |
| `429 Too Many Requests` | HTTP | Rate limit is exceeded for this endpoint. |


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
            "number": 4039,
            "accountType": "WALLET",
            "operationType": "DEPOSIT",
            "amount": "100",
            "transactionId": "d5525adb-9bf7-401f-ad23-92ca2ef43dcf",
            "asset": "BYN",
            "status": "PENDING",
            "fiatProvider": "ASSIST",
            "orderIdentity": "33c890ab73d54e37bcae1cb8f3c0daef",
            "createdAt": "2026-05-06T07:22:20+0000"
        }
    ],
    "cryptoOperations": [
        {
            "number": 4038,
            "accountType": "WALLET",
            "operationType": "DEPOSIT",
            "submitTimeout": "ADDITIONAL",
            "transactionId": "92662469-7c10-4ae4-a776-0e16a4481aca",
            "status": "NEW",
            "depositCryptoAddress": "TQMCCuCs3C2tQr894CntWobDQymep5K2Xk",
            "amount": "50",
            "asset": "TRX",
            "network": null,
            "txHash": null,
            "createdAt": "2026-05-06T07:21:47+0000"
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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatOperations` | `array of objects` | Yes | Current fiat wallet operations. |
| `cryptoOperations` | `array of objects` | Yes | Current crypto wallet operations. |
| `fiatOperations[].number` | `number` | Yes | Human-readable fiat operation number. |
| `fiatOperations[].accountType` | `string` | Yes | Fiat operation account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. |
| `fiatOperations[].operationType` | `string` | Yes | Fiat operation type, for example `DEPOSIT` or `WITHDRAWAL`. |
| `fiatOperations[].amount` | `number` | Yes | Fiat operation amount in fiat asset currency. |
| `fiatOperations[].transactionId` | `string` | Yes | Internal fiat transaction identifier. |
| `fiatOperations[].asset` | `string` | Yes | Fiat asset code of the operation. |
| `fiatOperations[].status` | `string` | Yes | Fiat transaction status. Allowed values: `NEW`, `PENDING_REVIEW`, `REJECTED`, `TIMEOUT`, `DECLINED`, `INVALID_AMOUNT`, `ERROR`, `AML_BLOCKED`, `PENDING`, `PROCESSING`, `APPROVED`. |
| `fiatOperations[].fiatProvider` | `string` | No | Fiat provider used by the fiat operation. |
| `fiatOperations[].orderIdentity` | `string` | No | Provider-side order reference used for support/reconciliation. |
| `fiatOperations[].createdAt` | `string` | Yes | Fiat operation creation date/time. |
| `cryptoOperations[].number` | `number` | Yes | Human-readable crypto operation number. |
| `cryptoOperations[].accountType` | `string` | Yes | Crypto operation account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. |
| `cryptoOperations[].operationType` | `string` | Yes | Crypto operation type, for example `DEPOSIT` or `WITHDRAWAL`. |
| `cryptoOperations[].amount` | `number` | Yes | Crypto operation amount in crypto asset units. |
| `cryptoOperations[].transactionId` | `string` | Yes | Internal crypto transaction identifier. |
| `cryptoOperations[].asset` | `string` | Yes | Crypto asset code of the operation. |
| `cryptoOperations[].status` | `string` | Yes | Crypto transaction status. Allowed values: `NEW`, `PENDING_REVIEW`, `NOT_FOUND`, `REJECTED`, `TIMEOUT`, `INVALID_AMOUNT`, `ERROR`, `AML_ERROR`, `AML_BLOCKED`, `ARREST`, `SUBMITTING`, `SUBMITTED`, `PENDING`, `SELECTED`, `CONFIRMED`, `PENDING_RESOLVE`. |
| `cryptoOperations[].submitTimeout` | `string` | No | Crypto deposit timeout policy/mode. |
| `cryptoOperations[].depositCryptoAddress` | `string` | No | Blockchain address where client sends funds for crypto deposit. |
| `cryptoOperations[].network` | `string` | No | Blockchain network of the crypto operation. |
| `cryptoOperations[].txHash` | `string/null` | No | Blockchain transaction hash after transfer is detected. |
| `cryptoOperations[].createdAt` | `string` | Yes | Crypto operation creation date/time. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to the merchant in access validation. |
| `400 Bad Request` | HTTP | Request parameters are invalid or cannot be parsed. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for this client or endpoint. |

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
    "accountType":"WALLET",
    "asset":{
        "code":"TRX",
        "network":"Tron",
        "amount":50
    }
} 
```

**Response**

```json
{
    "transactionId": "99922b6c-72e8-48a3-9886-a5191e88e96f",
    "depositCryptoAddress": "TQMCCuCs3C2tQr894CntWobDQymep5K2Xk"
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | Yes | Account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. Use `WALLET` for custodial wallet flow. |
| `asset.code` | `string` | Yes | Asset code used for the operation. |
| `asset.network` | `string` | Yes | Blockchain network of the selected crypto asset. |
| `asset.amount` | `number` | Yes | Operation amount in the selected asset. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Transaction identifier for tracking operation status, support cases, and reconciliation. |
| `depositCryptoAddress` | `string` | Yes | Blockchain address that must be shown to the client as the destination for crypto deposit. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 ACTIVE_DEPOSIT_REQUEST_FOUND` | BUSINESS | An uncompleted deposit already exists for this client/asset. |
| `400 INVALID_AMOUNT` | BUSINESS | Provided amount is invalid for deposit constraints. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to the merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for this operation. |


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
        "id": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "number": "**** **** **** 1111",
        "brand": "VISA",
        "providerId": "ASSIST",
        "providerType": "ASSIST",
        "status": "ENABLED",
        "isRestricted": false,
        "isCrypto": false,
        "country": "Russia"
    },
]
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `fiatAsset` | `string` | No | Fiat currency filter, for example `BYN`. |
| `orderType` | `string` | No | Operation type filter, for example `BUY` for fiat input. |
| `destination` | `string` | No | Flow destination filter. Use `SDK_ACCOUNTING` for custodial wallet operations. |
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
| `status` | `string` | Yes | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `isRestricted` | `boolean` | Yes | Shows whether this payment method is restricted. |
| `isCrypto` | `boolean` | Yes | Shows whether method is crypto-related. |
| `country` | `string` | No | Payment method country. |
| `currency` | `string` | No | Primary fiat currency. |
| `supportedCurrencies` | `array of strings` | No | Fiat currencies supported by this payment method. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to the merchant. |
| `400 INVALID_ORDER_TYPE` | BUSINESS | `orderType` value is unsupported for payment method resolution. |
| `400 INVALID_FIAT_ASSET` | BUSINESS | `fiatAsset` value is unsupported for the selected flow. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no `PAYMENT_API` permission for this client. |
| `429 Too Many Requests` | HTTP | Rate limit is exceeded for payment methods endpoint. |


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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | Yes | Account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. Use `WALLET` for custodial wallet flow. |
| `fiatProviderType` | `string` | Yes | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | Conditional | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | Conditional | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | Yes | Asset code used for the operation. |
| `asset.amount` | `number` | Yes | Operation amount in the selected asset. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `fiatPaymentLink` | `string` | No | Payment URL that client should open to complete fiat deposit. |
| `creationDate` | `string` | Yes | Creation timestamp in server date-time format. |
| `expirationMinutes` | `number` | No | Payment link lifetime in minutes. |
| `paymentDetails` | `object` | No | Provider-specific payment data. |
| `paymentDetails.paymentLink` | `string` | No | Provider payment URL. |
| `paymentDetails.notificationPhoneNumber` | `string/null` | No | Phone number returned by provider when the payment scenario requires notification or additional confirmation. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | BUSINESS | Fiat provider operation cannot be started or processed. |
| `400 INVALID_PAYMENT_TOKEN` | BUSINESS | `paymentToken`/`internalToken` is invalid, restricted, or missing. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for this operation. |


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
        "amount":100,
        "code":"TRX",
        "network":"Tron"
    },
    "toAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
}
```

**Response**

```json
{
    "id": "32b57e46-ee40-4e53-9144-4947648aedd6",
    "withdrawalAmount": "100",
    "commissionAmount": "0.263",
    "receivedAmount": "99.737",
    "expirationDate": "2026-05-06T07:44:41+0000"
}
```

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `asset.amount` | `number` | Yes | Operation amount in the selected asset. |
| `asset.code` | `string` | Yes | Asset code used for the operation. |
| `asset.network` | `string` | Yes | Blockchain network of the selected crypto asset. |
| `toAddress` | `string` | Yes | Destination crypto address. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Calculation id used to create withdrawal. |
| `withdrawalAmount` | `string` | Yes | Original withdrawal amount. |
| `commissionAmount` | `string` | Yes | Commission amount for the operation. |
| `receivedAmount` | `string` | Yes | Net amount expected after fees/commissions. |
| `expirationDate` | `string` | No | Expiration timestamp in server date-time format, if returned. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 INVALID_ADDRESS` | BUSINESS | Destination address is invalid for selected network or blocked as internal address. |
| `400 INVALID_AMOUNT` | BUSINESS | Amount is invalid (including fee greater than withdrawal amount). |
| `400 ACTIVE_WITHDRAWAL_REQUEST_FOUND` | BUSINESS | An uncompleted withdrawal already exists for this client/asset. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for this operation. |


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

### Headers

| Name | Type | Required | Description |
|---|---|---:|---|
| `x-api-key` | `string` | Yes | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
|---|---|---:|---|
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | Yes | Account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. Use `WALLET` for custodial wallet flow. |
| `calculationId` | `string` | Yes | Calculation id returned by withdrawal calculation endpoint. |
| `comment` | `string` | No | Optional memo/comment/tag for networks that require additional destination data. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | No | Created crypto withdrawal transaction identifier used for tracking status and support. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 INVALID_CALCULATION` | BUSINESS | `calculationId` is not found or expired. |
| `400 INVALID_STATUS` | BUSINESS | Withdrawal operation status does not allow execution. |
| `400 ACTIVE_WITHDRAWAL_REQUEST_FOUND` | BUSINESS | Another uncompleted withdrawal blocks this operation. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Operation is forbidden for this merchant (`AccessDeniedException`). |


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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `fiatProviderType` | `string` | Yes | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | Conditional | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | Conditional | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | Yes | Asset code used for the operation. |
| `asset.amount` | `number` | Yes | Operation amount in the selected asset. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string/null` | No | Calculation identifier when provider creates a reusable calculation. Can be `null` when the fiat calculation is direct and no follow-up calculation id is required. |
| `withdrawalAmount` | `string` | Yes | Amount requested for withdrawal. |
| `commissionAmount` | `string` | Yes | Commission amount for the operation. |
| `receivedAmount` | `string` | Yes | Net amount expected after fees/commissions. |
| `expirationDate` | `string/null` | No | Expiration timestamp in server date-time format, if returned. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | BUSINESS | Fiat withdrawal calculation cannot be produced by provider/flow. |
| `400 INVALID_PAYMENT_TOKEN` | BUSINESS | Payment token is invalid, unavailable, or unsupported. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for this operation. |


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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | Yes | Account scope. Allowed values: `WALLET`, `TRADING`, `BROKER`. Use `WALLET` for custodial wallet flow. |
| `fiatProviderType` | `string` | Yes | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | Conditional | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | Conditional | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | Yes | Asset code used for the operation. |
| `asset.amount` | `number` | Yes | Operation amount in the selected asset. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `transactionId` | `string` | Yes | Created fiat withdrawal transaction identifier used for tracking payout status and reconciliation. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 BALANCE_OPERATION_PROCESSING_ERROR` | BUSINESS | Fiat withdrawal cannot be started or provider rejected operation. |
| `400 INVALID_PAYMENT_TOKEN` | BUSINESS | Payment token/internal token is invalid or restricted. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Operation is forbidden for this merchant (`AccessDeniedException`). |


## 3) Buy (`buy`)

Buy flow is used when the client pays fiat through a provider and receives crypto to internal wallet balance. The flow is quote first, then order creation.

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
    "id": "bfb363eb-673d-4233-aad0-29bd186454f0",
    "rate": "TRX/BYN",
    "systemRateValue": "1.0263",
    "exchangeRateValue": "1.0263",
    "actualRateValue": "1.1",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "creationDate": "2026-05-06T08:17:13+0000",
    "expirationDate": "2026-05-06T08:17:43+0000",
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
        "amount": "45.454545",
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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Yes | Source asset code. |
| `input.amount` | `number` | Conditional | Source amount for quote/order calculation. |
| `input.provider` | `string` | Yes | Fiat provider used for payment. |
| `input.token` | `string` | Conditional | Payment token used for provider payment. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Yes | Destination asset code. |
| `output.amount` | `number` | Conditional | Destination amount for quote/order calculation. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used to create order. |
| `rate` | `string` | Yes | Rate pair for the operation. |
| `systemRateValue` | `string` | Yes | Base system rate at calculation time. |
| `exchangeRateValue` | `string` | Yes | Exchange rate applied to this quote/order. |
| `actualRateValue` | `string` | Yes | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `creationDate` | `string` | Yes | Creation timestamp in server date-time format. |
| `expirationDate` | `string` | Yes | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `input.feeAmount` / `output.feeAmount` | `string` | Yes | Fee amount on each operation leg. |
| `input.paymentType` | `string` | No | Fiat payment type selected by provider configuration. |
| `input.processingBank` | `string` | No | Processing bank selected for fiat provider route. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 INVALID_QUOTE` | BUSINESS | Quote input is inconsistent or cannot be calculated for provided payment details. |
| `400 CURRENCY_NOT_FOUND` | BUSINESS | One of input/output assets is unknown. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Provided client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no `ORDER_V2_API` permission. |
| `429 Too Many Requests` | HTTP | Rate limit is exceeded for quote creation endpoint. |


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
    "id": "3afe0970-fd7a-44b8-8822-949658b4d9c0",
    "number": 771000004292,
    "conditions": {
        "fromAsset": "BYN",
        "toAsset": "TRX",
        "fromGrossAmount": "50",
        "fromNetAmount": "46.65",
        "fromFeeAmount": "3.35",
        "toGrossAmount": "45.229785",
        "toNetAmount": "45.229785",
        "toFeeAmount": "0",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "1.0314",
        "exchangeRateValue": "1.0314",
        "actualRateValue": "1.1055"
    },
    "recalculationReason": null,
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "PROCESSING",
    "failureMessage": null,
    "completionDate": null,
    "creationDate": "2026-05-05T17:03:38+0000",
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
        "toToken": "50d53e2e-f086-472a-9ce2-cdcee43279cb",
        "link": "https://payments.t.paysecure.ru/pay/p2p/...",
        "processorTransactionId": "7a672d4f09794d3ea754f4cd6d01c796",
        "post": null,
        "paymentSystem": null,
        "processorTransactionNumber": null
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "45.229785",
        "transactionAmount": "45.229785",
        "feeAmount": "0",
        "status": "NEW",
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
| `quoteId` | `string` | Yes | Quote identifier returned by quote creation; required to create an order before quote expiration. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | No | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | No | Destination asset code in calculation conditions. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount in calculation conditions. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount in calculation conditions. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount in calculation conditions. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount in calculation conditions. |
| `conditions.rate` | `string` | No | Rate pair in calculation conditions. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate value in calculation conditions. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate value in calculation conditions. |
| `recalculationReason` | `string/null` | No | Recalculation reason when quote/order amounts were adjusted by system logic; `null` when no recalculation happened. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Yes | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `completionDate` | `string/null` | No | Order completion timestamp when order is finished; `null` while order is still active. |
| `creationDate` | `string` | Yes | Creation timestamp in server date-time format. |
| `sessionId` | `string/null` | No | Optional client session identifier bound to this order. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider operation leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata (for example `P2P`, `SBP`). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for fiat-provider operation leg. |
| `input.clientBank` / `output.clientBank` | `string/null` | No | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string/null` | No | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string/null` | No | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |
| `input.processorTransactionNumber` / `output.processorTransactionNumber` | `string/null` | No | External provider transaction number/reference shown by provider systems for support and reconciliation. |
| `input.post` / `output.post` | `string/null` | No | Additional provider payload or form-POST metadata when present. |
| `input.paymentSystem` / `output.paymentSystem` | `string/null` | No | Payment system metadata returned by provider integration. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 QUOTE_NOT_FOUND` | BUSINESS | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | BUSINESS | Quote exists but cannot be used for order creation. |
| `400 INVALID_CLIENT_STATUS` | BUSINESS | Client status/checks do not allow order creation (for example testing not completed). |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no access to quote/client used by this order. |


## 4) Sell (`sell`) - Merchant V3 Flow

Sell flow is used when the client sells crypto from internal wallet balance and receives fiat through a provider.

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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Yes | Source asset code. |
| `input.amount` | `number` | Conditional | Source amount for quote/order calculation. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Yes | Destination asset code. |
| `output.provider` | `string` | Yes | Fiat provider. |
| `output.token` | `string` | Conditional | Payment token for receiving fiat. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used for order creation. |
| `rate` | `string` | Yes | Rate pair for the operation. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `expirationDate` | `string` | Yes | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `output.amount` | `string` | Yes | Destination amount for quote/order calculation. |
| `output.feeAmount` | `string` | Yes | Fiat provider/exchange fee amount. |
| `output.paymentType` | `string` | No | Fiat payment type. |
| `output.processingBank` | `string` | No | Processing bank selected by provider route. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 INVALID_QUOTE` | BUSINESS | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | BUSINESS | Input/output asset is unknown. |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no `ORDER_V2_API` permission. |


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
| `quoteId` | `string` | Yes | Quote identifier returned by quote creation; required to create an order before quote expiration. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | No | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | No | Destination asset code in calculation conditions. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount in calculation conditions. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount in calculation conditions. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount in calculation conditions. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount in calculation conditions. |
| `conditions.rate` | `string` | No | Rate pair in calculation conditions. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate value in calculation conditions. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Yes | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider operation leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata (for example `P2P`, `SBP`). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for fiat-provider operation leg. |
| `input.clientBank` / `output.clientBank` | `string/null` | No | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string/null` | No | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string/null` | No | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |
| `input.post` / `output.post` | `string/null` | No | Additional provider payload or form-POST metadata when present. |
| `input.paymentSystem` / `output.paymentSystem` | `string/null` | No | Payment system metadata returned by provider integration. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 QUOTE_NOT_FOUND` | BUSINESS | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | BUSINESS | Quote exists but cannot be used for sell order creation. |
| `400 INSUFFICIENT_BALANCE` | BUSINESS | Client internal balance is not enough for requested sell operation. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no access to quote/client used by this order. |


## 5) Operation History / Details

History endpoint is used to fetch created orders and their operation details for support, reconciliation, and client-facing history.

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
| `externalClientId` | `string` | Conditional | External client identifier. Required when `clientIds` is not provided. At least one client selector must be present. |
| `clientIds` | `array of strings` | Conditional | Required when `externalClientId` is not provided. At least one client selector must be present, otherwise request fails with `Client id is required`. |
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
| `conditions` | `object` | No | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | No | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | No | Destination asset code in calculation conditions. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount in calculation conditions. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount in calculation conditions. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount in calculation conditions. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount in calculation conditions. |
| `conditions.rate` | `string` | No | Rate pair in calculation conditions. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate value in calculation conditions. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Yes | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure for historical orders; useful for support and merchant-side audit. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | No | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider operation leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata (for example `P2P`, `SBP`). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for fiat-provider operation leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
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

| Name | Code | Description |
|---|---|---|
| `400 Bad Request` | HTTP | Filter body is invalid or missing required client filter (`Client id is required`). |
| `400 CLIENT_NOT_FOUND` | BUSINESS | Provided client id/external client id is invalid for this merchant. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no permission for order history scope. |
| `429 Too Many Requests` | HTTP | Rate limit is exceeded for history endpoint. |


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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
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

| Name | Code | Description |
|---|---|---|
| `400 INVALID_MERCHANT_ID` | BUSINESS | No merchant identifier could be resolved for limit calculation context. |
| `400 INVALID_QUOTE` | BUSINESS | Limit request contains invalid pair/payment details for calculation context. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no `LIMIT_V2_API` permission. |
| `429 Too Many Requests` | HTTP | Rate limit is exceeded for limit endpoint. |


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
    "id": "95294f34-c8b6-4811-ae06-1dc76dd2c7ad",
    "rate": "TRX/USDT_TRC",
    "systemRateValue": "0.3422",
    "exchangeRateValue": "0.3422",
    "actualRateValue": "0.3474",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "creationDate": "2026-05-06T08:22:27+0000",
    "expirationDate": "2026-05-06T08:22:57+0000",
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "USDT_TRC",
        "amount": "5",
        "feeAmount": "0"
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "14.392168",
        "feeAmount": "0.21917"
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
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | Yes | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Yes | Source asset code. |
| `input.amount` | `number` | Conditional | Source amount for quote/order calculation. |
| `output.type` | `string` | Yes | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Yes | Destination asset code. |
| `output.amount` | `number` | Conditional | Destination amount for quote/order calculation. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Quote id used for conversion order. |
| `rate` | `string` | Yes | Rate pair for the operation. |
| `systemRateValue` | `string` | Yes | Base system rate at calculation time. |
| `exchangeRateValue` | `string` | Yes | Exchange rate applied to this quote/order. |
| `actualRateValue` | `string` | Yes | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `expirationDate` | `string` | Yes | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `feeAmount` | `string` | Yes | Fee amount on source/destination side. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 INVALID_QUOTE` | BUSINESS | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | BUSINESS | Asset id is unknown for conversion pair. |
| `400 INVALID_CLIENT_STATUS` | BUSINESS | Client status/checks do not allow conversion quote creation. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no `ORDER_V2_API` permission. |


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
    "id": "85179db7-02fc-48cc-8694-af655d67755d",
    "number": 161000004313,
    "conditions": {
        "fromAsset": "USDT_TRC",
        "toAsset": "TRX",
        "fromGrossAmount": "5",
        "fromNetAmount": "5",
        "fromFeeAmount": "0",
        "toGrossAmount": "14.611338",
        "toNetAmount": "14.392168",
        "toFeeAmount": "0.21917",
        "promoCode": null,
        "rate": "TRX/USDT_TRC",
        "systemRateValue": "0.3422",
        "exchangeRateValue": "0.3422",
        "actualRateValue": "0.3474"
    },
    "recalculationReason": null,
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "COMPLETED",
    "failureMessage": null,
    "completionDate": "2026-05-06T08:23:49+0000",
    "creationDate": "2026-05-06T08:23:40+0000",
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
        "amount": "14.392168",
        "transactionAmount": "14.392168",
        "feeAmount": "0.21917",
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
| `quoteId` | `string` | Yes | Quote identifier returned by quote creation; required to create an order before quote expiration. |

### Response

| Name | Type | Required | Description |
|---|---|---:|---|
| `id` | `string` | Yes | Conversion order id. |
| `number` | `number` | Yes | Human-readable order number. |
| `conditions` | `object` | No | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | No | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | No | Destination asset code in calculation conditions. |
| `conditions.fromGrossAmount` | `string` | No | Source gross amount before source-side fees. |
| `conditions.fromNetAmount` | `string` | No | Source net amount in calculation conditions. |
| `conditions.fromFeeAmount` | `string` | No | Source-side fee amount in calculation conditions. |
| `conditions.toGrossAmount` | `string` | No | Destination gross amount before destination-side fees. |
| `conditions.toNetAmount` | `string` | No | Destination net amount in calculation conditions. |
| `conditions.toFeeAmount` | `string` | No | Destination-side fee amount in calculation conditions. |
| `conditions.rate` | `string` | No | Rate pair in calculation conditions. |
| `conditions.systemRateValue` | `string` | No | Base system rate at calculation time. |
| `conditions.exchangeRateValue` | `string` | No | Exchange rate value in calculation conditions. |
| `conditions.actualRateValue` | `string` | No | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Yes | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Yes | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string/null` | No | Human-readable reason of failure when conversion cannot be completed; use for support/debugging. |
| `input` | `object` | Yes | Source operation details object. |
| `output` | `object` | Yes | Destination operation details object. |
| `input.type` / `output.type` | `string` | No | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | No | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | No | Operation amount for each leg. |
| `input.transactionAmount` / `output.transactionAmount` | `string` | No | Provider/settlement amount for operation leg. |
| `input.feeAmount` / `output.feeAmount` | `string` | No | Fee amount on each operation leg. |
| `input.status` / `output.status` | `string` | Yes | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `input.failureMessage` / `output.failureMessage` | `string/null` | No | Failure reason for a specific operation leg. |
| `input.expirationDate` / `output.expirationDate` | `string/null` | No | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string/null` | No | Provider code for fiat-provider operation leg. |
| `input.paymentType` / `output.paymentType` | `string/null` | No | Provider payment type metadata (for example `P2P`, `SBP`). |
| `input.processingBank` / `output.processingBank` | `string/null` | No | Processing bank metadata for fiat-provider operation leg. |
| `input.link` / `output.link` | `string/null` | No | Provider payment URL for redirect/confirmation flows. |
| `input.processorTransactionId` / `output.processorTransactionId` | `string/null` | No | External provider transaction id for reconciliation. |

### Errors

| Name | Code | Description |
|---|---|---|
| `400 QUOTE_NOT_FOUND` | BUSINESS | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | BUSINESS | Quote exists but cannot be used for conversion order creation. |
| `400 INSUFFICIENT_BALANCE` | BUSINESS | Source internal balance is not enough to execute conversion. |
| `401 Unauthorized` | HTTP | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | HTTP | Merchant has no access to quote/client used by this conversion. |


