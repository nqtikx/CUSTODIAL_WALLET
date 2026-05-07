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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="203" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="577">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="226" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="634">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatAssets</td>
      <td>array of objects</td>
      <td>List of fiat assets that can be shown to the client as available wallet currencies for this merchant flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatAssets[].id</td>
      <td>string</td>
      <td>Internal asset identifier used in API requests and routing logic.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatAssets[].code</td>
      <td>string</td>
      <td>Currency code that can be displayed to the client in UI.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoAssets</td>
      <td>array of objects</td>
      <td>List of crypto assets/networks that can be used in deposit, withdrawal, buy, sell, or conversion flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoAssets[].id</td>
      <td>string</td>
      <td>Internal crypto asset identifier used in API requests; may include network-specific suffixes such as USDT_TRC.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoAssets[].code</td>
      <td>string</td>
      <td>Asset ticker displayed to the client; can differ from id when asset is network-specific.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoAssets[].network</td>
      <td>string</td>
      <td>Blockchain network that must be used for deposits/withdrawals of this asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoAssets[].protocol</td>
      <td>string</td>
      <td>Token protocol shown to prevent sending funds through the wrong network.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_DESTINATION</td>
      <td>BUSINESS</td>
      <td>destination value cannot be mapped to supported enum for merchant assets.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">429 Too Many Requests</td>
      <td>HTTP</td>
      <td>Rate limit is exceeded for this endpoint.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="194" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="586">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="258" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="602">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations</td>
      <td>array of objects</td>
      <td>Current fiat wallet operations.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations</td>
      <td>array of objects</td>
      <td>Current crypto wallet operations.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].number</td>
      <td>number</td>
      <td>Human-readable fiat operation number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].accountType</td>
      <td>string</td>
      <td>Fiat operation account scope. Value: WALLET.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].operationType</td>
      <td>string</td>
      <td>Fiat operation type, for example DEPOSIT or WITHDRAWAL.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].amount</td>
      <td>number</td>
      <td>Fiat operation amount in fiat asset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].transactionId</td>
      <td>string</td>
      <td>Internal fiat transaction identifier.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].asset</td>
      <td>string</td>
      <td>Fiat asset code of the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].status</td>
      <td>string</td>
      <td>Fiat transaction status. Allowed values: NEW, PENDING_REVIEW, REJECTED, TIMEOUT, DECLINED, INVALID_AMOUNT, ERROR, AML_BLOCKED, PENDING, PROCESSING, APPROVED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].fiatProvider</td>
      <td>string</td>
      <td>Fiat provider used by the fiat operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].orderIdentity</td>
      <td>string</td>
      <td>Provider-side order reference used for support/reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatOperations[].createdAt</td>
      <td>string</td>
      <td>Fiat operation creation date/time.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].number</td>
      <td>number</td>
      <td>Human-readable crypto operation number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].accountType</td>
      <td>string</td>
      <td>Crypto operation account scope. Value: WALLET.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].operationType</td>
      <td>string</td>
      <td>Crypto operation type, for example DEPOSIT or WITHDRAWAL.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].amount</td>
      <td>number</td>
      <td>Crypto operation amount in crypto asset units.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].transactionId</td>
      <td>string</td>
      <td>Internal crypto transaction identifier.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].asset</td>
      <td>string</td>
      <td>Crypto asset code of the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].status</td>
      <td>string</td>
      <td>Crypto transaction status. Allowed values: NEW, PENDING_REVIEW, NOT_FOUND, REJECTED, TIMEOUT, INVALID_AMOUNT, ERROR, AML_ERROR, AML_BLOCKED, ARREST, SUBMITTING, SUBMITTED, PENDING, SELECTED, CONFIRMED, PENDING_RESOLVE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].submitTimeout</td>
      <td>string</td>
      <td>Crypto deposit timeout policy/mode.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].depositCryptoAddress</td>
      <td>string</td>
      <td>Blockchain address where client sends funds for crypto deposit.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].network</td>
      <td>string</td>
      <td>Blockchain network of the crypto operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].txHash</td>
      <td>string | null</td>
      <td>Blockchain transaction hash after transfer is detected.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoOperations[].createdAt</td>
      <td>string</td>
      <td>Crypto operation creation date/time.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to the merchant in access validation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 Bad Request</td>
      <td>HTTP</td>
      <td>Request parameters are invalid or cannot be parsed.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this client or endpoint.</td>
    </tr>
  </tbody>
