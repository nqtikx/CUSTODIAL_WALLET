# CUSTODIAL_WALLET

The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.  
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.  

> BASE_URL https://api.dev.wbdevel.net

## 1) Base Data

Base data endpoints are used before any wallet operation. They help the merchant show available assets, current operations, and account balances to the client.

### Step 1.1 Get available assets

Use this endpoint to retrieve all fiat and crypto assets available for custodial wallet operations. Use the response to build asset selectors and validate supported routes before any operation.

**POST** `/api/v2/exchange/merchant/assets?destination=EXCHANGE`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `fiatAssets` | `array of objects` | List of fiat assets that can be shown to the client as available wallet currencies for this merchant flow. |
| `fiatAssets[].id` | `string` | Internal asset identifier used in API requests and routing logic. |
| `fiatAssets[].code` | `string` | Currency code that can be displayed to the client in UI. |
| `cryptoAssets` | `array of objects` | List of crypto assets/networks that can be used in deposit, withdrawal, buy, sell, or conversion flows. |
| `cryptoAssets[].id` | `string` | Internal crypto asset identifier used in API requests; may include network-specific suffixes such as `USDT_TRC`. |
| `cryptoAssets[].code` | `string` | Asset ticker displayed to the client; can differ from `id` when asset is network-specific. |
| `cryptoAssets[].network` | `string` | Blockchain network that must be used for deposits/withdrawals of this asset. |
| `cryptoAssets[].protocol` | `string` | Token protocol shown to prevent sending funds through the wrong network. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_DESTINATION` | `BUSINESS` | `destination` value cannot be mapped to supported enum for merchant assets. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |
| `429 Too Many Requests` | `HTTP` | Rate limit is exceeded for this endpoint. |


### Step 1.2 Get current balance operations

Use this endpoint to fetch the client's current fiat and crypto wallet operations and statuses. Use the response to show live operation state in UI and support dashboards.

**GET** `/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `fiatOperations` | `array of objects` | Current fiat wallet operations. |
| `cryptoOperations` | `array of objects` | Current crypto wallet operations. |
| `fiatOperations[].number` | `number` | Human-readable fiat operation number. |
| <nobr>`fiatOperations[].accountType`</nobr> | `string` | Fiat operation account scope. Value: `WALLET`. |
| <nobr>`fiatOperations[].operationType`</nobr> | `string` | Fiat operation type, for example `DEPOSIT` or `WITHDRAWAL`. |
| `fiatOperations[].amount` | `number` | Fiat operation amount in fiat asset currency. |
| <nobr>`fiatOperations[].transactionId`</nobr> | `string` | Internal fiat transaction identifier. |
| `fiatOperations[].asset` | `string` | Fiat asset code of the operation. |
| `fiatOperations[].status` | `string` | Fiat transaction status. Allowed values: `NEW`, `PENDING_REVIEW`, `REJECTED`, `TIMEOUT`, `DECLINED`, `INVALID_AMOUNT`, `ERROR`, `AML_BLOCKED`, `PENDING`, `PROCESSING`, `APPROVED`. |
| <nobr>`fiatOperations[].fiatProvider`</nobr> | `string` | Fiat provider used by the fiat operation. |
| <nobr>`fiatOperations[].orderIdentity`</nobr> | `string` | Provider-side order reference used for support/reconciliation. |
| `fiatOperations[].createdAt` | `string` | Fiat operation creation date/time. |
| `cryptoOperations[].number` | `number` | Human-readable crypto operation number. |
| <nobr>`cryptoOperations[].accountType`</nobr> | `string` | Crypto operation account scope. Value: `WALLET`. |
| <nobr>`cryptoOperations[].operationType`</nobr> | `string` | Crypto operation type, for example `DEPOSIT` or `WITHDRAWAL`. |
| `cryptoOperations[].amount` | `number` | Crypto operation amount in crypto asset units. |
| <nobr>`cryptoOperations[].transactionId`</nobr> | `string` | Internal crypto transaction identifier. |
| `cryptoOperations[].asset` | `string` | Crypto asset code of the operation. |
| `cryptoOperations[].status` | `string` | Crypto transaction status. Allowed values: `NEW`, `PENDING_REVIEW`, `NOT_FOUND`, `REJECTED`, `TIMEOUT`, `INVALID_AMOUNT`, `ERROR`, `AML_ERROR`, `AML_BLOCKED`, `ARREST`, `SUBMITTING`, `SUBMITTED`, `PENDING`, `SELECTED`, `CONFIRMED`, `PENDING_RESOLVE`. |
| <nobr>`cryptoOperations[].submitTimeout`</nobr> | `string` | Crypto deposit timeout policy/mode. |
| <nobr>`cryptoOperations[].depositCryptoAddress`</nobr> | `string` | Blockchain address where client sends funds for crypto deposit. |
| `cryptoOperations[].network` | `string` | Blockchain network of the crypto operation. |
| `cryptoOperations[].txHash` | `string \| null` | Blockchain transaction hash after transfer is detected. |
| `cryptoOperations[].createdAt` | `string` | Crypto operation creation date/time. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to the merchant in access validation. |
| `400 Bad Request` | `HTTP` | Request parameters are invalid or cannot be parsed. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this client or endpoint. |

## 2) Deposit (`deposit`)

### Step 2.1 Create crypto deposit

Use this endpoint to create a crypto deposit operation and generate a destination address. Use the response to provide deposit instructions and track the operation by transaction id.

