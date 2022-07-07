Created by Elliptic Marketing, Mailstats handles email marketing deliveries and logic for multiple clients. 
This guide covers the process of integrating Mailstats with an external application, in order for Mailstats to deliver emails based on foreign data. 

## Table of Contents
- [Setting up Webhooks](#setting-up-webhooks)
- [Payload Requirements](#payload-requirements)
- [Responses and Errors](#responses-and-errors)
- [Webhook Events](#webhook-events)
    - [Subscription](#subscription)
    - [Abandoned Cart](#abandoned-cart)
    - [Order](#order)
    - [Order Cancelled](#order-cancelled)
    - [Product is Back in Stock](#product-is-back-in-stock)

## Setting up webhooks
Your app can send data to Mailstats via webhooks. You can set up different webhook events based on the type of data that you are sending.
### Getting your URL schema
The basic schema for each endpoint looks like the following:
`https://mailstats.ellipticmarketing.com/api/integrations/{app_name}/{country_name}/{event}`

For example:
` https://mailstats.ellipticmarketing.com/api/integrations/vtex/chile/subscription`

## Payload Requirements

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

## Responses and Errors

- 200 - SUCCESS: Returned if your data payload is accepted.
- 401 - UNAUTHORIZED: Returned when your token is missing or is incorrect. 
- 5xx - SERVER ERROR: Returned when there's an issue with the server. Your app MUST retry sending the same payload after a set cooldown period. We recommend setting a wait period of 15 minutes for 10 retries.

## Webhook Events
### Subscription

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

### Abandoned Cart
This endpoint should be used after a user abandons a shopping cart without finalizing a purchase. You should only send any webhook data after you ensure that the cart is abandoned. For example, after 15 minutes of user inactivity. 

**Webhook path:** `/abandonedcart`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `cart_url`* – This is an URL that can regenerate the shopping cart for the customer. 
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `ean`*
    - `name`*
    - `url`*
    - `thumbnail_url`*
    
\* Denotes a mandatory field

### Order
This endpoint should be used after a customer successfully places an order. 

**Webhook path:** `/order`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `order_id`*
- `order_date`
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `ean`*
    - `name`*
    - `url`*
    - `thumbnail_url`*
    
\* Denotes a mandatory field

### Order Cancelled
This endpoint should be used after an order is cancelled. 

**Webhook path:** `/ordercancelled`

**Payload requirements:**

Your data payload must include the following:

- `order_id`*

\* Denotes a mandatory field

### Product is Back in Stock

This endpoint should be used when a product comes back in stock and a customer has subscribed to be notified when the product is back in stock. As multiple customers may subscribe to the same product, your app must send separate POST calls for each customer. 

**Webhook path:** `/backinstock`

**Payload requirements:**

Your data payload must include the following:

- `email`*
- `customer_id` – This is a unique ID that your app assigns each customer.
- `first_name`
- `last_name`
- `products`* – An array of the products left in the cart. Each product item must include the following:
    - `ean`*
    - `name`*
    - `url`*
    - `thumbnail_url`*

\* Denotes a mandatory field