</table>

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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="209" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="571">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">accountType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Account scope. Use WALLET for custodial wallet flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.code</td>
      <td>string</td>
      <td>Yes</td>
      <td>Asset code used for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.network</td>
      <td>string</td>
      <td>Yes</td>
      <td>Blockchain network of the selected crypto asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.amount</td>
      <td>number</td>
      <td>Yes</td>
      <td>Operation amount in the selected asset.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">transactionId</td>
      <td>string</td>
      <td>Transaction identifier for tracking operation status, support cases, and reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">depositCryptoAddress</td>
      <td>string</td>
      <td>Blockchain address that must be shown to the client as the destination for crypto deposit.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="254" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="626">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 ACTIVE_DEPOSIT_REQUEST_FOUND</td>
      <td>BUSINESS</td>
      <td>An uncompleted deposit already exists for this client/asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_AMOUNT</td>
      <td>BUSINESS</td>
      <td>Provided amount is invalid for deposit constraints.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to the merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="206" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="574">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatAsset</td>
      <td>string</td>
      <td>No</td>
      <td>Fiat currency filter, for example BYN.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">orderType</td>
      <td>string</td>
      <td>No</td>
      <td>Operation type filter. Allowed values: BUY (fiat input), SELL (fiat output).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">providers</td>
      <td>array of strings</td>
      <td>No</td>
      <td>Optional list of allowed fiat providers.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">isCrypto</td>
      <td>boolean</td>
      <td>No</td>
      <td>Optional filter for crypto-related payment methods.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">countryGroup</td>
      <td>string</td>
      <td>No</td>
      <td>Optional country group filter.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Payment method token. Pass this value as paymentToken in fiat deposit/withdrawal or fiat-provider quote requests.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>string</td>
      <td>Masked payment method number shown to client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">brand</td>
      <td>string</td>
      <td>Payment method brand, for example VISA.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">providerId</td>
      <td>string</td>
      <td>Payment provider identifier used in integrations and filters (for example ASSIST, CA, MTS).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">providerType</td>
      <td>string</td>
      <td>Provider category/type returned by provider integration. Usually matches providerId for standard routes.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">status</td>
      <td>string</td>
      <td>Payment method status. Allowed values: ENABLED, DIRECTION_DISABLED, CURRENCY_DISABLED, UNKNOWN. See status descriptions below.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">isRestricted</td>
      <td>boolean</td>
      <td>Shows whether this payment method is restricted.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">isCrypto</td>
      <td>boolean</td>
      <td>Shows whether method is crypto-related.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">country</td>
      <td>string</td>
      <td>Payment method country.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">currency</td>
      <td>string</td>
      <td>Primary fiat currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">supportedCurrencies</td>
      <td>array of strings</td>
      <td>Fiat currencies supported by this payment method.</td>
    </tr>
  </tbody>
</table>