**POST** `/api/v2/exchange/merchant/balance/crypto/deposit`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | `Yes` | Account scope. Use `WALLET` for custodial wallet flow. |
| `asset.code` | `string` | `Yes` | Asset code used for the operation. |
| `asset.network` | `string` | `Yes` | Blockchain network of the selected crypto asset. |
| `asset.amount` | `number` | `Yes` | Operation amount in the selected asset. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `transactionId` | `string` | Transaction identifier for tracking operation status, support cases, and reconciliation. |
| <nobr>`depositCryptoAddress`</nobr> | `string` | Blockchain address that must be shown to the client as the destination for crypto deposit. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| <nobr>`400 ACTIVE_DEPOSIT_REQUEST_FOUND`</nobr> | `BUSINESS` | An uncompleted deposit already exists for this client/asset. |
| `400 INVALID_AMOUNT` | `BUSINESS` | Provided amount is invalid for deposit constraints. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to the merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 2.2 Get fiat payment methods

Use this endpoint to retrieve available fiat payment methods for the selected client and flow. Use the response to select a valid payment token for deposit or withdrawal requests.

**POST** `/api/v2/exchange/merchant/payment/method`

**Headers**
 - `x-api-key: {{x-api-key}}`

**Request**

```json
{
  "clientId": "{{clientId}}",
  "fiatAsset": "BYN",
  "orderType": "BUY",
  "destination": "EXCHANGE"
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
    }
]
```

### Headers

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `fiatAsset` | `string` | `No` | Fiat currency filter, for example `BYN`. |
| `orderType` | `string` | `No` | Operation type filter. Allowed values: `BUY` (fiat input), `SELL` (fiat output). |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |
| `providers` | `array of strings` | `No` | Optional list of allowed fiat providers. |
| `isCrypto` | `boolean` | `No` | Optional filter for crypto-related payment methods. |
| `countryGroup` | `string` | `No` | Optional country group filter. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Payment method token. Pass this value as `paymentToken` in fiat deposit/withdrawal or fiat-provider quote requests. |
| `number` | `string` | Masked payment method number shown to client. |
| `brand` | `string` | Payment method brand, for example `VISA`. |
| `providerId` | `string` | Payment provider identifier used in integrations and filters (for example `ASSIST`, `CA`, `MTS`). |
| `providerType` | `string` | Provider category/type returned by provider integration. Usually matches `providerId` for standard routes. |
| `status` | `string` | Payment method status. Allowed values: `ENABLED`, `DIRECTION_DISABLED`, `CURRENCY_DISABLED`, `UNKNOWN`. See status descriptions below. |
| `isRestricted` | `boolean` | Shows whether this payment method is restricted. |
| `isCrypto` | `boolean` | Shows whether method is crypto-related. |
| `country` | `string` | Payment method country. |
| `currency` | `string` | Primary fiat currency. |
| <nobr>`supportedCurrencies`</nobr> | `array of strings` | Fiat currencies supported by this payment method. |

#### Payment method `status` values

| Value | Description |
| --- | --- |
| `ENABLED` | Payment method is available for the requested direction and currency. |
| `DIRECTION_DISABLED` | Provider has no route for the requested `orderType` (`BUY`/`SELL`). |
| `CURRENCY_DISABLED` | Direction is supported, but `fiatAsset` is not in `supportedCurrencies` of the route. |
| `UNKNOWN` | `orderType` or `fiatAsset` was not provided in the request and the status cannot be resolved. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to the merchant. |
| `400 INVALID_ORDER_TYPE` | `BUSINESS` | `orderType` value is unsupported for payment method resolution. |
| `400 INVALID_FIAT_ASSET` | `BUSINESS` | `fiatAsset` value is unsupported for the selected flow. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |
| `429 Too Many Requests` | `HTTP` | Rate limit is exceeded for payment methods endpoint. |


### Step 2.3 Create fiat deposit

Use this endpoint to initiate a fiat deposit through a selected payment provider. Use the response to redirect the client to provider payment flow or render payment details.

**POST** `/api/v2/exchange/merchant/balance/fiat/deposit`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | `Yes` | Account scope. Use `WALLET` for custodial wallet flow. |
| <nobr>`fiatProviderType`</nobr> | `string` | `Yes` | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | `Conditional` | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | `Conditional` | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | `Yes` | Asset code used for the operation. |
| `asset.amount` | `number` | `Yes` | Operation amount in the selected asset. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `fiatPaymentLink` | `string` | Payment URL that client should open to complete fiat deposit. |
| `creationDate` | `string` | Creation timestamp in server date-time format. |
| `expirationMinutes` | `number` | Payment link lifetime in minutes. |
| `paymentDetails` | `object` | Provider-specific payment data. |
| <nobr>`paymentDetails.paymentLink`</nobr> | `string` | Provider payment URL. |
| <nobr>`paymentDetails.notificationPhoneNumber`</nobr> | `string \| null` | Phone number returned by provider when the payment scenario requires notification or additional confirmation. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| <nobr>`400 BALANCE_OPERATION_PROCESSING_ERROR`</nobr> | `BUSINESS` | Fiat provider operation cannot be started or processed. |
| `400 INVALID_PAYMENT_TOKEN` | `BUSINESS` | `paymentToken`/`internalToken` is invalid, restricted, or missing. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


## 3) Send (`withdrawal`)

### Step 3.1 Calculate crypto withdrawal

Use this endpoint to calculate crypto withdrawal fees and net payout before submission. Use the response to show final amounts and keep the calculation id for withdrawal creation.

