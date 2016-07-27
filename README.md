# SpectroCoin Bitcoin Wallet API specification
========================
This document describes [SpectroCoin](https://spectrocoin.com) wallet service API specification.

# Contents

* [Requirements](#requirements)
* [API](#api)
   * [Introduction](#introduction)
   * [POST /oauth2/auth](#post-oauth2auth)
   * [POST /oauth2/refresh](#post-oauth2refresh)
   * [POST /wallet/send/{currency}](#post-walletsendcurrency)
   * [GET /wallet/exchange/calculate/buy](#get-walletexchangecalculatebuy)
   * [GET /wallet/exchange/calculate/sell](#get-walletexchangecalculatesell)
   * [POST /wallet/exchange/buy](#post-walletexchangebuy)
   * [POST /wallet/exchange/sell](#post-walletexchangesell)
   * [GET /wallet/accounts](#get-walletaccounts)
   * [GET /wallet/account/{accountId}](#get-walletaccountaccountid)
   * [GET /wallet/deposit/BTC/last](#get-walletdepositbtclast)
   * [GET /wallet/deposit/BTC/fresh](#get-walletdepositbtcfresh)
* [Errors](#errors)
  * [Validation error](#validation-error)
  * [Error codes](#error-codes)
* [Example applications](#example-applications)

# Requirements

* Must have a SpectroCoin account ([Sign Up!](https://spectrocoin.com/en/signup.html)) to setup wallet API.
* Must [setup](https://spectrocoin.com/en/walletAPI.html) wallet API instance on SpectroCoin API configuration page.

# API

Wallet API is REST based web service. Accept ``GET`` or ``POST`` (``application/json``) requests. Responses are ``application/json`` content type.
API authentication is performed using OAuth2 (``/oauth2``). Wallet functions is accessible through ``/wallet``.
Root REST API address: ``https://spectrocoin.com/api/r/``

### Responses

Wallet API can return specific response object (HTTP **200**), validation error (HTTP **203**), suspicious activity error (HTTP **403**) or internal error (HTTP **500**).

## Introduction
![](https://github.com/SpectroFinance/SpectroCoin-Wallet-API/blob/master/wallet%20api.jpg)

SpectroCoin use two services:
  * **Authorization service** ``/oauth2`` - necessary for getting access to wallet api private service methods usage.
  * **Wallet api service** ``/wallet`` - allows user to check account balances, spend, exchange currency together with other wallet related activities.

## Authorization service

1. To get access tokens (access and refresh tokens) you must call [**/oath2/auth**](#post-oauth2auth) REST method providing required fields.
2. Response will contain token information (access_token, refresh_token, scope, expires_in..). Having access tokens you can build Authorization header value.
3. All other REST methods within security scope **must have extra HTTP header** - ``Authorization``. Value should start with Bearer space and access token
``Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625``

### Access tokens life time

These tokens have a finite lifetime and you must write code to detect when an access token expires. You can do this either by keeping track of the ``expires_in`` value returned in the response from the token request (the value is expressed in seconds), or handle the error response (``1003``) from the API endpoint when an expired token is detected. If access token is expired need to get new token by using refresh token. If refresh token is expired, then need to authenticate from the beginning.

### Scope
The scope parameter is used to indicate a list of permissions that are requested by the client. List of scopes should be separated with space. Example: ``send_currency currency_exchange user_account``.

## POST /oauth2/auth

Method to authenticate user and get access tokens to use wallet REST API. Refresh token is used to issue new access tokens when original access token expire.

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
client_id	| String	|+	| wallet_8f452af79dcb7fed5979d11cc33910bb
client_secret |	String	| +	| test-secret
version |	String	| + |	1.0
scope |	String	| +	| send_currency currency_exchange user_account

Example HTTP request:

```http
POST https://spectrocoin.com/api/r/oauth2/auth
Connection: Keep-Alive
Content-Length: 137
Content-Type: application/json
Host: spectrocoin.com

{
   "client_id": "wallet_8f452af79dcb7fed5979d11cc33910bb",
   "client_secret": "test-secret",
   "version": "1.0",
   "scope": "user_account send_currency currency_exchange"
}

```
### Response

Possible validation error codes: [1, 1003, 2001](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
access_token  |	String  |	+ |	42e0f8d6cc2f30d...2df5455b6d05308e6e064b99625
expires_in  |	String	| +	| 300 (seconds)
refresh_token	| String  |	+	| 3cbba02103fb479...21923fe35038ccf4ebbc8fe6084
scope	| String	| +	| send_currency currency_exchange user_account
token_type  |	String	| -	| bearer

JSON Response example:
```http
{
   "access_token": "42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625",
   "token_type": "bearer",
   "expires_in": 300,
   "refresh_token": "3cbba02103fb47991921923fe350da11299038ccf4ebbc866fc4b2615b46c013a3211509b0fe6084",
   "scope": "send_currency currency_exchange user_account"
}
```

## POST /oauth2/refresh
Method to exchange user wallet REST API refresh token to new pair of oauth2 tokens.

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
client_id	| String	| +	| wallet_8f452af79dcb7fed5979d11cc33910bb
client_secret	| String  |	+ |	test-secret
refresh_token	| String	| +	| cfee255c08b61b08...1588a4de1fc75a89b529619
version	| String	| +	| 1.0

Example HTTP request:

```http
POST https://spectrocoin.com/api/r/oauth2/refresh
Connection: Keep-Alive
Content-Length: 177
Content-Type: application/json
Host: spectrocoin.com

{
   "client_id": "wallet_8f452af79dcb7fed5979d11cc33910bb",
   "client_secret": "test-secret",
   "version": "1.0",
   "refresh_token": "cfee255c08b61b08…1588a4de1fc75a89b529619"
}
```

### Response

Possible validation error codes: [1, 1003, 2001, 2003](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
access_token	| String	| +	| f6f2e066917...3cd28756350041fd00c8ff741e0c1bf
expires_in	| String	| +	| 300 (seconds)
refresh_token	| String  |	+	| f4a74245757...be20d23fc81cff0750613cd319abaf1
scope	| String  |	+	| send_currency currency_exchange user_account
token_type  |	String	| -	| bearer

JSON Response example:
```http
{
   "access_token": "f6f2e0669172035b98a241d6e2afdf531184a400ec8321e1f3cd28756350041fd00c8ff741e0c1bf",
   "token_type": "bearer",
   "expires_in": 300,
   "refresh_token": "f4a742457578f5d0b029bf39776c8a6b45b43f7bf5e97514cbe20d23fc81cff0750613cd319abaf1",
   "scope": "send_currency currency_exchange user_account"
}
```

Validation example:
```http
[{
   "code": 2003,
   "message": "Refresh token expired, please authenticate."
}]
```

## POST /wallet/send/{currency}
Operation to send currency to receiver (bitcoin address, email address). Additional fees may apply depending on send currency and receiver type. Receiver will receive provided amount and currency if sender has enough balance to cover possible additional fee, otherwise fee will be deducted from send amount. You can provide multiple Bitcoin addresses as receivers to include all payments into one transaction.

### Security

Schema	| Scope
------|-------
OAuth2	|	send_currency

### Path parameters
Field	| Type	| Required  |	Example
------|-------|-----------|--------
currency	| String	| +	| EUR, BTC

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
amount	| Double	| +	| 13.19, 0.00145021
receiver	| String  |	+ |	test_cs7@spectrocoin.com, 12KKCFWLPayT8VAbhHRhs7VCS1LPUGGfqv

Example HTTP request:

```http
POST https://spectrocoin.com/api/r/wallet/send/EUR
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Content-Length: 47
Content-Type: application/json
Host: spectrocoin.com

[
  {
	"amount": 0.2,
	"receiver": "12KKCFWLPayT8VAbhHRhs7VCS1LPUGGfqv"
  }
]
```

Example for multiple BTC receivers:
```http
POST https://spectrocoin.com/api/r/wallet/send/BTC
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Content-Length: 47
Content-Type: application/json
Host: spectrocoin.com

[
  {
	"amount": 0.2,
	"receiver": "12KKCFWLPayT8VAbhHRhs7VCS1LPUGGfqv"
  },
  {
	"amount": 0.3,
	"receiver": "12KKCFWLPayT8VAbhHRhs7VCS1LPUGGfqv"
  }
]
```


### Response

Possible validation error codes: [1, 1001, 1003, 1004, 1005, 1008, 3001, 3002, 3003, 3005, 3006, 3016, 3017 5003, 5021](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
paymentId	| Long	| +	| 153
withdrawAmount	| Double	| +	| 13.19, 0.00145021
receiveAmount	| Double  |	+	| 13.19, 0.00145021
currency	| String  |	+	| EUR, BTC, USD...
status  |	String	| -	| Statuses: `NEW`, `PENDING` or `PAID`

* `NEW` – initial status, should be processed in near future or manually 
* `PENDING` – money from sender account is charged and reserved for receiver
* `PAID` – receiver got money

JSON Response example:
```json
{
   "paymentId": "153",
   "withdrawAmount": 13.19,
   "receiveAmount": 13.19,
   "currency": "EUR",
   "status": "PAID"
}
```

Validation example:
```http
[
      {
      "code": 1,
      "message": "amount is required"
   },
      {
      "code": 1,
      "message": "receiver is required"
   }
]
```

## GET /wallet/exchange/calculate/buy
Operation used to calculate needed pay amount to receive required receive amount at the current moment.

### Security

Schema	| Scope
------|-------
OAuth2	|	currency_exchange

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
receiveAmount	| Double	| +	| 0.00212015, 3.01
receiveCurrency	| String  |	+ |	BTC
payCurrency | String  |	+	|	EUR

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/exchange/calculate/buy?receiveAmount=3.01&receiveCurrency=USD&payCurrency=EUR
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Host: spectrocoin.com
```

### Response

Possible validation error codes: [1, 1003, 1004, 1005, 7008](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
payAmount	| Double	| +	| 2.35
payCurrency	| String  |	+	| EUR

JSON Response example:
```http
{
   "payAmount": 2.35,
   "payCurrency": "EUR"
}
```

## GET /wallet/exchange/calculate/sell
Operation used to calculate receive amount of currency exchange using pay amount at the current moment.

### Security

Schema	| Scope
------|-------
OAuth2	|	currency_exchange

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
payAmount	| Double	| +	| 10.15, 14.80219015
receiveCurrency	| String  |	+ |	BTC
payCurrency |	String	|	+	|	EUR

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/exchange/calculate/sell?payAmount=10.15&payCurrency=EUR&receiveCurrency=BTC
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Host: spectrocoin.com
```

### Response

Possible validation error codes: [1, 1003, 1004, 1005, 7008](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
receiveAmount	| Double	| +	| 19.23, 0.0001257
receiveCurrency	| String  |	+	| EUR

JSON Response example:
```http
{
   "receiveAmount": 0.04610103,
   "receiveCurrency": "BTC"
}
```

## POST /wallet/exchange/buy
Operation used to buy requested amount of receive currency while paying with pay currency.

### Security

Schema	| Scope
------|-------
OAuth2	|	currency_exchange

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
receiveAmount	| Double	| +	| 3.01, 0.04610103
receiveCurrency	| String  |	+ |	USD
payCurrency |	String	|	+	|	EUR

Example HTTP request:

```http
POST https://spectrocoin.com/api/r/wallet/exchange/buy
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Content-Length: 54
Content-Type: application/json
Host: spectrocoin.com

{
   "receiveAmount": 3.01,
   "receiveCurrency": "USD",
   "payCurrency": "EUR"
}

```

### Response

Possible validation error codes: [1, 1001, 1002, 1003, 1004, 1005, 7006, 7007, 7008, 7009, 7010](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
exchangeId	|	Long	|	+	|	50
payAmount	| Double	| +	| 2.35, 0.046101
payCurrency	| String  |	+	| EUR
receiveAmount	| Double	| +	| 19.23, 0.0001257
receiveCurrency	| String  |	+	| USD
status  |	String	| -	| Statuses: NEW, PENDING or PAID

* `NEW` – initial status
* `PENDING` – waiting for approval
* `PAID` – exchange completed

JSON Response example:
```http
{
   "exchangeId": 50,
   "status": "PAID",
   "payAmount": 2.35,
   "payCurrency": "EUR",
   "receiveCurrency": "USD",
   "receiveAmount": 3.01
}
```

## POST /wallet/exchange/sell
Operation used to exchange pay currency amount to receive currency.

### Security

Schema	| Scope
------|-------
OAuth2	|	currency_exchange

### Request

Field	| Type	| Required  |	Example
------|-------|-----------|--------
payAmount	| Double	| +	| 1.15, 2.35040101
receiveCurrency	| String  |	+ |	EUR
payCurrency |	String	|	+	|	BTC

Example HTTP request:

```http
POST https://spectrocoin.com/api/r/wallet/exchange/sell
Authorization: Bearer 42e0f8d6cc2f30de2b1dad7cc2df5455b6d05308e6e06495c766dcb43853ba6d17b77b7623899625
Connection: Keep-Alive
Content-Length: 50
Content-Type: application/json
Host: spectrocoin.com

{
   "payAmount": 1.15,
   "receiveCurrency": "BTC",
   "payCurrency": "EUR"
}

```

### Response

Possible validation error codes: [1, 1001, 1002, 1003, 1004, 1005, 7006, 7007, 7008, 7009, 7010](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
exchangeId	|	Long	|	+	|	51
payAmount	| Double	| +	|	1.15
payCurrency	| String  |	+	| EUR
receiveAmount	| Double	| +	| 0.00522326
receiveCurrency	| String  |	+	| BTC
status  |	String	| +	| Statuses: `NEW`, `PENDING` or `PAID`

* `NEW` – initial status
* `PENDING` – waiting for approval
* `PAID` – exchange completed

JSON Response example:
```http
{
   "exchangeId": 51,
   "status": "PAID",
   "payAmount": 1.15,
   "payCurrency": "EUR",
   "receiveCurrency": "BTC",
   "receiveAmount": 0.00522326
}
```

## GET /wallet/accounts
Operation used to get user accounts information.

### Security

Schema	| Scope
------|-------
OAuth2	|	user_account

### Request

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/accounts
Authorization: Bearer f6f2e0669172035b98a241d6e2afdf531184a400ec8321e1f3cd28756350041fd00c8ff741e0c1bf
Connection: Keep-Alive
Host: spectrocoin.com
```

### Response
Response is an array of accounts. Account structure is defined bellow.

Possible validation error codes: [1, 1003, 1004](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
accountId	|	Long	|	+	|	2626
balance	| Double	| +	|	351.76
availableBalance	| Double	| +	|	351.76
reservedAmount	| Double	| -	|	0
currencyName	| String  |	+	| Euro
currencyCode	| String  |	+	| EUR
currencySymbol  |	String	| -	| €

JSON Response example:
```http
{"accounts": [
   {
      "accountId": 1505,
      "balance": 1677.11,
      "currencyName": "EURO",
      "currencyCode": "EUR",
      "availableBalance": 1677.11,
      "reservedAmount": 0
   },
    {
      "accountId": 2625,
      "balance": 104.49,
      "currencyName": "GBP",
      "currencyCode": "GBP",
      "availableBalance": 104.49,
      "reservedAmount": 0
   },
    {
      "accountId": 2503,
      "balance": 25.76161379,
      "currencyName": "Bitcoin",
      "currencyCode": "BTC",
      "availableBalance": 17.76161379,
      "reservedAmount": 18
   },
    {
      "accountId": 2583,
      "balance": 627.51,
      "currencyName": "USD",
      "currencyCode": "USD",
      "availableBalance": 627.51,
      "reservedAmount": 0
   }
]}
```

## GET /wallet/account/{accountId}
Operation used to get paged transaction history of user account.

### Security

Schema	| Scope|
------|-------|
OAuth2	|	user_account|

### Path parameters
Field	| Type  |	Always return	| Example
------|-------|---------------|--------
accountId	|	Long	|	+	|	2626

### Request
Field	| Type  |	Always return	| Example
------|-------|---------------|--------
page	|	Integer	|	+	|	1
size	|	Integer	|	+	|	4

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/account/1505?page=1&size=4
Authorization: Bearer f6f2e0669172035b98a241d6e2afdf531184a400ec8321e1f3cd28756350041fd00c8ff741e0c1bf
Connection: Keep-Alive
Host: spectrocoin.com
```

### Response
Response is an array of transaction info and total transactions count. Historical data is returned in descending order. If user don’t define page and size in request by default we return first 25 records. History structure is defined bellow.

Possible validation error codes: [1, 1003, 1004, 6001](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
totalCount	|	Long	|	+	|	425
history	| Array of **History structure**	| - | |

#### History structure

|Field	| Type  |	Always return	| Example|
|------|-------|---------------|--------|
|transactionId	|	String	|	+	|	SC000004848|
|amount	|	Double	|	+	|	-1.15|
|description	|	String	|	+	|	EUR sell for exchange. 1.15 EUR|
|date	|	Date	|	+	|	2016-03-10T14:27:31.000Z|
|status  |	String	| +	| Statuses: `PENDING`, `PROCESSED`  or `FAILED`|

* `PENDING`  – waiting for action
* `PROCESSED` – transaction completed
* `FAILED` – transaction incomplete for some reason

JSON Response example:
```http
{
   "totalCount": 425,
   "history":    
   [
      {
         "transactionNo": "SC000004848",
         "status": "PROCESSED",
         "amount": -1.15,
         "description": "EUR sell for exchange. 1.15 EUR",
         "date": "2016-03-10T14:27:31.000Z"
      },
      {
         "transactionNo": "SC000004846",
         "status": "PROCESSED",
         "amount": -2.35,
         "description": "EUR sell for exchange. 2.35 EUR",
         "date": "2016-03-10T14:18:12.000Z"
      },
      {
         "transactionNo": "SC000004841",
         "status": "PROCESSED",
         "amount": -13.19,
         "description": "EUR sell for exchange. 13.19 EUR",
         "date": "2016-03-10T14:12:40.000Z"
      },
      {
         "transactionNo": "SC000004776",
         "status": "PROCESSED",
         "amount": -1.01,
         "description": "EUR sell for exchange. 1.01 EUR",
         "date": "2016-03-09T14:39:13.000Z"
      }
   ]}
```
## GET /wallet/deposit/BTC/last
Operation used to get last BTC deposit address.

### Security

Schema	| Scope|
------|-------|
OAuth2	|	user_account|

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/deposit/BTC/last
Authorization: Bearer f6f2e0669172035b98a241d6e2afdf531184a400ec8321e1f3cd28756350041fd00c8ff741e0c1bf
Connection: Keep-Alive
Host: spectrocoin.com
```
### Response
Return last Bitcoin address.

Possible validation error codes: [1, 1003, 1004](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
btcAddress	|	String	|	+	|	1M4bqMd471TTwNtUSeHPhSW5qQy1Y48p5b

JSON Response example:
```http
{
	"btcAddress": "1M4bqMd471TTwNtUSeHPhSW5qQy1Y48p5b"
}
```

## GET /wallet/deposit/BTC/fresh
Operation used to get new BTC deposit address.

### Security

Schema	| Scope|
------|-------|
OAuth2	|	user_account|

Example HTTP request:

```http
GET https://spectrocoin.com/api/r/wallet/deposit/BTC/fresh
Authorization: Bearer f6f2e0669172035b98a241d6e2afdf531184a400ec8321e1f3cd28756350041fd00c8ff741e0c1bf
Connection: Keep-Alive
Host: spectrocoin.com
```
### Response
Return new Bitcoin address.

Possible validation error codes: [1, 1003, 1004](#error-codes)

Field	| Type  |	Always return	| Example
------|-------|---------------|--------
btcAddress	|	String	|	+	|	1Bm3ENaZrtrExi56ywM6DWKQgJDmsrMQ41

JSON Response example:
```http
{
	"btcAddress": "1Bm3ENaZrtrExi56ywM6DWKQgJDmsrMQ41"
}
```

## Errors
Invalid request or internal errors will result HTTP response with code **500**.

### Validation error
Validation errors will result HTTP response with code **203**. Response is an array of error information:

Field	| Type	| Required  |	Example
------|-------|-----------|--------
code	| Short	| +	| 1
message	| String	| +	| Unable to sell Bitcoin

JSON Response example:
```http
[
	{
	  "code":1005,
	  "message":"Unknown value: BTR"
	}
]
```

### Error codes
Current available error codes:

Code | Message
------|-------
1	|	 Invalid validation (dynamic message)
97	|	 Unsupported media type
100	|	 Unexpected error
1001	|  {value} can't be null
1002	|	 {amount} should be more than zero
1003	|	 Forbidden
1004	|	 Unauthorized
1005	|	 Unknown value: ???
1008	|	 Amount should be more than {amount} {currency}
2001	|	 Specified client not found for this credentials. client_id: {client_id}, version: {version}
2002	|	 Application for this user is disabled
2003	|	 Refresh token expired, please authenticate
3001	|	 Balance not enough to send
3002	|	 Invalid email or address
3003	|	 Invalid email address
3006	|	 Amount too small to send
3016	|  Destination count reached. Max allowed destinations: {count}
3017	|  Data have empty fields
5003	|	 User not verified
5021	|  Unsupported multiple coins send to email address
6001	|	 Member account {accountId} not found for this user.
7006	|	 Exceeds {period} user pay limit! - {amount} {currency}
7007	|	 Exceeds {period} user receive limit! - {amount} {currency}
7008	|	 Unsupported currency exchange.
7009	|	 Pay amount can't be smaller than {amount} {currency}
7010	|	 Receive amount can't be smaller than {amount} {currency}


# Example applications

There are several sample SpectroCoin wallet API client applications. You should customize them for your needs.

## Java

Sample Java application **coming soon**.

## PHP

Sample PHP application **coming soon**.
