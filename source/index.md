---
title: ProxyPay API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='http://www.proxypay.co.ao'>ProxyPay</a> is crafted with &#9825; by <a href='http://www.timeboxed.co.ao'>Timeboxed</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the ProxyPay API! This API allows developers to easily integrate with Multicaixa to accept payments. You have endpoints to generate numeric references for payments and
to receive notification of payments when they occur.

The ProxyPay API is organized around <a href="http://en.wikipedia.org/wiki/Representational_State_Transfer">REST</a>. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which can be understood by off-the-shelf HTTP clients. <a href="http://www.json.org">JSON</a> will be returned in all responses from the API, including errors.

# Authentication

You authenticate to the ProxyPay API by providing your API key in the request. Be sure to keep it a secret!

Authentication to the API occurs via HTTP Basic Auth. Provide `api` as the basic auth username and your API key as the password.

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. You must authenticate for all requests.

> To authorize, use this code:

```shell
curl "https://api.proxypay.co.ao" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
```

> Make sure to replace `reh8inj33o3algd2tpi6tkcnrqf8rjj2` with your own API key.

<aside class="notice">
You must replace `reh8inj33o3algd2tpi6tkcnrqf8rjj2` in the example with your own API key.
</aside>

# Versioning

By default, all requests receive the latest version of the API, which is currently `v1`. We encourage you to explicitly request this version via the `Accept` header.

```shell
curl "https://api.proxypay.co.ao" \
  -H 'Accept: application/vnd.proxypay.v1+json'
```

# References

A reference must be created whenever you want to receive a payment of a given amount.

To create one you must specify at least an `amount` and an `expiry date`. Additionaly you can specify any number of `custom_fields`.

A reference will have the following attributes (all attributes, except `custom_fields`, have  JSON strings as values):

Attribute | Description
--------- | -----------
id | A unique identifier assigned by the server.
entity_id | A numeric string of length 5, that identifies your company in the Multicaixa system.
number | A numeric string of length 9, assigned by the server that identifies the reference in the Multicaixa system.
expiry_date | A date in the format `YYYY-MM-DD` (ISO 8601) and `WAT` timezone. The reference will be considered valid for payment until the end of the specified day.
amount | The amount to be paid, specified with units and *exactly two decimal places*, with a `.` as separator. The amount can be a number up to `999999999.99`. All amounts are in `AOA` (Angolan Kwanza).
status | This is a read-only attribute and will be set to `active`, `paid`, `expired` or `deleted`.
custom_fields | This is a `map` with an arbitrary number of keys. All values must be strings. This can hold any information useful to reconcile the payment notification, because the payment event will inherit all the custom_field values. Examples include the invoice number, PO number, customer ID, etc..

## Get All References

```shell
curl "https://api.proxypay.co.ao/references" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
```

> The above command returns JSON structured like this:

```json
{
  "references": [
    {
      "status": "active",
      "number": "664654696",
      "id": "8uW7O0N4LvOuQsYAFc",
      "expiry_date": "2015-11-05",
      "custom_fields": {
        "invoice": "2015/0121"
      },
      "amount": "5000.00"
    },
    {
      "status": "deleted",
      "number": "234123215",
      "id": "8uVlQn87qCPnF9U3Um",
      "expiry_date": "2015-02-15",
      "custom_fields": {
        "invoice": "2015/0145"
      },
      "amount": "12222.00"
    },
    {
      "status": "paid",
      "number": "283749832",
      "id": "8uVigNJ7Jj4hvVMdhQ",
      "expiry_date": "2015-02-28",
      "custom_fields": {
        "invoice": "2015/0118"
      },
      "amount": "5000.00"
    },
    {
      "status": "expired",
      "number": "293840932",
      "id": "8uUIvA8jlPKnSwwKPI",
      "expiry_date": "2014-12-12",
      "custom_fields": {
        "invoice": "2014/0097"
      },
      "amount": "11123.23"
    }
  ],
  "meta": {
    "total_count": 4,
    "offset": 0,
    "limit": 20
  }
}
```

This endpoint retrieves all references in the system, sorted by descending order of creation date.

### HTTP Request