**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `asset.amount` | `number` | `Yes` | Operation amount in the selected asset. |
| `asset.code` | `string` | `Yes` | Asset code used for the operation. |
| `asset.network` | `string` | `Yes` | Blockchain network of the selected crypto asset. |
| `toAddress` | `string` | `Yes` | Destination crypto address. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Calculation id used to create withdrawal. |
| `withdrawalAmount` | `string` | Original withdrawal amount. |
| `commissionAmount` | `string` | Commission amount for the operation. |
| `receivedAmount` | `string` | Net amount expected after fees/commissions. |
| `expirationDate` | `string` | Expiration timestamp in server date-time format, if returned. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_ADDRESS` | `BUSINESS` | Destination address is invalid for selected network or blocked as internal address. |
| `400 INVALID_AMOUNT` | `BUSINESS` | Amount is invalid (including fee greater than withdrawal amount). |
| <nobr>`400 ACTIVE_WITHDRAWAL_REQUEST_FOUND`</nobr> | `BUSINESS` | An uncompleted withdrawal already exists for this client/asset. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 3.2 Create crypto withdrawal

Use this endpoint to create a crypto withdrawal using a valid calculation context. Use the response to store transaction id and monitor withdrawal lifecycle.

**POST** `/api/v2/exchange/merchant/balance/crypto/withdrawal`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | `Yes` | Account scope. Use `WALLET` for custodial wallet flow. |
| `calculationId` | `string` | `Yes` | Calculation id returned by withdrawal calculation endpoint. |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `transactionId` | `string` | Created crypto withdrawal transaction identifier used for tracking status and support. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_CALCULATION` | `BUSINESS` | `calculationId` is not found or expired. |
| `400 INVALID_STATUS` | `BUSINESS` | Withdrawal operation status does not allow execution. |
| <nobr>`400 ACTIVE_WITHDRAWAL_REQUEST_FOUND`</nobr> | `BUSINESS` | Another uncompleted withdrawal blocks this operation. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 3.3 Calculate fiat withdrawal

Use this endpoint to calculate fiat withdrawal commission and expected payout amount. Use the response to confirm final withdrawal values with the client.

**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| <nobr>`fiatProviderType`</nobr> | `string` | `Yes` | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | `Conditional` | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | `Conditional` | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | `Yes` | Asset code used for the operation. |
| `asset.amount` | `number` | `Yes` | Operation amount in the selected asset. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string \| null` | Calculation identifier when provider creates a reusable calculation. Can be `null` when the fiat calculation is direct and no follow-up calculation id is required. |
| `withdrawalAmount` | `string` | Amount requested for withdrawal. |
| `commissionAmount` | `string` | Commission amount for the operation. |
| `receivedAmount` | `string` | Net amount expected after fees/commissions. |
| `expirationDate` | `string \| null` | Expiration timestamp in server date-time format, if returned. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| <nobr>`400 BALANCE_OPERATION_PROCESSING_ERROR`</nobr> | `BUSINESS` | Fiat withdrawal calculation cannot be produced by provider/flow. |
| `400 INVALID_PAYMENT_TOKEN` | `BUSINESS` | Payment token is invalid, unavailable, or unsupported. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 3.4 Create fiat withdrawal

Use this endpoint to create a fiat withdrawal from custodial wallet balance. Use the response to persist transaction id and track provider payout status.

**POST** `/api/v2/exchange/merchant/balance/fiat/withdrawal`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `accountType` | `string` | `Yes` | Account scope. Use `WALLET` for custodial wallet flow. |
| <nobr>`fiatProviderType`</nobr> | `string` | `Yes` | Fiat provider code for payment/payout processing. |
| `paymentToken` | `string` | `Conditional` | Payment method token used for fiat-provider operations. |
| `internalToken` | `string` | `Conditional` | Internal payment token used for provider-specific routing when applicable. |
| `asset.code` | `string` | `Yes` | Asset code used for the operation. |
| `asset.amount` | `number` | `Yes` | Operation amount in the selected asset. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `transactionId` | `string` | Created fiat withdrawal transaction identifier used for tracking payout status and reconciliation. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| <nobr>`400 BALANCE_OPERATION_PROCESSING_ERROR`</nobr> | `BUSINESS` | Fiat withdrawal cannot be started or provider rejected operation. |
| `400 INVALID_PAYMENT_TOKEN` | `BUSINESS` | Payment token/internal token is invalid or restricted. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


## 4) Buy crypto (`buy`)

Buy crypto flow is used when the client pays fiat through a provider and receives crypto to internal wallet balance. The flow is quote first, then order creation.

### Step 4.1 Create quote

Use this endpoint to create a buy crypto quote and lock rate/amounts for a short time. Use the response to display final buy terms and pass quote id to order creation.

**POST** `/api/v3/exchange/merchant/quote`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | `Yes` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | `Yes` | Source asset code. |
| `input.amount` | `number` | `Conditional` | Source amount for quote/order calculation. |
| `input.provider` | `string` | `Yes` | Fiat provider used for payment. |
| `input.token` | `string` | `Conditional` | Payment token used for provider payment. |
| `output.type` | `string` | `Yes` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | `Yes` | Destination asset code. |
| `output.amount` | `number` | `Conditional` | Destination amount for quote/order calculation. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Quote id used to create order. |
| `rate` | `string` | Rate pair for the operation. Display this value to the client as the final pair label. |
| `systemRateValue` | `string` | Base system rate at calculation time. |
| `exchangeRateValue` | `string` | Exchange rate applied to this quote/order. |
| `actualRateValue` | `string` | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `creationDate` | `string` | Creation timestamp in server date-time format. |
| `expirationDate` | `string` | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` | `string` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Source asset code. |
| `input.amount` | `string` | Source amount used in quote calculation. |
| <nobr>`input.feeAmount`</nobr> / <nobr>`output.feeAmount`</nobr> | `string` | Fee amount on each operation leg, in the corresponding leg asset currency (`input.asset` / `output.asset`). |
| `input.provider` | `string \| null` | Fiat provider code for source leg when source type is `FIAT_PROVIDER`. |
| `input.token` | `string \| null` | Payment token used by provider source leg, if required by provider flow. |
| `input.paymentType` | `string` | Fiat payment type selected by provider configuration. |
| `input.processingBank` | `string` | Processing bank selected for fiat provider route. |
| `output.type` | `string` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Destination asset code. |
| `output.amount` | `string` | Destination amount used in quote calculation. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote input is inconsistent or cannot be calculated for provided payment details. |
| `400 CURRENCY_NOT_FOUND` | `BUSINESS` | One of input/output assets is unknown. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Provided client id is invalid or not linked to merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |
| `429 Too Many Requests` | `HTTP` | Rate limit is exceeded for quote creation endpoint. |


