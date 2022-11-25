Created by Elliptic Marketing, Mailstats handles email marketing deliveries and logic for multiple clients. 
This guide covers the process of integrating Mailstats with an external application, in order for Mailstats to deliver emails based on foreign data. 

## Table of Contents
- [Webhooks](#webhooks)
  - [Setting up Webhooks](#setting-up-webhooks)
  - [Webhooks Payload Requirements](#webhooks-payload-requirements)
  - [Webhooks Responses and Errors](#webhooks-responses-and-errors)
  - [Webhook Events](#webhook-events)
      - [Subscription](#subscription)
      - [Abandoned Cart](#abandoned-cart)
      - [Order](#order)
      - [Order Cancelled](#order-cancelled)
      - [Product is Back in Stock](#product-is-back-in-stock)
- [API](#api)
  - [Setting up the API](#setting-up-the-api)
  - [API Payload Requirements](#api-payload-requirements)
  - [API Responses and Errors](#api-responses-and-errors)
  - [API Endpoints](#api-endpoints)
      - [Subscribers](#subscribers)

## Webhooks

### Setting up webhooks
Your app can send data to Mailstats via webhooks. You can set up different webhook events based on the type of data that you are sending.

#### Getting your URL schema
The basic schema for each endpoint looks like the following:
`https://mailstats.ellipticmarketing.com/api/integrations/{app_name}/{country_name}/{event}`

For example:
` https://mailstats.ellipticmarketing.com/api/integrations/vtex/chile/subscription`

### Webhooks Payload Requirements

- Each payload must be sent through a `POST` request. 
- Data must be JSON encoded.
- Each request must include an authorization header with a token provided by Elliptic Marketing. An authorization header may look as follows: `Authorization: Bearer xxxxxxxxx`.

Payload Example:

```
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@test.test",
  "customer_id": "AX-BC-00264",
  "birthday": "1990-12-31",
}
```

### Webhooks Responses and Errors

- 200 - SUCCESS: Returned if your data payload is accepted.
- 401 - UNAUTHORIZED: Returned when your token is missing or is incorrect. 
- 5xx - SERVER ERROR: Returned when there's an issue with the server. Your app MUST retry sending the same payload after a set cooldown period. We recommend setting a wait period of 15 minutes for 10 retries.

### Webhook Events
#### Subscription

This endpoint should be used when a customer provides an email address with the intention of subscribing to a promotional mailing list. You must have consent from the customer before providing their information to Mailstats.

**Webhook path:** `/subscription`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each user.
- `first_name`
- `last_name`
- `birthday` - This is the customer's birthday in the format `YYYY-MM-DD`.

\* Denotes a mandatory field

#### Abandoned Cart
This endpoint should be used after a user abandons a shopping cart without finalizing a purchase. You should only send any webhook data after you ensure that the cart is abandoned. For example, after 15 minutes of user inactivity. 

**Webhook path:** `/abandonedcart`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `cart_id`
- `cart_time` - Time in [ISO 8601](https://www.w3.org/TR/NOTE-datetime) format (e.g., 2019-02-01T03:45:27+04:00).
- `cart_url`* – This is an URL that can regenerate the shopping cart for the customer. 
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `reference`*
    - `name`*
    - `url`*
    - `thumbnail_url`*
    
\* Denotes a mandatory field

#### Order
This endpoint should be used after a customer successfully places an order. 

**Webhook path:** `/order`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `order_id`*
- `order_time` - Time in [ISO 8601](https://www.w3.org/TR/NOTE-datetime) format (e.g., 2019-02-01T03:45:27+04:00).
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `reference`*
    - `name`*
    - `url`*
    - `thumbnail_url`*
    
\* Denotes a mandatory field

#### Order Cancelled
This endpoint should be used after an order is cancelled. 

**Webhook path:** `/ordercancelled`

**Payload requirements:**

Your data payload must include the following:

- `order_id`*

\* Denotes a mandatory field

#### Product is Back in Stock

This endpoint should be used when a product comes back in stock and a customer has subscribed to be notified when the product is back in stock. As multiple customers may subscribe to the same product, your app must send separate POST calls for each customer. 

**Webhook path:** `/backinstock`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `reference`*
    - `name`*
    - `url`*
    - `thumbnail_url`*

\* Denotes a mandatory field


## API

### Setting up the API
Your app can request data from Mailstats via the API. You can hit different API endpoints based on the type of data that you are requesting.

#### Getting your URL schema
The basic schema for each endpoint looks like the following:
`https://mailstats.ellipticmarketing.com/api/resources/{app_name}/{model}`

For example:
`https://mailstats.ellipticmarketing.com/api/resources/diorlatina/subscribers`

### API Payload Requirements

- Each request must include an authorization header with a token provided by Elliptic Marketing. An authorization header may look as follows: `Authorization: Bearer xxxxxxxxx`.
- Each request must include a `Content-Type` header with a value of `application/json`.

### API Responses and Errors

- 200 - SUCCESS: Returned if your data payload is accepted.
- 400 - BAD REQUEST: Returned when your request fails validation.
- 401 - UNAUTHORIZED: Returned when your token is missing or is incorrect.
- 429 - TOO MANY REQUESTS: Returned when you have made too many requests in a given time period.

### API Endpoints

#### Subscribers

**Endpoint:** `/subscribers`

**Method:** `GET`

**Description:** Returns a list of all subscribers.

**Query parameters:**
- `from` – The date from which to return subscribers based on last update. Must be in the format `YYYY-MM-DD` or other [format compatible](https://www.php.net/manual/en/datetime.formats.php) with PHP's `strtotime` function.
- `to` – The date to which to return subscribers based on last update.  Must be in the format `YYYY-MM-DD` or other [format compatible](https://www.php.net/manual/en/datetime.formats.php) with PHP's `strtotime` function.
- `status` – Includes records matching subscription status of `subscribed`, or `unsubscribed`.
- `source` – Includes records matching the given source.
- `exclude_source` – Excludes records matching the given source.
- `limit` – Deprecated. Use `status` instead.

**Example Request:** `https://mailstats.ellipticmarketing.com/api/resources/diorlatina/subscribers?from=2020-01-01&to=2020-01-31&exclude_source=vtex`
**Example Response:**

```
{
    "data": [
        {
            "email": "example@example.com",
            "unsubscribed_at": "2021-07-20T16:02:24.000000Z",
            "created_at": "2020-01-01T01:03:13.000000Z",
            "updated_at": "2020-01-01T16:02:24.000000Z"
        },
        ...
        {
            "email": "example2@example.com",
            "unsubscribed_at": null,
            "created_at": "2020-01-03T08:00:00.000000Z",
            "updated_at": "2020-01-03T08:14:55.000000Z"
        },
    ],
    "links": {
        "first": null,
        "last": null,
        "prev": null,
        "next": "http://mailstats.ellipticmarketing.com/api/resources/diorlatina/subscribers?cursor=eyJ1cGRhdGVkX2F0IjoiMjAyMS0wNy0wOSAxNzo0MzozNiIsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX9"
    },
    "meta": {
        "path": "http://mailstats.ellipticmarketing.com/api/resources/diorlatina/subscribers",
        "per_page": 25,
        "next_cursor": "eyJ1cGRhdGVkX2F0IjoiMjAyMS0wNy0wOSAxNzo0MzozNiIsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX9",
        "prev_cursor": null
    }
}
```