`GET https://api.proxypay.co.ao/references`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit | 20 | Max. number of references to return.
offset | 0 | Skip `offset` records. Usefull for pagination.
status | - | Filter parameter. If set to either `active`, `deleted`, `expired` or `paid` will return only references with that status value.
q | - | Filter parameter. will return any reference for which the value of the `q` parameter matches the beginning of the reference `number` or any of the `custom_fields`.

## Get a Specific Reference

```shell
curl "https://api.proxypay.co.ao/references/8uVigNJ7Jj4hvVMdhQ" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
```

> The above command returns JSON structured like this:

```json
{
  "reference": {
    "updated_at": "2015-02-20T17:30:00Z",
    "status": "paid",
    "number": "283749832",
    "id": "8uVigNJ7Jj4hvVMdhQ",
    "expiry_date": "2015-02-28",
    "entity_id": "99999",
    "custom_fields": {
      "invoice": "2014/0097"
    },
    "created_at": "2015-02-01T08:36:37Z",
    "amount": "5000.00"
  }
}
```

This endpoint retrieves a specific reference.

### HTTP Request

`GET https://api.proxypay.co.ao/references/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the reference to retrieve


## Generate a New Reference

```shell
curl -XPOST "https://api.proxypay.co.ao/references" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2" \
  -H 'Content-Type: application/json' \
  -d '{"reference": { "amount": "25000.00", "expiry_date": "2015-05-15", "custom_fields": {"invoice": "2015/0399"}}}'
```

> The above command returns JSON structured like this:

```json
{
  "reference": {
    "updated_at": "2015-05-10T17:43:10Z",
    "status": "active",
    "number": "892374833",
    "id": "8uWqsbFsHNUzS19DY8",
    "expiry_date": "2015-05-15",
    "entity_id": "99999",
    "custom_fields": {
      "invoice": "2015/0399"
    },
    "created_at": "2015-05-10T17:43:10Z",
    "amount": "25000.00"
  }
}
```

This endpoint generates a new payment reference for a given amount.

### HTTP Request

`POST https://api.proxypay.co.ao/references`

### URL Parameters

Parameter | Description
--------- | -----------


# Payment Events - Pull API

This endpoint can be used to fetch any new payment events that occured in the system. It works conceptually as a queue, and all payments
are returned in the order they ocurred.

You can retrieve up to 100 payments in a single call. All payments returned will be reserved for 120 seconds, meaning that they are not returned in a subsequent call to the endpoint. If an acknowledgement indicating that they have been sucessfully processed is not sent within that timeframe, they will become available again.

A Payment event will have the following attributes (all attributes, except `custom_fields`, have JSON strings as values):