### Payment method status values

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Status</th>
      <th width="680">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">ENABLED</td>
      <td>Payment method can be used for the selected flow, direction, and currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">DIRECTION_DISABLED</td>
      <td>Payment method exists, but is not available for selected orderType.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">CURRENCY_DISABLED</td>
      <td>Payment method exists, but does not support selected fiatAsset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">UNKNOWN</td>
      <td>Status cannot be resolved because required filters were not provided.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to the merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_ORDER_TYPE</td>
      <td>BUSINESS</td>
      <td>orderType value is unsupported for payment method resolution.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_FIAT_ASSET</td>
      <td>BUSINESS</td>
      <td>fiatAsset value is unsupported for the selected flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">429 Too Many Requests</td>
      <td>HTTP</td>
      <td>Rate limit is exceeded for payment methods endpoint.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="218" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="562">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">accountType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Account scope. Use WALLET for custodial wallet flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatProviderType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Fiat provider code for payment/payout processing.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Payment method token used for fiat-provider operations.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">internalToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Internal payment token used for provider-specific routing when applicable.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.code</td>
      <td>string</td>
      <td>Yes</td>
      <td>Asset code used for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.amount</td>
      <td>number</td>
      <td>Yes</td>
      <td>Operation amount in the selected asset.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="256" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="604">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatPaymentLink</td>
      <td>string</td>
      <td>Payment URL that client should open to complete fiat deposit.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDate</td>
      <td>string</td>
      <td>Creation timestamp in server date-time format.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationMinutes</td>
      <td>number</td>
      <td>Payment link lifetime in minutes.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentDetails</td>
      <td>object</td>
      <td>Provider-specific payment data.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentDetails.paymentLink</td>
      <td>string</td>
      <td>Provider payment URL.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentDetails.notificationPhoneNumber</td>
      <td>string | null</td>
      <td>Phone number returned by provider when the payment scenario requires notification or additional confirmation.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="266" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="614">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 BALANCE_OPERATION_PROCESSING_ERROR</td>
      <td>BUSINESS</td>
      <td>Fiat provider operation cannot be started or processed.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_PAYMENT_TOKEN</td>
      <td>BUSINESS</td>
      <td>paymentToken/internalToken is invalid, restricted, or missing.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="209" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="571">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.amount</td>
      <td>number</td>
      <td>Yes</td>
      <td>Operation amount in the selected asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.code</td>
      <td>string</td>
      <td>Yes</td>
      <td>Asset code used for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.network</td>
      <td>string</td>
      <td>Yes</td>
      <td>Blockchain network of the selected crypto asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">toAddress</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination crypto address.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Calculation id used to create withdrawal.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">withdrawalAmount</td>
      <td>string</td>
      <td>Original withdrawal amount.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">commissionAmount</td>
      <td>string</td>
      <td>Commission amount for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">receivedAmount</td>
      <td>string</td>
      <td>Net amount expected after fees/commissions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationDate</td>
      <td>string</td>
      <td>Expiration timestamp in server date-time format, if returned.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="260" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="620">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_ADDRESS</td>
      <td>BUSINESS</td>
      <td>Destination address is invalid for selected network or blocked as internal address.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_AMOUNT</td>
      <td>BUSINESS</td>
      <td>Amount is invalid (including fee greater than withdrawal amount).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 ACTIVE_WITHDRAWAL_REQUEST_FOUND</td>
      <td>BUSINESS</td>
      <td>An uncompleted withdrawal already exists for this client/asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="209" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="571">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">accountType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Account scope. Use WALLET for custodial wallet flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">calculationId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Calculation id returned by withdrawal calculation endpoint.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">transactionId</td>
      <td>string</td>
      <td>Created crypto withdrawal transaction identifier used for tracking status and support.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="260" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="620">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_CALCULATION</td>
      <td>BUSINESS</td>
      <td>calculationId is not found or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_STATUS</td>
      <td>BUSINESS</td>
      <td>Withdrawal operation status does not allow execution.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 ACTIVE_WITHDRAWAL_REQUEST_FOUND</td>
      <td>BUSINESS</td>
      <td>Another uncompleted withdrawal blocks this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="218" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="562">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatProviderType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Fiat provider code for payment/payout processing.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Payment method token used for fiat-provider operations.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">internalToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Internal payment token used for provider-specific routing when applicable.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.code</td>
      <td>string</td>
      <td>Yes</td>
      <td>Asset code used for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.amount</td>
      <td>number</td>
      <td>Yes</td>
      <td>Operation amount in the selected asset.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string | null</td>
      <td>Calculation identifier when provider creates a reusable calculation. Can be null when the fiat calculation is direct and no follow-up calculation id is required.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">withdrawalAmount</td>
      <td>string</td>
      <td>Amount requested for withdrawal.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">commissionAmount</td>
      <td>string</td>
      <td>Commission amount for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">receivedAmount</td>
      <td>string</td>
      <td>Net amount expected after fees/commissions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationDate</td>
      <td>string | null</td>
      <td>Expiration timestamp in server date-time format, if returned.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="266" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="614">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 BALANCE_OPERATION_PROCESSING_ERROR</td>
      <td>BUSINESS</td>
      <td>Fiat withdrawal calculation cannot be produced by provider/flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_PAYMENT_TOKEN</td>
      <td>BUSINESS</td>
      <td>Payment token is invalid, unavailable, or unsupported.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="218" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="562">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">accountType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Account scope. Use WALLET for custodial wallet flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatProviderType</td>
      <td>string</td>
      <td>Yes</td>
      <td>Fiat provider code for payment/payout processing.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">paymentToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Payment method token used for fiat-provider operations.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">internalToken</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Internal payment token used for provider-specific routing when applicable.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.code</td>
      <td>string</td>
      <td>Yes</td>
      <td>Asset code used for the operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">asset.amount</td>
      <td>number</td>
      <td>Yes</td>
      <td>Operation amount in the selected asset.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">transactionId</td>
      <td>string</td>
      <td>Created fiat withdrawal transaction identifier used for tracking payout status and reconciliation.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="266" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="614">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 BALANCE_OPERATION_PROCESSING_ERROR</td>
      <td>BUSINESS</td>
      <td>Fiat withdrawal cannot be started or provider rejected operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_PAYMENT_TOKEN</td>
      <td>BUSINESS</td>
      <td>Payment token/internal token is invalid or restricted.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="212" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="568">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>number</td>
      <td>Conditional</td>
      <td>Source amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider</td>
      <td>string</td>
      <td>Yes</td>
      <td>Fiat provider used for payment.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.token</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Payment token used for provider payment.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.amount</td>
      <td>number</td>
      <td>Conditional</td>
      <td>Destination amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="248" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="612">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Quote id used to create order.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">rate</td>
      <td>string</td>
      <td>Rate pair for the operation. Display this value to the client as the final pair label.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDate</td>
      <td>string</td>
      <td>Creation timestamp in server date-time format.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationDate</td>
      <td>string</td>
      <td>Expiration timestamp in server date-time format, if returned.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>string</td>
      <td>Source amount used in quote calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount / output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on each operation leg, in the corresponding leg asset currency (input.asset / output.asset).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider</td>
      <td>string | null</td>
      <td>Fiat provider code for source leg when source type is FIAT_PROVIDER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.token</td>
      <td>string | null</td>
      <td>Payment token used by provider source leg, if required by provider flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentType</td>
      <td>string</td>
      <td>Fiat payment type selected by provider configuration.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processingBank</td>
      <td>string</td>
      <td>Processing bank selected for fiat provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.amount</td>
      <td>string</td>
      <td>Destination amount used in quote calculation.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote input is inconsistent or cannot be calculated for provided payment details.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CURRENCY_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>One of input/output assets is unknown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Provided client id is invalid or not linked to merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">429 Too Many Requests</td>
      <td>HTTP</td>
      <td>Rate limit is exceeded for quote creation endpoint.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="251" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="529">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">quoteId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Quote identifier returned by quote creation; required to create an order before quote expiration.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destinationCryptoAddress</td>
      <td>string</td>
      <td>No</td>
      <td>Destination wallet address for crypto-out flows (used when output.type is CRYPTO_TRANSFER).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">bankIdentifier</td>
      <td>string</td>
      <td>No</td>
      <td>Optional bank identifier used by selected fiat provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">returnUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on successful payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on failed payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">additionalTimeout</td>
      <td>boolean</td>
      <td>No</td>
      <td>Extended-timeout flag for slow payment flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">outputPaymentProcessingType</td>
      <td>string</td>
      <td>No</td>
      <td>Optional payment processing type for the output leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="316" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="544">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Order id.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>number</td>
      <td>Human-readable order number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions</td>
      <td>object</td>
      <td>Detailed order/quote calculation breakdown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromAsset</td>
      <td>string</td>
      <td>Source asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toAsset</td>
      <td>string</td>
      <td>Destination asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromGrossAmount</td>
      <td>string</td>
      <td>Source gross amount before source-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromNetAmount</td>
      <td>string</td>
      <td>Source net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromFeeAmount</td>
      <td>string</td>
      <td>Source-side fee amount in calculation conditions, in conditions.fromAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toGrossAmount</td>
      <td>string</td>
      <td>Destination gross amount before destination-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toNetAmount</td>
      <td>string</td>
      <td>Destination net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toFeeAmount</td>
      <td>string</td>
      <td>Destination-side fee amount in calculation conditions, in conditions.toAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.rate</td>
      <td>string</td>
      <td>Rate pair in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">recalculationReason</td>
      <td>string | null</td>
      <td>Recalculation reason when quote/order amounts were adjusted by system logic; null when no recalculation happened.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">status</td>
      <td>string</td>
      <td>Current order lifecycle state. Allowed values: PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failureMessage</td>
      <td>string | null</td>
      <td>Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">completionDate</td>
      <td>string | null</td>
      <td>Order completion timestamp when order is finished; null while order is still active.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDate</td>
      <td>string</td>
      <td>Creation timestamp in server date-time format.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">sessionId</td>
      <td>string | null</td>
      <td>Optional client session identifier bound to this order.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type / output.type</td>
      <td>string</td>
      <td>Operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset / output.asset</td>
      <td>string</td>
      <td>Asset code used for each operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount / output.amount</td>
      <td>string</td>
      <td>Operation amount for each leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.transactionAmount / output.transactionAmount</td>
      <td>string</td>
      <td>Provider/settlement amount for operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount / output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on each operation leg, in the corresponding leg asset currency (input.asset / output.asset).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.status / output.status</td>
      <td>string</td>
      <td>Leg status. Allowed values: NEW, PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.failureMessage / output.failureMessage</td>
      <td>string | null</td>
      <td>Failure reason for a specific operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.expirationDate / output.expirationDate</td>
      <td>string | null</td>
      <td>Expiration timestamp for operation leg context, if provided.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider / output.provider</td>
      <td>string | null</td>
      <td>Provider code for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentType / output.paymentType</td>
      <td>string | null</td>
      <td>Provider payment type metadata (for example P2P, SBP).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processingBank / output.processingBank</td>
      <td>string | null</td>
      <td>Processing bank metadata for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.clientBank / output.clientBank</td>
      <td>string | null</td>
      <td>Client bank metadata if returned by provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.fromToken / output.fromToken</td>
      <td>string | null</td>
      <td>Source payment token used by provider leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.toToken / output.toToken</td>
      <td>string | null</td>
      <td>Destination payment token used by provider leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.link / output.link</td>
      <td>string | null</td>
      <td>Provider payment URL for redirect/confirmation flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processorTransactionId / output.processorTransactionId</td>
      <td>string | null</td>
      <td>External provider transaction id for reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processorTransactionNumber / output.processorTransactionNumber</td>
      <td>string | null</td>
      <td>External provider transaction number/reference shown by provider systems for support and reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.post / output.post</td>
      <td>string | null</td>
      <td>Additional provider payload or form-POST metadata when present.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentSystem / output.paymentSystem</td>
      <td>string | null</td>
      <td>Payment system metadata returned by provider integration.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 QUOTE_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Quote id is missing, expired, or unknown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote exists but cannot be used for order creation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_CLIENT_STATUS</td>
      <td>BUSINESS</td>
      <td>Client status/checks do not allow order creation (for example testing not completed).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no access to quote/client used by this order.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="215" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="565">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>number</td>
      <td>Conditional</td>
      <td>Source amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.provider</td>
      <td>string</td>
      <td>Yes</td>
      <td>Fiat provider.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.token</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Payment token for receiving fiat.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="222" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="638">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Quote id used for order creation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">rate</td>
      <td>string</td>
      <td>Rate pair for the operation. Display this value to the client as the final pair label.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDate</td>
      <td>string</td>
      <td>Creation timestamp in server date-time format.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationDate</td>
      <td>string</td>
      <td>Expiration timestamp in server date-time format, if returned.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>string</td>
      <td>Source amount used in quote calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount</td>
      <td>string</td>
      <td>Fee amount on source leg, in input.asset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.amount</td>
      <td>string</td>
      <td>Destination amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.feeAmount</td>
      <td>string</td>
      <td>Fiat provider/exchange fee amount, in output.asset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.provider</td>
      <td>string | null</td>
      <td>Fiat provider code for destination leg when destination type is FIAT_PROVIDER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.token</td>
      <td>string | null</td>
      <td>Payment token used by provider destination leg, if required by provider flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.paymentType</td>
      <td>string</td>
      <td>Fiat payment type.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.processingBank</td>
      <td>string</td>
      <td>Processing bank selected by provider route.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote input is inconsistent or cannot be calculated.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CURRENCY_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Input/output asset is unknown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Client id is invalid or not linked to merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="251" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="529">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">quoteId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Quote identifier returned by quote creation; required to create an order before quote expiration.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destinationCryptoAddress</td>
      <td>string</td>
      <td>No</td>
      <td>Destination wallet address for crypto-out flows (used when output.type is CRYPTO_TRANSFER).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">bankIdentifier</td>
      <td>string</td>
      <td>No</td>
      <td>Optional bank identifier used by selected fiat provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">returnUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on successful payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on failed payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">additionalTimeout</td>
      <td>boolean</td>
      <td>No</td>
      <td>Extended-timeout flag for slow payment flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">outputPaymentProcessingType</td>
      <td>string</td>
      <td>No</td>
      <td>Optional payment processing type for the output leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="300" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="560">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Order id.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>number</td>
      <td>Human-readable order number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions</td>
      <td>object</td>
      <td>Detailed order/quote calculation breakdown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromAsset</td>
      <td>string</td>
      <td>Source asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toAsset</td>
      <td>string</td>
      <td>Destination asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromGrossAmount</td>
      <td>string</td>
      <td>Source gross amount before source-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromNetAmount</td>
      <td>string</td>
      <td>Source net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromFeeAmount</td>
      <td>string</td>
      <td>Source-side fee amount in calculation conditions, in conditions.fromAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toGrossAmount</td>
      <td>string</td>
      <td>Destination gross amount before destination-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toNetAmount</td>
      <td>string</td>
      <td>Destination net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toFeeAmount</td>
      <td>string</td>
      <td>Destination-side fee amount in calculation conditions, in conditions.toAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.rate</td>
      <td>string</td>
      <td>Rate pair in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">status</td>
      <td>string</td>
      <td>Current order lifecycle state. Allowed values: PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failureMessage</td>
      <td>string | null</td>
      <td>Human-readable reason of failure when order cannot be completed; use it for support/debugging, not as a stable business code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type / output.type</td>
      <td>string</td>
      <td>Operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset / output.asset</td>
      <td>string</td>
      <td>Asset code used for each operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount / output.amount</td>
      <td>string</td>
      <td>Operation amount for each leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.transactionAmount / output.transactionAmount</td>
      <td>string</td>
      <td>Provider/settlement amount for operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount / output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on each operation leg, in the corresponding leg asset currency (input.asset / output.asset).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.status / output.status</td>
      <td>string</td>
      <td>Leg status. Allowed values: NEW, PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.failureMessage / output.failureMessage</td>
      <td>string | null</td>
      <td>Failure reason for a specific operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.expirationDate / output.expirationDate</td>
      <td>string | null</td>
      <td>Expiration timestamp for operation leg context, if provided.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider / output.provider</td>
      <td>string | null</td>
      <td>Provider code for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentType / output.paymentType</td>
      <td>string | null</td>
      <td>Provider payment type metadata (for example P2P, SBP).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processingBank / output.processingBank</td>
      <td>string | null</td>
      <td>Processing bank metadata for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.clientBank / output.clientBank</td>
      <td>string | null</td>
      <td>Client bank metadata if returned by provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.fromToken / output.fromToken</td>
      <td>string | null</td>
      <td>Source payment token used by provider leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.toToken / output.toToken</td>
      <td>string | null</td>
      <td>Destination payment token used by provider leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.link / output.link</td>
      <td>string | null</td>
      <td>Provider payment URL for redirect/confirmation flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processorTransactionId / output.processorTransactionId</td>
      <td>string | null</td>
      <td>External provider transaction id for reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.post / output.post</td>
      <td>string | null</td>
      <td>Additional provider payload or form-POST metadata when present.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentSystem / output.paymentSystem</td>
      <td>string | null</td>
      <td>Payment system metadata returned by provider integration.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 QUOTE_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Quote id is missing, expired, or unknown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote exists but cannot be used for sell order creation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INSUFFICIENT_BALANCE</td>
      <td>BUSINESS</td>
      <td>Client internal balance is not enough for requested sell operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no access to quote/client used by this order.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="248" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="532">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">page</td>
      <td>number</td>
      <td>No</td>
      <td>Query parameter with page index. Default example uses 0.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">size</td>
      <td>number</td>
      <td>No</td>
      <td>Query parameter with page size. Default example uses 20.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">sort</td>
      <td>string</td>
      <td>No</td>
      <td>Query parameter for sorting, for example creationDate,desc.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Required when externalClientId is not provided. Used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">externalClientId</td>
      <td>string</td>
      <td>Conditional</td>
      <td>Required when clientId is not provided. External client identifier.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">numbers</td>
      <td>array of number</td>
      <td>No</td>
      <td>Filter by human-readable order numbers.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">orderIds</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by order ids (UUID).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">sessionIds</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by SDK session ids (UUID).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">statuses</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by order status. Allowed values: PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDateFrame</td>
      <td>object</td>
      <td>No</td>
      <td>Creation date range filter. Object { "from": "ISO-8601", "to": "ISO-8601" }.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">completionDateFrame</td>
      <td>object</td>
      <td>No</td>
      <td>Completion date range filter. Object { "from": "ISO-8601", "to": "ISO-8601" }.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">inputAssets</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by source asset codes.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">outputAssets</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by destination asset codes.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">assets</td>
      <td>array of string</td>
      <td>No</td>
      <td>Asset filter applied to either source or destination leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">inputAmount</td>
      <td>object</td>
      <td>No</td>
      <td>Source amount range filter { "from": number, "to": number }.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">outputAmount</td>
      <td>object</td>
      <td>No</td>
      <td>Destination amount range filter { "from": number, "to": number }.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destinations</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by flow destination.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fiatTransactionProviders</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by fiat provider id (for example ASSIST).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoTransactionAddresses</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by crypto destination addresses.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">cryptoTransactionHashes</td>
      <td>array of string</td>
      <td>No</td>
      <td>Filter by crypto blockchain transaction hashes.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="300" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="560">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">content</td>
      <td>array of objects</td>
      <td>Page content with order objects.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Order id.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>number</td>
      <td>Human-readable order number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions</td>
      <td>object</td>
      <td>Detailed order/quote calculation breakdown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromAsset</td>
      <td>string</td>
      <td>Source asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toAsset</td>
      <td>string</td>
      <td>Destination asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromGrossAmount</td>
      <td>string</td>
      <td>Source gross amount before source-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromNetAmount</td>
      <td>string</td>
      <td>Source net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromFeeAmount</td>
      <td>string</td>
      <td>Source-side fee amount in calculation conditions, in conditions.fromAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toGrossAmount</td>
      <td>string</td>
      <td>Destination gross amount before destination-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toNetAmount</td>
      <td>string</td>
      <td>Destination net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toFeeAmount</td>
      <td>string</td>
      <td>Destination-side fee amount in calculation conditions, in conditions.toAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.rate</td>
      <td>string</td>
      <td>Rate pair in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">status</td>
      <td>string</td>
      <td>Current order lifecycle state. Allowed values: PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failureMessage</td>
      <td>string | null</td>
      <td>Human-readable reason of failure for historical orders; useful for support and merchant-side audit.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type / output.type</td>
      <td>string</td>
      <td>Operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset / output.asset</td>
      <td>string</td>
      <td>Asset code used for each operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount / output.amount</td>
      <td>string</td>
      <td>Operation amount for each leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.transactionAmount / output.transactionAmount</td>
      <td>string</td>
      <td>Provider/settlement amount for operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount / output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on each operation leg, in the corresponding leg asset currency (input.asset / output.asset).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.status / output.status</td>
      <td>string</td>
      <td>Leg status. Allowed values: NEW, PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.failureMessage / output.failureMessage</td>
      <td>string | null</td>
      <td>Failure reason for a specific operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider / output.provider</td>
      <td>string | null</td>
      <td>Provider code for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentType / output.paymentType</td>
      <td>string | null</td>
      <td>Provider payment type metadata (for example P2P, SBP).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processingBank / output.processingBank</td>
      <td>string | null</td>
      <td>Processing bank metadata for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.link / output.link</td>
      <td>string | null</td>
      <td>Provider payment URL for redirect/confirmation flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processorTransactionId / output.processorTransactionId</td>
      <td>string | null</td>
      <td>External provider transaction id for reconciliation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">totalElements</td>
      <td>number</td>
      <td>Total number of matching orders.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">totalPages</td>
      <td>number</td>
      <td>Total number of pages.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>number</td>
      <td>Current page number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">size</td>
      <td>number</td>
      <td>Current page size.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">first</td>
      <td>boolean</td>
      <td>true when current page is first page.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">last</td>
      <td>boolean</td>
      <td>true when current page is last page.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">numberOfElements</td>
      <td>number</td>
      <td>Number of items on current page.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">empty</td>
      <td>boolean</td>
      <td>true when content array is empty.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 Bad Request</td>
      <td>HTTP</td>
      <td>Filter body is invalid or missing required client filter (Client id is required).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CLIENT_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Provided client id/external client id is invalid for this merchant.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">429 Too Many Requests</td>
      <td>HTTP</td>
      <td>Rate limit is exceeded for history endpoint.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="239" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="541">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fromAsset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fromPaymentDetails.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source payment type. For conversion use INTERNAL_BALANCE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">toAsset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination asset.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">toPaymentDetails.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination payment type. For conversion use INTERNAL_BALANCE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fromMinAmount</td>
      <td>string</td>
      <td>Minimum allowed source amount.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">fromMaxAmount</td>
      <td>string</td>
      <td>Maximum allowed source amount.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">toMinAmount</td>
      <td>string</td>
      <td>Minimum allowed destination amount.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">toMaxAmount</td>
      <td>string</td>
      <td>Maximum allowed destination amount.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_MERCHANT_ID</td>
      <td>BUSINESS</td>
      <td>No merchant identifier could be resolved for limit calculation context.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Limit request contains invalid pair/payment details for calculation context.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">429 Too Many Requests</td>
      <td>HTTP</td>
      <td>Rate limit is exceeded for limit endpoint.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="209" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="571">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>number</td>
      <td>Conditional</td>
      <td>Source amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Yes</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.amount</td>
      <td>number</td>
      <td>Conditional</td>
      <td>Destination amount for quote/order calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="220" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Quote id used for conversion order.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">rate</td>
      <td>string</td>
      <td>Rate pair for the operation. Display this value to the client as the final pair label.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">creationDate</td>
      <td>string</td>
      <td>Creation timestamp in server date-time format.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">expirationDate</td>
      <td>string</td>
      <td>Expiration timestamp in server date-time format, if returned.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type</td>
      <td>string</td>
      <td>Source operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset</td>
      <td>string</td>
      <td>Source asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount</td>
      <td>string</td>
      <td>Source amount used in quote calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount</td>
      <td>string</td>
      <td>Fee amount on source leg, in input.asset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.type</td>
      <td>string</td>
      <td>Destination operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.asset</td>
      <td>string</td>
      <td>Destination asset code.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.amount</td>
      <td>string</td>
      <td>Destination amount used in quote calculation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on destination leg, in output.asset currency.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote input is inconsistent or cannot be calculated.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 CURRENCY_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Asset id is unknown for conversion pair.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_CLIENT_STATUS</td>
      <td>BUSINESS</td>
      <td>Client status/checks do not allow conversion quote creation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no permission for this operation.</td>
    </tr>
  </tbody>
