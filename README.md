<center>

# Pomelo Pay Connect API v2.0 
---
Pomelo Pay

<img src="https://avatars3.githubusercontent.com/u/38243760?s=200&v=4" width="150"></img>

Official API Documentation for [Pomelo Pay Connect](https://dashboard.pomelopay.com).
</center>

## API Environments

| Environment| Base URL
|---|---|
| Production| https://api.pomelopay.com/public              

## Authentication

Our system uses a `client_id` an `client_secret` for API authentication.

You can create new third party applications using your merchant portal.
A new `client_id` and `client_secret` (API Key) will be issued immediately. 

If you are performing direct HTTP requests to the API it sufficient to specific the API Key in the header.

| Header| Content
|---|---|
| Authorization | `api_key    ` 

If you are using a SDK or other libraries it is usually required to specify both the application id `(client_id)` and API key `(client_secret)`.


| SDK| Example
|---|---|
| PHP | `$client = new Client('apikey', 'appid');`


## Pagination

For list views the results will be paginated.
The envelop for the results is as follows:

```
{
  "count": 100 // total count of all items,
  "items": [{...},{...}] // array of returned items
}
```

The first page parameter if you do not set it explicitely will be 1.

| queryString| Example
|---|---|
| page | `?page=2`

## Errors

We use both error messages and HTTP status codes. The error message envelop is as follows:

```
{
  "code": "PP-xx-xx" // error code,
  "message": "Transaction not found" // readable message explaining the error
}
```

| HTTP status code| Description
|---|---|
| 400 | Bad request, check input parameters
| 401 | Not authorized
| 404 | Resource not found
| 500 | Internal Server Error

## Transaction object

The transaction object exists of specific fields that are either user-defined are auto-generated by our system.

\* indicates a required field.

| Field | Data type | Description | Example | User-Defined | Limitations |
|---|---|---|---|---|---|
| amount* |int | The amount of the transaction in cents (e.g. 2000 = 20.00 | 2000 | Yes | Minimum is 100 cents in any local currency |
| currency* |string | The amount of the transaction in cents (e.g. 2000 = 20.00 | 2000 | Yes | Limited to the currencies available for the merchant |
| signature* | string | A checksum for the transaction | mMBR0TXWHvuno/8su3SKMg==  | Yes | According to specification |
| deviceId* | string | The client_id of the application | 123456789  | Yes | Must be the app id for the specified API Key |
| appVersion* | string | A custom version string from your application | 123456789  | Yes | Must be the app id for the specified API Key |
| apiVersion* | string | API used for this request | 2.0  | Yes | Only 2.0 is currently supported |
| signMethod* | string | The method used to generate the signature | sha1  | Yes | Must be `sha1` or `md5` |
| localId | string | A local reference by the merchant or third-party system | ID1234 | Yes | - |
| customerReference | string | A reference made available to the customer | Invoice 2020-123 | Yes | - |
| redirectUrl | string | The optional URL to redirect to after completion or failure of the payment | https://myshop.com/order/13245  | Yes | Only applicable in e-commerce transactions |
| webhook | string | A custom webhook specifically for this transaction. Updates will be sent to both this URL and the optional company-wide webhook | https://foo.bar/hookIn/1234 | Yes | must be a valid URL starting with https:// |
| expires | string | Set a validity for the payment in ISO timestring | 2021-01-01T12:12:12.000001Z | Yes | Must be at least 1 hour in the future and in ISO timestring (e.g. new Date().toISOString() and in UTC timezone |
| id | string | The unique ID of the transaction in our system | 5e22e1037ac57f000841efff  | No | - |
| qrcode | object | object containing the URL of the QR Code for this transaction | `{ url: 'https://example.org/image.png'}`  | No | - |
| provider | string | The payment provider for this transaction | card  | Yes | Limited to the available payment providers for this merchant |
| created | string | The creation timestamp in UTC for this transaction |2020-01-02T10:42:14.898Z  | No | - |
| state | string | The status of this transaction | CONFIRMED  | No | Can only be one of the following: QR\_CODE\_GENERATED, CONFIRMED, CANCELLED, REFUND_REQUESTED, REFUNDED |

## API Operations


The following exposed API operations from the Pomelo Pay API are available using the API Client.
See below for more details about each resource.

### 💳 Transactions

#### Create a new transaction with or without a specific payment method.

| Description | Data |
|---|---|
| Method | POST     
| Production URL| https://api.pomelopay.com/public/transactions
| Headers| Authorization: api_key, Accept: application/json, Content-Type: application/json

Body Example:

```
{
  "localId": "123456789",
  "customerReference": "invoice 1",
  "signature": "mMBR0TXWHvuno/8su3SKMg==",
  "amount": 2000,
  "currency": "GBP",
  "provider": "card",
  "appVersion": "vendingSoftware1.0",
  "apiVersion": "2.0",
  "deviceId": "12345789",
  "signMethod": "sha1",
  "webhook": "https://www.foo.bar/hookIn/123",
  "redirectUrl": "https://www.shop.url/order/1234",
}
```

#### Query a specific transaction

| Description | Data |
|---|---|
| Method | GET     
| Production URL| https://api.pomelopay.com/public/transactions/<:id>
| Headers| Authorization: api_key, Accept: application/json, Content-Type: application/json
| Example| GET https://api.pomelopay.com/public/transactions/5e22e1037ac57f000841efff

#### List transactions

| Description | Data |
|---|---|
| Method | GET     
| Production URL| https://api.pomelopay.com/public/transactions
| Headers| Authorization: api_key, Accept: application/json, Content-Type: application/json
| Example| GET https://api.pomelopay.com/public/transactions?page=3

### 🌐 Webhooks

#### Receive and process webhooks

Please see the webhook section for more information

## Verifying data integrity

On sending a transaction you must on application side create a signature with your api key.

On receiving webhooks it is strongly recommended to calculate the signature on your side and compare it with the specified signature to ensure data integrity.

The calculation of the signature is as follows:

| Signature method | Data |
|---|---|
| md5 | md5('amount=2000&currency=GBP&apiKey=mysecretkey').digest('base64')
| sha1| sha1('amount=2000&currency=GBP&apiKey=mysecretkey').digest('hex')

## Webhooks

Our system uses webhooks to receive **real-time notifications** about any transaction state changes. The webhooks are using the `POST` http method.

It is strongly recommended to check the **signature** from the transaction and validate it to ensure the data is coming from an authentic source.

Alternatively you can also specify a custom URL for a single specific transaction. 

> For more information about verifying data integrity you can check the Verifing Data Integrity section. 

### Setting your webhook URL

- Navigate to [https://dashboard.pomelopay.com](https://dashboard.pomelopay.com)
- Login with your merchant account
- From the left-hand side menu select "Connect"
- From the secondary navigation select "Webhooks"
- Enter the URL on which your system will listen for webhooks events
- The specified URL should be a publicly accessible endpoint that accepts `POST` data in `JSON` format

### Type of webhook events
#### Transaction created

```
{
  "localId": "123456789" // the optional local ID from your internal system,
  "customerReference": "invoice 1" // the optional customer reference ,
  "transactionId": "5e22e1037ac57f000841efff" // unique transaction id,
  "state": "QR_CODE_GENERATED" // state of the transction,
  "created": "2020-01-02T10:42:14.898Z" // creation time in UTC,
  "signature": "mMBR0TXWHvuno/8su3SKMg==" // signature of the transaction,
  "amount": 2000 // amount in cents,
  "currency": "GBP" // currency of the transaction,
  "provider": "card" // payment method  ,
  "qrCode": {
    "url": "https://example.org/code.png" // URL location of the QR code
  }
}
```

#### Transaction updated

```
{
  "localId": "123456789" // the optional local ID from your internal system,
  "customerReference": "invoice 1" // the optional customer reference ,
  "transactionId": "5e22e1037ac57f000841efff" // unique transaction id,
  "state": "CONFIRMED" // state of the transction,
  "created": "2020-01-02T10:42:14.898Z" // creation time in UTC,
  "signature": "mMBR0TXWHvuno/8su3SKMg==" // signature of the transaction,
  "amount": 2000 // amount in cents,
  "currency": "GBP" // currency of the transaction,
  "provider": "card" // payment method,
  "qrCode": {
    "url": "https://example.org/code.png" // URL location of the QR code
  }
}
```


## SDK's and Frameworks

If you're looking to use this API in a specific stack or framework you currently have the following options:

| Framework                  | Package                                                        |
| ---------------------------|----------------------------------------------------------------| 
| PHP7                       | [https://github.com/pomelopay/pomelopay-connect-php](https://github.com/pomelopay/pomelopay-connect-php)                     |
| Wordpress                  | [https://wordpress.org/plugins/pomelo-payment-gateway/](https://wordpress.org/plugins/pomelo-payment-gateway/) |
| WooCommerce                | [https://wordpress.org/plugins/pomelo-payment-gateway/](https://wordpress.org/plugins/pomelo-payment-gateway/) |
| Other                      | Build one and get featured here!                                |

## About

Sign up with [Pomelo Pay](https://dashboard.pomelopay.com) and start accepting online and instore payments for your business today!


## Support

If need support with the Pomelo Pay Connect API please open an issue on this repository.

For all other support you can contact us at  📧 [info@pomelopay.com](mailto:info@pomelopay.com)

---
[https://dashboard.pomelopay.com](https://dashboard.pomelopay.com)