### Step 4.2 Create buy order

Use this endpoint to create a buy order from a valid non-expired quote. Use the response to track order execution and render provider/payment metadata.

**POST** `/api/v3/exchange/merchant/order`

**Headers**
 - `x-api-key: {{x-api-key}}`

**Request**

```json
{
  "quoteId": "47b2985a-2fe3-427c-9a18-6b16736c460e",
  "destination": "EXCHANGE",
  "returnUrl": "https://google.com",
  "failUrl": "https://google.com"
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | `string` | `Yes` | Quote identifier returned by quote creation; required to create an order before quote expiration. |
| <nobr>`destinationCryptoAddress`</nobr> | `string` | `No` | Destination wallet address for crypto-out flows (used when `output.type` is `CRYPTO_TRANSFER`). |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |
| `bankIdentifier` | `string` | `No` | Optional bank identifier used by selected fiat provider route. |
| `returnUrl` | `string` | `No` | URL the client should be redirected to on successful payment flow. |
| `failUrl` | `string` | `No` | URL the client should be redirected to on failed payment flow. |
| `additionalTimeout` | `boolean` | `No` | Extended-timeout flag for slow payment flows. |
| <nobr>`outputPaymentProcessingType`</nobr> | `string` | `No` | Optional payment processing type for the output leg. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Order id. |
| `number` | `number` | Human-readable order number. |
| `conditions` | `object` | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | Destination asset code in calculation conditions. |
| <nobr>`conditions.fromGrossAmount`</nobr> | `string` | Source gross amount before source-side fees. |
| <nobr>`conditions.fromNetAmount`</nobr> | `string` | Source net amount in calculation conditions. |
| <nobr>`conditions.fromFeeAmount`</nobr> | `string` | Source-side fee amount in calculation conditions, in `conditions.fromAsset` currency. |
| <nobr>`conditions.toGrossAmount`</nobr> | `string` | Destination gross amount before destination-side fees. |
| <nobr>`conditions.toNetAmount`</nobr> | `string` | Destination net amount in calculation conditions. |
| <nobr>`conditions.toFeeAmount`</nobr> | `string` | Destination-side fee amount in calculation conditions, in `conditions.toAsset` currency. |
| `conditions.rate` | `string` | Rate pair in calculation conditions. |
| <nobr>`conditions.systemRateValue`</nobr> | `string` | Base system rate at calculation time. |
| <nobr>`conditions.exchangeRateValue`</nobr> | `string` | Exchange rate value in calculation conditions. |
| <nobr>`conditions.actualRateValue`</nobr> | `string` | Effective client-facing rate value in calculation conditions. |
| `recalculationReason` | `string \| null` | Recalculation reason when quote/order amounts were adjusted by system logic; `null` when no recalculation happened. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string \| null` | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `completionDate` | `string \| null` | Order completion timestamp when order is finished; `null` while order is still active. |
| `creationDate` | `string` | Creation timestamp in server date-time format. |
| `sessionId` | `string \| null` | Optional client session identifier bound to this order. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` / `output.type` | `string` | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | Operation amount for each leg. |
| <nobr>`input.transactionAmount`</nobr> / <nobr>`output.transactionAmount`</nobr> | `string` | Provider/settlement amount for operation leg. |
| <nobr>`input.feeAmount`</nobr> / <nobr>`output.feeAmount`</nobr> | `string` | Fee amount on each operation leg, in the corresponding leg asset currency (`input.asset` / `output.asset`). |
| `input.status` / `output.status` | `string` | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| <nobr>`input.failureMessage`</nobr> / <nobr>`output.failureMessage`</nobr> | `string \| null` | Failure reason for a specific operation leg. |
| <nobr>`input.expirationDate`</nobr> / <nobr>`output.expirationDate`</nobr> | `string \| null` | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string \| null` | Provider code for fiat-provider operation leg. |
| <nobr>`input.paymentType`</nobr> / <nobr>`output.paymentType`</nobr> | `string \| null` | Provider payment type metadata (for example `P2P`, `SBP`). |
| <nobr>`input.processingBank`</nobr> / <nobr>`output.processingBank`</nobr> | `string \| null` | Processing bank metadata for fiat-provider operation leg. |
| `input.clientBank` / `output.clientBank` | `string \| null` | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string \| null` | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string \| null` | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string \| null` | Provider payment URL for redirect/confirmation flows. |
| <nobr>`input.processorTransactionId`</nobr> / <nobr>`output.processorTransactionId`</nobr> | `string \| null` | External provider transaction id for reconciliation. |
| <nobr>`input.processorTransactionNumber`</nobr> / <nobr>`output.processorTransactionNumber`</nobr> | `string \| null` | External provider transaction number/reference shown by provider systems for support and reconciliation. |
| `input.post` / `output.post` | `string \| null` | Additional provider payload or form-POST metadata when present. |
| <nobr>`input.paymentSystem`</nobr> / <nobr>`output.paymentSystem`</nobr> | `string \| null` | Payment system metadata returned by provider integration. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 QUOTE_NOT_FOUND` | `BUSINESS` | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote exists but cannot be used for order creation. |
| <nobr>`400 INVALID_CLIENT_STATUS`</nobr> | `BUSINESS` | Client status/checks do not allow order creation (for example testing not completed). |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no access to quote/client used by this order. |