</table>


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

<table width="100%">
  <thead>
    <tr>
      <th width="197" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="583">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">x-api-key</td>
      <td>string</td>
      <td>Yes</td>
      <td>Authenticates the merchant server-to-server request. Use the API key issued for the merchant and target environment.</td>
    </tr>
  </tbody>
</table>

### Request

<table width="100%">
  <thead>
    <tr>
      <th width="251" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Type</th>
      <th width="100">Required</th>
      <th width="529">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">quoteId</td>
      <td>string</td>
      <td>Yes</td>
      <td>Quote identifier returned by quote creation; required to create an order before quote expiration.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destinationCryptoAddress</td>
      <td>string</td>
      <td>No</td>
      <td>Destination wallet address for crypto-out flows (used when output.type is CRYPTO_TRANSFER).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">comment</td>
      <td>string</td>
      <td>No</td>
      <td>Optional. Used only for the TON network as a transfer memo for the recipient. For other networks the value is ignored. Format: free-form string provided by the receiving party (exchange/wallet).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">bankIdentifier</td>
      <td>string</td>
      <td>No</td>
      <td>Optional bank identifier used by selected fiat provider route.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">returnUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on successful payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failUrl</td>
      <td>string</td>
      <td>No</td>
      <td>URL the client should be redirected to on failed payment flow.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">additionalTimeout</td>
      <td>boolean</td>
      <td>No</td>
      <td>Extended-timeout flag for slow payment flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">outputPaymentProcessingType</td>
      <td>string</td>
      <td>No</td>
      <td>Optional payment processing type for the output leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">destination</td>
      <td>string</td>
      <td>No</td>
      <td>Optional flow destination filter. Recommended value: EXCHANGE.</td>
    </tr>
  </tbody>