Attribute | Description
--------- | -----------
id | A unique identifier of the payment event.
entity_id | A numeric string of length 5, that identifies your company in the Multicaixa system.
reference_number | A numeric string of length 9, that identifies the reference originating the payment in the Multicaixa system.
reference_id | The unique identifier of the reference that originated the payment
datetime | Date and time in UTC (ISO 8601) when the payment occurred.
amount | The amount to be paid, specified with units and *exactly two decimal places*, with a `.` as separator. The amount can be a number up to `999999999.99`. All amounts are in `AOA` (Angolan Kwanza).
terminal_type | The type of terminal that accepted the payment. `01` for ATM, `05` or `06` for Internet Banking.
terminal_transaction_id | Local identifier of the payment in the terminal. It matches the Transaction ID printed in the ATM receipt.
terminal_location | Description of the location where the payment ocurred (valid for payments in ATM's)
custom_fields | This is copied from the reference that originated the payment, for reconcilitation purposes.

## Fetch New Payments

```shell
curl "https://api.proxypay.co.ao/events/payments" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
```

> The above command returns JSON structured like this:

```json
{
  "payments": [
    {
      "terminal_type": "01",
      "terminal_transaction_id": "00123",
      "terminal_location": "Luanda",
      "terminal_id": "00456",
      "reference_number": "283749832",
      "reference_id": "8uVigNJ7Jj4hvVMdhQ",
      "id": "449500352608",
      "entity_id": "99999",
      "datetime": "2015-05-10T17:43:10Z",
      "custom_fields": {
        "invoice": "2014/0097"
      },
      "amount": "5000.00"
    }
  ]
}
```

This endpoint retrieves all Payment events that have not been acknowledged.

### HTTP Request

`GET https://api.proxypay.co.ao/events/payments`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
n | 100 | Max. number of payment events to return. This must be between `1` and `100`


<aside class="notice">
You must must acknowledge the payment events after processing them.
</aside>

## Acknowledge a Payment

```shell
curl -XDELETE "https://api.proxypay.co.ao/events/payments/449500352608" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
```

> The above command returns HTTP Status `204 No Content`.

This endpoint acknowledges that a specific payment has been processed.

### HTTP Request

`DELETE https://api.proxypay.co.ao/events/payments/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the payment to acknowledge as processed


Note: If your HTTP client doesn't allow sending a request with the HTTP verb `DELETE`, you can use `POST` instead and pass the additional URL parameter `_method=delete`.


## Acknowledge Multiple Payments

```shell
curl -XDELETE "https://api.proxypay.co.ao/events/payments" \
  -u "api:reh8inj33o3algd2tpi6tkcnrqf8rjj2"
  -H 'Content-Type: application/json'
  -d '{"ids": ["449500352608", "449500352609"]}'
```

> The above command returns HTTP Status `204 No Content`.

This endpoint acknowledges that multiple payments have been processed.

### HTTP Request

`DELETE https://api.proxypay.co.ao/events/payments`

### URL Parameters

Parameter | Description
--------- | -----------

Note: If your HTTP client doesn't allow sending a request with the HTTP verb `DELETE`, you can use `POST` instead and pass the additional URL parameter `_method=delete`.


# Payment Events - Push API

As an alternative to the Pull API, ProxyPay can instead push new payment events to a URL provided by you.

You must provide an HTTP or HTTPS endpoint that is publicly accessible over the Internet.

## Callback

Your callback endpoint will receive the data encoded as JSON.

In some situations, such as a timeout when calling your endpoint, the same payment may be submitted multiple times. You should make sure your logic is idempotent.

To authenticate the request, we include an HMAC-SHA-256 signature in every request. In addition to all the attributes of the payment event, already detailed in the Pull API, we also submit the following extra attributes:

Attribute | Description
--------- | -----------
timestamp | A Unix Epoch timestamp generated at the time of the request
signature | Signature calculated as `HMAC-SHA-256(key, timestamp+checksum_data)`, where key is your API Key, and represented in hex digits.


> Your endpoint will receive a POST request with a JSON structured like this:

```json
{
  "payment": {
    "terminal_type": "01",
    "terminal_transaction_id": "00123",
    "terminal_location": "Luanda",
    "terminal_id": "00456",
    "reference_number": "283749832",
    "reference_id": "8uVigNJ7Jj4hvVMdhQ",
    "id": "449500352608",
    "entity_id": "99999",
    "datetime": "2015-05-10T17:43:10Z",
    "custom_fields": {
      "invoice": "2014/0097",
      "customer_name": "Acme"
    },
    "amount": "5000.00"
  },
  "meta": {
    "timestamp": "1428262214",
    "signature": "809427D33F649EDB8A7C123D76E5B426C37DFE6B56C9160509CFCAA01C86F844"
  }
}
```

Your endpoint must return `HTTP Status 200` after successfully processing the event. Any other status code will be considered an error and ProxyPay will retry after 10 minutes.

The `checksum_data` is calculated by appending the values of all attributes in the following order:

`checksum_data` = `amount` + `datetime` + `entity_id` + `id` + `reference_id` + `reference_number` + `terminal_id` + `terminal_location` + `terminal_transaction_id` + `terminal_type` + *customer_fields_in_alphabetical_order*

In the example, *customer_fields_in_alphabetical_order* = `customer_name` + `invoice`

*Example*

**API Key**

`h5a4e6ctej01hn9agh7uggt5n8r29ups`

**timestamp**

`1428262214`

**checksum_data**

 `5000.002015-05-10T17:43:10Z999994495003526088uVigNJ7Jj4hvVMdhQ28374983200456Luanda0012301Acme2014/0097`

**signature**

`809427D33F649EDB8A7C123D76E5B426C37DFE6B56C9160509CFCAA01C86F844`

`signature` == `HMAC-SHA-256`(`API Key`, `timestamp`+`checksum_data`)


### HTTP Request

`POST http(s)://<your callback url>`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------


### Validating the Signature

To validate the signature, simply calculate the expected signature using `HMAC-SHA-256`, compare it to the signature in the request and reject the data if they are different.