## 5) Sell crypto (`sell`)

Sell crypto flow is used when the client sells crypto from internal wallet balance and receives fiat through a provider.

### Step 5.1 Create quote

Use this endpoint to create a sell crypto quote and lock rate/amounts for sell flow. Use the response to show sell terms and pass quote id to order creation.

**POST** `/api/v3/exchange/merchant/quote`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | `Yes` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | `Yes` | Source asset code. |
| `input.amount` | `number` | `Conditional` | Source amount for quote/order calculation. |
| `output.type` | `string` | `Yes` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | `Yes` | Destination asset code. |
| `output.provider` | `string` | `Yes` | Fiat provider. |
| `output.token` | `string` | `Conditional` | Payment token for receiving fiat. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Quote id used for order creation. |
| `rate` | `string` | Rate pair for the operation. Display this value to the client as the final pair label. |
| `systemRateValue` | `string` | Base system rate at calculation time. |
| `exchangeRateValue` | `string` | Exchange rate applied to this quote/order. |
| `actualRateValue` | `string` | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `creationDate` | `string` | Creation timestamp in server date-time format. |
| `expirationDate` | `string` | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` | `string` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Source asset code. |
| `input.amount` | `string` | Source amount used in quote calculation. |
| `input.feeAmount` | `string` | Fee amount on source leg, in `input.asset` currency. |
| `output.type` | `string` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Destination asset code. |
| `output.amount` | `string` | Destination amount for quote/order calculation. |
| `output.feeAmount` | `string` | Fiat provider/exchange fee amount, in `output.asset` currency. |
| `output.provider` | `string \| null` | Fiat provider code for destination leg when destination type is `FIAT_PROVIDER`. |
| `output.token` | `string \| null` | Payment token used by provider destination leg, if required by provider flow. |
| `output.paymentType` | `string` | Fiat payment type. |
| <nobr>`output.processingBank`</nobr> | `string` | Processing bank selected by provider route. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | `BUSINESS` | Input/output asset is unknown. |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Client id is invalid or not linked to merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 5.2 Create sell order

Use this endpoint to create a sell order from a valid non-expired quote. Use the response to track order progress and payout operation details.

**POST** `/api/v3/exchange/merchant/order`

**Headers**
 - `x-api-key: {{x-api-key}}`

**Request**

```json
{
  "quoteId": "47b2985a-2fe3-427c-9a18-6b16736c460e",
  "destination": "EXCHANGE",
  "returnUrl": "https://google.com",
  "failUrl": "https://google.com"
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | `string` | `Yes` | Quote identifier returned by quote creation; required to create an order before quote expiration. |
| <nobr>`destinationCryptoAddress`</nobr> | `string` | `No` | Destination wallet address for crypto-out flows (used when `output.type` is `CRYPTO_TRANSFER`). |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |
| `bankIdentifier` | `string` | `No` | Optional bank identifier used by selected fiat provider route. |
| `returnUrl` | `string` | `No` | URL the client should be redirected to on successful payment flow. |
| `failUrl` | `string` | `No` | URL the client should be redirected to on failed payment flow. |
| `additionalTimeout` | `boolean` | `No` | Extended-timeout flag for slow payment flows. |
| <nobr>`outputPaymentProcessingType`</nobr> | `string` | `No` | Optional payment processing type for the output leg. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Order id. |
| `number` | `number` | Human-readable order number. |
| `conditions` | `object` | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | Destination asset code in calculation conditions. |
| <nobr>`conditions.fromGrossAmount`</nobr> | `string` | Source gross amount before source-side fees. |
| <nobr>`conditions.fromNetAmount`</nobr> | `string` | Source net amount in calculation conditions. |
| <nobr>`conditions.fromFeeAmount`</nobr> | `string` | Source-side fee amount in calculation conditions, in `conditions.fromAsset` currency. |
| <nobr>`conditions.toGrossAmount`</nobr> | `string` | Destination gross amount before destination-side fees. |
| <nobr>`conditions.toNetAmount`</nobr> | `string` | Destination net amount in calculation conditions. |
| <nobr>`conditions.toFeeAmount`</nobr> | `string` | Destination-side fee amount in calculation conditions, in `conditions.toAsset` currency. |
| `conditions.rate` | `string` | Rate pair in calculation conditions. |
| <nobr>`conditions.systemRateValue`</nobr> | `string` | Base system rate at calculation time. |
| <nobr>`conditions.exchangeRateValue`</nobr> | `string` | Exchange rate value in calculation conditions. |
| <nobr>`conditions.actualRateValue`</nobr> | `string` | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string \| null` | Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` / `output.type` | `string` | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | Operation amount for each leg. |
| <nobr>`input.transactionAmount`</nobr> / <nobr>`output.transactionAmount`</nobr> | `string` | Provider/settlement amount for operation leg. |
| <nobr>`input.feeAmount`</nobr> / <nobr>`output.feeAmount`</nobr> | `string` | Fee amount on each operation leg, in the corresponding leg asset currency (`input.asset` / `output.asset`). |
| `input.status` / `output.status` | `string` | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| <nobr>`input.failureMessage`</nobr> / <nobr>`output.failureMessage`</nobr> | `string \| null` | Failure reason for a specific operation leg. |
| <nobr>`input.expirationDate`</nobr> / <nobr>`output.expirationDate`</nobr> | `string \| null` | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string \| null` | Provider code for fiat-provider operation leg. |
| <nobr>`input.paymentType`</nobr> / <nobr>`output.paymentType`</nobr> | `string \| null` | Provider payment type metadata (for example `P2P`, `SBP`). |
| <nobr>`input.processingBank`</nobr> / <nobr>`output.processingBank`</nobr> | `string \| null` | Processing bank metadata for fiat-provider operation leg. |
| `input.clientBank` / `output.clientBank` | `string \| null` | Client bank metadata if returned by provider route. |
| `input.fromToken` / `output.fromToken` | `string \| null` | Source payment token used by provider leg. |
| `input.toToken` / `output.toToken` | `string \| null` | Destination payment token used by provider leg. |
| `input.link` / `output.link` | `string \| null` | Provider payment URL for redirect/confirmation flows. |
| <nobr>`input.processorTransactionId`</nobr> / <nobr>`output.processorTransactionId`</nobr> | `string \| null` | External provider transaction id for reconciliation. |
| `input.post` / `output.post` | `string \| null` | Additional provider payload or form-POST metadata when present. |
| <nobr>`input.paymentSystem`</nobr> / <nobr>`output.paymentSystem`</nobr> | `string \| null` | Payment system metadata returned by provider integration. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 QUOTE_NOT_FOUND` | `BUSINESS` | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote exists but cannot be used for sell order creation. |
| <nobr>`400 INSUFFICIENT_BALANCE`</nobr> | `BUSINESS` | Client internal balance is not enough for requested sell operation. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no access to quote/client used by this order. |


## 6) Operation History / Details

History endpoint is used to fetch created orders and their operation details for support, reconciliation, and client-facing history.

### Step 6.1 Get order history/details

Use this endpoint to fetch paged order history with optional filters and detailed operation data. Use the response to power history UI, reporting, and support investigations.

**POST** `/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `page` | `number` | `No` | Query parameter with page index. Default example uses `0`. |
| `size` | `number` | `No` | Query parameter with page size. Default example uses `20`. |
| `sort` | `string` | `No` | Query parameter for sorting, for example `creationDate,desc`. |
| `clientId` | `string` | `Conditional` | Required when `externalClientId` is not provided. Used to scope the request to a specific client. |
| <nobr>`externalClientId`</nobr> | `string` | `Conditional` | Required when `clientId` is not provided. External client identifier. |
| `numbers` | `array of number` | `No` | Filter by human-readable order numbers. |
| `orderIds` | `array of string` | `No` | Filter by order ids (UUID). |
| `sessionIds` | `array of string` | `No` | Filter by SDK session ids (UUID). |
| `statuses` | `array of string` | `No` | Filter by order status. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| <nobr>`creationDateFrame`</nobr> | `object` | `No` | Creation date range filter. Object `{ "from": "ISO-8601", "to": "ISO-8601" }`. |
| <nobr>`completionDateFrame`</nobr> | `object` | `No` | Completion date range filter. Object `{ "from": "ISO-8601", "to": "ISO-8601" }`. |
| `inputAssets` | `array of string` | `No` | Filter by source asset codes. |
| `outputAssets` | `array of string` | `No` | Filter by destination asset codes. |
| `assets` | `array of string` | `No` | Asset filter applied to either source or destination leg. |
| `inputAmount` | `object` | `No` | Source amount range filter `{ "from": number, "to": number }`. |
| `outputAmount` | `object` | `No` | Destination amount range filter `{ "from": number, "to": number }`. |
| `destinations` | `array of string` | `No` | Filter by flow destination. |
| <nobr>`fiatTransactionProviders`</nobr> | `array of string` | `No` | Filter by fiat provider id (for example `ASSIST`). |
| <nobr>`cryptoTransactionAddresses`</nobr> | `array of string` | `No` | Filter by crypto destination addresses. |
| <nobr>`cryptoTransactionHashes`</nobr> | `array of string` | `No` | Filter by crypto blockchain transaction hashes. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `content` | `array of objects` | Page content with order objects. |
| `id` | `string` | Order id. |
| `number` | `number` | Human-readable order number. |
| `conditions` | `object` | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | Destination asset code in calculation conditions. |
| <nobr>`conditions.fromGrossAmount`</nobr> | `string` | Source gross amount before source-side fees. |
| <nobr>`conditions.fromNetAmount`</nobr> | `string` | Source net amount in calculation conditions. |
| <nobr>`conditions.fromFeeAmount`</nobr> | `string` | Source-side fee amount in calculation conditions, in `conditions.fromAsset` currency. |
| <nobr>`conditions.toGrossAmount`</nobr> | `string` | Destination gross amount before destination-side fees. |
| <nobr>`conditions.toNetAmount`</nobr> | `string` | Destination net amount in calculation conditions. |
| <nobr>`conditions.toFeeAmount`</nobr> | `string` | Destination-side fee amount in calculation conditions, in `conditions.toAsset` currency. |
| `conditions.rate` | `string` | Rate pair in calculation conditions. |
| <nobr>`conditions.systemRateValue`</nobr> | `string` | Base system rate at calculation time. |
| <nobr>`conditions.exchangeRateValue`</nobr> | `string` | Exchange rate value in calculation conditions. |
| <nobr>`conditions.actualRateValue`</nobr> | `string` | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string \| null` | Human-readable reason of failure for historical orders; useful for support and merchant-side audit. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` / `output.type` | `string` | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | Operation amount for each leg. |
| <nobr>`input.transactionAmount`</nobr> / <nobr>`output.transactionAmount`</nobr> | `string` | Provider/settlement amount for operation leg. |
| <nobr>`input.feeAmount`</nobr> / <nobr>`output.feeAmount`</nobr> | `string` | Fee amount on each operation leg, in the corresponding leg asset currency (`input.asset` / `output.asset`). |
| `input.status` / `output.status` | `string` | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| <nobr>`input.failureMessage`</nobr> / <nobr>`output.failureMessage`</nobr> | `string \| null` | Failure reason for a specific operation leg. |
| `input.provider` / `output.provider` | `string \| null` | Provider code for fiat-provider operation leg. |
| <nobr>`input.paymentType`</nobr> / <nobr>`output.paymentType`</nobr> | `string \| null` | Provider payment type metadata (for example `P2P`, `SBP`). |
| <nobr>`input.processingBank`</nobr> / <nobr>`output.processingBank`</nobr> | `string \| null` | Processing bank metadata for fiat-provider operation leg. |
| `input.link` / `output.link` | `string \| null` | Provider payment URL for redirect/confirmation flows. |
| <nobr>`input.processorTransactionId`</nobr> / <nobr>`output.processorTransactionId`</nobr> | `string \| null` | External provider transaction id for reconciliation. |
| `totalElements` | `number` | Total number of matching orders. |
| `totalPages` | `number` | Total number of pages. |
| `number` | `number` | Current page number. |
| `size` | `number` | Current page size. |
| `first` | `boolean` | `true` when current page is first page. |
| `last` | `boolean` | `true` when current page is last page. |
| `numberOfElements` | `number` | Number of items on current page. |
| `empty` | `boolean` | `true` when `content` array is empty. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 Bad Request` | `HTTP` | Filter body is invalid or missing required client filter (`Client id is required`). |
| `400 CLIENT_NOT_FOUND` | `BUSINESS` | Provided client id/external client id is invalid for this merchant. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |
| `429 Too Many Requests` | `HTTP` | Rate limit is exceeded for history endpoint. |


## 7) Conversion (`conversion`)

For custodial wallet, conversion goes through internal balance (`USER_BALANCE` / `INTERNAL_BALANCE`).

### Step 7.1 Check limits

Use this endpoint to check conversion min/max limits for the selected asset pair. Use the response to validate entered amounts before quote creation.

**POST** `/api/v3/exchange/merchant/limit`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `fromAsset` | `string` | `Yes` | Source asset. |
| <nobr>`fromPaymentDetails.type`</nobr> | `string` | `Yes` | Source payment type. For conversion use `INTERNAL_BALANCE`. |
| `toAsset` | `string` | `Yes` | Destination asset. |
| <nobr>`toPaymentDetails.type`</nobr> | `string` | `Yes` | Destination payment type. For conversion use `INTERNAL_BALANCE`. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `fromMinAmount` | `string` | Minimum allowed source amount. |
| `fromMaxAmount` | `string` | Maximum allowed source amount. |
| `toMinAmount` | `string` | Minimum allowed destination amount. |
| `toMaxAmount` | `string` | Maximum allowed destination amount. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_MERCHANT_ID` | `BUSINESS` | No merchant identifier could be resolved for limit calculation context. |
| `400 INVALID_QUOTE` | `BUSINESS` | Limit request contains invalid pair/payment details for calculation context. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |
| `429 Too Many Requests` | `HTTP` | Rate limit is exceeded for limit endpoint. |


### Step 7.2 Create quote

Use this endpoint to create a conversion quote between internal balance assets. Use the response to show conversion terms and pass quote id to swap creation.

**POST** `/api/v3/exchange/merchant/quote`

**Headers**
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | `Yes` | Client identifier used to scope the request to a specific client. |
| `input.type` | `string` | `Yes` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | `Yes` | Source asset code. |
| `input.amount` | `number` | `Conditional` | Source amount for quote/order calculation. |
| `output.type` | `string` | `Yes` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | `Yes` | Destination asset code. |
| `output.amount` | `number` | `Conditional` | Destination amount for quote/order calculation. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Quote id used for conversion order. |
| `rate` | `string` | Rate pair for the operation. Display this value to the client as the final pair label. |
| `systemRateValue` | `string` | Base system rate at calculation time. |
| `exchangeRateValue` | `string` | Exchange rate applied to this quote/order. |
| `actualRateValue` | `string` | Effective client-facing rate after adjustments. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `creationDate` | `string` | Creation timestamp in server date-time format. |
| `expirationDate` | `string` | Expiration timestamp in server date-time format, if returned. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` | `string` | Source operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` | `string` | Source asset code. |
| `input.amount` | `string` | Source amount used in quote calculation. |
| `input.feeAmount` | `string` | Fee amount on source leg, in `input.asset` currency. |
| `output.type` | `string` | Destination operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `output.asset` | `string` | Destination asset code. |
| `output.amount` | `string` | Destination amount used in quote calculation. |
| `output.feeAmount` | `string` | Fee amount on destination leg, in `output.asset` currency. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote input is inconsistent or cannot be calculated. |
| `400 CURRENCY_NOT_FOUND` | `BUSINESS` | Asset id is unknown for conversion pair. |
| <nobr>`400 INVALID_CLIENT_STATUS`</nobr> | `BUSINESS` | Client status/checks do not allow conversion quote creation. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no permission for this operation. |


### Step 7.3 Create swap operation

Use this endpoint to create and execute a swap operation from a valid conversion quote. Use the response to persist order identifiers and final operation statuses.

**POST** `/api/v3/exchange/merchant/order`

**Headers**
 - `x-api-key: {{x-api-key}}`

**Request**

```json
{
  "quoteId": "47b2985a-2fe3-427c-9a18-6b16736c460e",
  "destination": "EXCHANGE",
  "returnUrl": "https://google.com",
  "failUrl": "https://google.com"
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
| --- | --- | --- | --- |
| <nobr>`x-api-key`</nobr> | `string` | `Yes` | Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment. |

### Request

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | `string` | `Yes` | Quote identifier returned by quote creation; required to create an order before quote expiration. |
| <nobr>`destinationCryptoAddress`</nobr> | `string` | `No` | Destination wallet address for crypto-out flows (used when `output.type` is `CRYPTO_TRANSFER`). |
| `comment` | `string` | `No` | Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet). |
| `bankIdentifier` | `string` | `No` | Optional bank identifier used by selected fiat provider route. |
| `returnUrl` | `string` | `No` | URL the client should be redirected to on successful payment flow. |
| `failUrl` | `string` | `No` | URL the client should be redirected to on failed payment flow. |
| `additionalTimeout` | `boolean` | `No` | Extended-timeout flag for slow payment flows. |
| <nobr>`outputPaymentProcessingType`</nobr> | `string` | `No` | Optional payment processing type for the output leg. |
| `destination` | `string` | `No` | Optional flow destination filter. Recommended value: `EXCHANGE`. |

### Response

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Conversion order id. |
| `number` | `number` | Human-readable order number. |
| `conditions` | `object` | Detailed order/quote calculation breakdown. |
| `conditions.fromAsset` | `string` | Source asset code in calculation conditions. |
| `conditions.toAsset` | `string` | Destination asset code in calculation conditions. |
| <nobr>`conditions.fromGrossAmount`</nobr> | `string` | Source gross amount before source-side fees. |
| <nobr>`conditions.fromNetAmount`</nobr> | `string` | Source net amount in calculation conditions. |
| <nobr>`conditions.fromFeeAmount`</nobr> | `string` | Source-side fee amount in calculation conditions, in `conditions.fromAsset` currency. |
| <nobr>`conditions.toGrossAmount`</nobr> | `string` | Destination gross amount before destination-side fees. |
| <nobr>`conditions.toNetAmount`</nobr> | `string` | Destination net amount in calculation conditions. |
| <nobr>`conditions.toFeeAmount`</nobr> | `string` | Destination-side fee amount in calculation conditions, in `conditions.toAsset` currency. |
| `conditions.rate` | `string` | Rate pair in calculation conditions. |
| <nobr>`conditions.systemRateValue`</nobr> | `string` | Base system rate at calculation time. |
| <nobr>`conditions.exchangeRateValue`</nobr> | `string` | Exchange rate value in calculation conditions. |
| <nobr>`conditions.actualRateValue`</nobr> | `string` | Effective client-facing rate value in calculation conditions. |
| `clientId` | `string` | Client identifier used to scope the request to a specific client. |
| `status` | `string` | Current order lifecycle state. Allowed values: `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| `failureMessage` | `string \| null` | Human-readable reason of failure when conversion cannot be completed; use for support/debugging. |
| `input` | `object` | Source operation details object. |
| `output` | `object` | Destination operation details object. |
| `input.type` / `output.type` | `string` | Operation channel. Allowed values: `INTERNAL_BALANCE`, `FIAT_PROVIDER`, `CRYPTO_TRANSFER`. |
| `input.asset` / `output.asset` | `string` | Asset code used for each operation leg. |
| `input.amount` / `output.amount` | `string` | Operation amount for each leg. |
| <nobr>`input.transactionAmount`</nobr> / <nobr>`output.transactionAmount`</nobr> | `string` | Provider/settlement amount for operation leg. |
| <nobr>`input.feeAmount`</nobr> / <nobr>`output.feeAmount`</nobr> | `string` | Fee amount on each operation leg, in the corresponding leg asset currency (`input.asset` / `output.asset`). |
| `input.status` / `output.status` | `string` | Leg status. Allowed values: `NEW`, `PROCESSING`, `EXPIRED`, `COMPLETED`, `FAILED`. |
| <nobr>`input.failureMessage`</nobr> / <nobr>`output.failureMessage`</nobr> | `string \| null` | Failure reason for a specific operation leg. |
| <nobr>`input.expirationDate`</nobr> / <nobr>`output.expirationDate`</nobr> | `string \| null` | Expiration timestamp for operation leg context, if provided. |
| `input.provider` / `output.provider` | `string \| null` | Provider code for fiat-provider operation leg. |
| <nobr>`input.paymentType`</nobr> / <nobr>`output.paymentType`</nobr> | `string \| null` | Provider payment type metadata (for example `P2P`, `SBP`). |
| <nobr>`input.processingBank`</nobr> / <nobr>`output.processingBank`</nobr> | `string \| null` | Processing bank metadata for fiat-provider operation leg. |
| `input.link` / `output.link` | `string \| null` | Provider payment URL for redirect/confirmation flows. |
| <nobr>`input.processorTransactionId`</nobr> / <nobr>`output.processorTransactionId`</nobr> | `string \| null` | External provider transaction id for reconciliation. |

### Errors

| Name | Code | Description |
| --- | --- | --- |
| `400 QUOTE_NOT_FOUND` | `BUSINESS` | Quote id is missing, expired, or unknown. |
| `400 INVALID_QUOTE` | `BUSINESS` | Quote exists but cannot be used for conversion order creation. |
| <nobr>`400 INSUFFICIENT_BALANCE`</nobr> | `BUSINESS` | Source internal balance is not enough to execute conversion. |
| `401 Unauthorized` | `HTTP` | `x-api-key` is missing, invalid, or expired. |
| `403 Forbidden` | `HTTP` | Merchant has no access to quote/client used by this conversion. |