</table>

### Response

<table width="100%">
  <thead>
    <tr>
      <th width="300" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="140">Type</th>
      <th width="560">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">id</td>
      <td>string</td>
      <td>Conversion order id.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">number</td>
      <td>number</td>
      <td>Human-readable order number.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions</td>
      <td>object</td>
      <td>Detailed order/quote calculation breakdown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromAsset</td>
      <td>string</td>
      <td>Source asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toAsset</td>
      <td>string</td>
      <td>Destination asset code in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromGrossAmount</td>
      <td>string</td>
      <td>Source gross amount before source-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromNetAmount</td>
      <td>string</td>
      <td>Source net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.fromFeeAmount</td>
      <td>string</td>
      <td>Source-side fee amount in calculation conditions, in conditions.fromAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toGrossAmount</td>
      <td>string</td>
      <td>Destination gross amount before destination-side fees.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toNetAmount</td>
      <td>string</td>
      <td>Destination net amount in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.toFeeAmount</td>
      <td>string</td>
      <td>Destination-side fee amount in calculation conditions, in conditions.toAsset currency.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.rate</td>
      <td>string</td>
      <td>Rate pair in calculation conditions.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.systemRateValue</td>
      <td>string</td>
      <td>Base system rate at the moment of quote calculation. Used as a reference value.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.exchangeRateValue</td>
      <td>string</td>
      <td>Rate used by the exchange engine to calculate the quote.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">conditions.actualRateValue</td>
      <td>string</td>
      <td>Final client-facing rate applied to the quote/order. Show this value to the client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">clientId</td>
      <td>string</td>
      <td>Client identifier used to scope the request to a specific client.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">status</td>
      <td>string</td>
      <td>Current order lifecycle state. Allowed values: PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">failureMessage</td>
      <td>string | null</td>
      <td>Human-readable reason of failure when conversion cannot be completed; use for support/debugging.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input</td>
      <td>object</td>
      <td>Source operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">output</td>
      <td>object</td>
      <td>Destination operation details object.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.type / output.type</td>
      <td>string</td>
      <td>Operation channel. Allowed values: INTERNAL_BALANCE, FIAT_PROVIDER, CRYPTO_TRANSFER.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.asset / output.asset</td>
      <td>string</td>
      <td>Asset code used for each operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.amount / output.amount</td>
      <td>string</td>
      <td>Operation amount for each leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.transactionAmount / output.transactionAmount</td>
      <td>string</td>
      <td>Provider/settlement amount for operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.feeAmount / output.feeAmount</td>
      <td>string</td>
      <td>Fee amount on each operation leg, in the corresponding leg asset currency (input.asset / output.asset).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.status / output.status</td>
      <td>string</td>
      <td>Leg status. Allowed values: NEW, PROCESSING, EXPIRED, COMPLETED, FAILED.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.failureMessage / output.failureMessage</td>
      <td>string | null</td>
      <td>Failure reason for a specific operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.expirationDate / output.expirationDate</td>
      <td>string | null</td>
      <td>Expiration timestamp for operation leg context, if provided.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.provider / output.provider</td>
      <td>string | null</td>
      <td>Provider code for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.paymentType / output.paymentType</td>
      <td>string | null</td>
      <td>Provider payment type metadata (for example P2P, SBP).</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processingBank / output.processingBank</td>
      <td>string | null</td>
      <td>Processing bank metadata for fiat-provider operation leg.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.link / output.link</td>
      <td>string | null</td>
      <td>Provider payment URL for redirect/confirmation flows.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">input.processorTransactionId / output.processorTransactionId</td>
      <td>string | null</td>
      <td>External provider transaction id for reconciliation.</td>
    </tr>
  </tbody>
</table>

### Errors

<table width="100%">
  <thead>
    <tr>
      <th width="240" style="word-break: break-word; white-space: normal;">Name</th>
      <th width="120">Code</th>
      <th width="640">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 QUOTE_NOT_FOUND</td>
      <td>BUSINESS</td>
      <td>Quote id is missing, expired, or unknown.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INVALID_QUOTE</td>
      <td>BUSINESS</td>
      <td>Quote exists but cannot be used for conversion order creation.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">400 INSUFFICIENT_BALANCE</td>
      <td>BUSINESS</td>
      <td>Source internal balance is not enough to execute conversion.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">401 Unauthorized</td>
      <td>HTTP</td>
      <td>x-api-key is missing, invalid, or expired.</td>
    </tr>
    <tr>
      <td style="word-break: break-word; white-space: normal;">403 Forbidden</td>
      <td>HTTP</td>
      <td>Merchant has no access to quote/client used by this conversion.</td>
    </tr>
  </tbody>
</table>


