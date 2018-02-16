# Quick start using the API

## Get access token

All the calls to the API need to include an access token header. To get the access token, POST to `/connect/token/` with the body:

```json
{
	"client_id" : "12983019283",
	"client_secret" : "120csk20fa9g1350vm",
	"grant_type": "client_credentials"
}
```

The response body will look something like this:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjlFM0Q1Qjg1NUU0RTM4OTk3Q0Y5OTk4RkJEMDA5NzAyQzBGMzFFMkUiLCJ0eXAiOiJKV1QiLCJ4NXQiOiJuajFiaFY1T09KbDgtWm1QdlFDWEFzRHpIaTQifQ.eyJuYmYiOjE1MDk1NDAzNzAsImV4cCI6MTUwOTU0Mzk3MCwiaXNzIjoiaHR0cDovL3drY2xvdWQtZnJvbnQtbGItMTI2ODc4NjA3MS5ldS1jZW50cmFsLTEuZWxiLmFtYXpvbmF3cy5jb206ODMiLCJhdWQiOlsiaHR0cDovL3drY2xvdWQtZnJvbnQtbGItMTI2ODc4NjA3MS5ldS1jZW50cmFsLTEuZWxiLmFtYXpvbmF3cy5jb206ODMvcmVzb3VyY2VzIiwicHVibGljLmNoZWNrb3V0LmFwaSJdLCJjbGllbnRfaWQiOiI0NzUwN2U5Ny1iZDBlLTQ4OGUtOWFkYi03ZDY0NGQyYmVmZGMiLCJzY29wZSI6WyJwdWJsaWMuY2hlY2tvdXQuYXBpLnJldGFpbGVyIl19.zuE5F6ChHLh5ZOT-CaaXAMLod6roalzz2DKu93_yKGQ-BzHRkgaDXQRMr4xKf5op-o990lW4Hp513vGXWOvIqYiWEV8fv49tw3jivpXm3els2ewebkTjeAptJ99_yKw58Qut9dLIreAX_x2PFH6i5nU7XZXJdx8vmspC1fSXOryuqSNU29pbqPwmRzZyi0qpo9LiDAkXdgm7e2hOilUOJEkdS708EugfT3qBEQ8ONLGUNCIq-q6aLoQEDq0dp_uR6EYDyEPRGG1paWy5Cp7cyD63aTpsXlgGCiIaPDA6O2PTogKw6HadTN4oKHxZ2sFyRPWqbtAe0VOOmSC-ErpZaw",
  "expires_in": "3600",
  "token_type": "\"Bearer\""
}
```

When performing any other calls to the API the `access_token` value has to be included in the Authorization header.


## Create checkout

The first step is to POST to `/checkouts` with the following body:
```json
{
  "payment_types": "leasing",
  "order_reference_id": "a1be9394-182d-49c7-a470-ea59e68ce3ef",
  "order_references": [
    {
      "key": "partner_order_number",
      "value": "123456"
    }
  ],
  "cart_items": [
    {
      "product_id": "ez-41239b",
      "product_name": "Kylskåp EZ3",
      "price_ex_vat": {
        "amount": "14995.50",
        "currency": "SEK"
      },
      "quantity": 1,
      "vat_percentage": "25",
      "vat_amount": {
        "amount": "14995.50",
        "currency": "SEK"
      },
      "image_url": "https://unsplash.it/500/500"
    }
  ],
  "shipping_cost_ex_vat": {
    "amount": "14995.50",
    "currency": "SEK"
  },
  "customer_organization_number": "222222-2222",
  "purchaser_name": "Anders Svensson",
  "purchaser_email": "purchaser@gmail.com",
  "purchaser_phone": "070-1234567",
  "billing_address": {
    "company_name": "Star Republic",
    "street_address": "Ekelundsgatan 9",
    "postal_code": "41118",
    "city": "Gothenburg",
    "country": "Sweden"
  },
  "delivery_address": {
    "company_name": "Star Republic",
    "street_address": "Ekelundsgatan 9",
    "postal_code": "41118",
    "city": "Gothenburg",
    "country": "Sweden"
  },
  "recipient_name": "Anders Svensson",
  "recipient_phone": "070-1234567",
  "request_domain": "https://www.wasa-partner.se",
  "confirmation_callback_url": "https://www.wasa-partner.se/payment-callback/",
  "ping_url": "https://www.wasa-partner.se/order-updated/"
}
```

Note that the property `order_references` is a collection so that even if you don't want to create an order in your system before creating a Wasa Kredit checkout you have the possibility to supply a temporary identifier to be able to match the Wasa Kredit order with some reference in your system. You also have the option to add additional reference identifiers at a later time, for example when your final order is created.

The URL that you supply through the `ping_url` property should be an endpoint that is set up to receive a POST message and return an http status code 200 response on success.

You will receive an html snippet which you should embed in your web page, inside of which the Wasa Kredit Checkout widget will handle the payment flow.

## Initialize checkout

After creating the Wasa Kredit checkout by POSTing to `/checkouts`, as described above, the resulting html snippet needs to be explicitly initialized through a javascript call to the global `window.wasaCheckout.init()` function. The `init` method call will populate the \<div\> contained in the html snippet and link it to an internal iframe.

```javascript
<script>
    window.wasaCheckout.init();
</script>
```

### Handling custom checkout callbacks

Optionally, you're able to pass an options object to the `init`-function. Use this if you want to manually handle the onComplete, onRedirect and onCancel events.

```javascript
<script>
    var options = {
      onComplete: function(orderReferences){
        //[...]
      },
      onRedirect: function(orderReferences){
        //[...]
      },
      onCancel: function(orderReferences){
        //[...]
      }
    };   
    window.wasaCheckout.init(options);
</script>
```

The `onComplete` event will be raised when a User has completed the checkout process. We recommend that you convert your cart/checkout to an order here if you haven't done it already.

The `onRedirect` event will be raised the user clicks the "back to store/proceed"-button. The default behaviour will redirect the user to the `confirmation_callback_url` passed into the `CreateCheckout`-request.

The `onCancel` event will be raised if the checkout process is canceled by the user or Wasa Kredit.

All callback functions will get the `orderReferences` parameter passed from the checkout. This parameter consists of an Array of `KeyValue` objects.

These are the same values as the ones that was passed to the `CreateCheckout`-request as the `order_references` property.

```javascript
orderReferences = [
	{ key: "partner_checkout_id", value: "900123" },
	{ key: "partner_reserved_order_number", value: "123456" }
];    
```

## Handling order status changes via pingbacks

When calling the Create Checkout operation, Wasa Kredit will also create an order. When the order is created or when the order status is updated, you will receive a POST to the supplied `ping_url`, which contains the following body:

```json
{
  "order_id" : "9c722707-123a-44e7-9eba-93e3a372d57e",
  "order_status": "initialized"
}
```

For further information about the order status flow, see the [order flow chart](https://developer.wasakredit.se/order-flow-chart/).

Using the Wasa Kredit order id (`order_id`), provided in the pingback body, you are able to get the entire order object through a GET from `/orders/{order_id}`, which will return the following response:

```json
{
  "customer_organization_number": "222222-2222",
  "delivery_address": {
    "company_name": "Star Republic",
    "street_address": "Ekelundsgatan 9",
    "postal_code": "41118",
    "city": "Gothenburg",
    "country": "Sweden"
  },
  "billing_address": {
    "company_name": "Star Republic",
    "street_address": "Ekelundsgatan 9",
    "postal_code": "41118",
    "city": "Gothenburg",
    "country": "Sweden"
  },
  "order_references": [
    {
      "key": "partner_order_number",
      "value": "123456"
    }
  ],
  "purchaser_email": "purchaser@gmail.com",
  "recipient_name": "Anders Svensson",
  "recipient_phone": "070-1234567",
  "status": {
    "status": "delivered"
  },
  "cart_items": [
    {
      "product_id": "ez-41239b",
      "product_name": "Kylskåp EZ3",
        "amount": "14995.50",
        "currency": "SEK"
      },
      "quantity": 1,
      "vat_percentage": "25",
      "vat_amount": {
        "amount": "14995.50",
        "currency": "SEK"
      },
      "image_url": "https://unsplash.it/500/500"
    }
  ]
}
```

### Pingback Statuses

Each time the order status is updated you will receive a pingback, enabling you to take action on the status change.

The possible order statuses are:

- initialized
  - The order has been created but the order agreement has not been signed by the customer.
- canceled
  - The purchase was not approved by Wasa Kredit. If you have created an order in your system it can safely be deleted.
- pending
  - The checkout has been completed and a customer has signed the order agreement, but additional signees is required or the order has not yet been fully approved by Wasa Kredit.
- ready_to_ship
  - All necessary signees have signed the order agreement and the order has been fully approved by Wasa Kredit. The order must now be shipped to the customer before Wasa Kredit will issue the payment to you as a partner.
- shipped
  - This status is set by the partner when the order item(s) have been shipped to the customer.

### Best Practises

The preferred point in time to create the order in your system is when receiveing a pingback with order status "pending".

To match the Wasa Kredit order against your internal cart/checkout/order, issue a GET against `/orders/{order_id}` and use the `order_references`-object in the response.

## Create order reference

To be able to match your internal cart/checkout/order against the Wasa Kredit order you are able to provide an unlimited set of order references. Order references might be provided in two ways.

1. In the Create Checkout request.
2. By calling the Add Order Reference operation through a POST to `/orders/{order_id}/order-references`. This might be done at anytime as long as you have the Wasa Kredit order id. *Notice that this operation will add the additional order references to any previous order references.*

The order reference object is a list of key-value pairs where the key is the reference identifier (i.e. describes the type of reference) and the value is the actual reference id.

```json
[
  {
    "key" : "partner-cart-reference",
    "value" : "56065dae-5d00-4ccd-aa8f-009ba8d0d137"
  },
  {
    "key" : "partner-checkout-reference",
    "value" : "d79903b0-108b-48f5-9ee1-6ca8062031e2"
  }
]
```

Order referenses are added one at a time through POSTs to `/orders/{order_id}/order-references`.

## Complete an order

1. When an order is ready to be shipped to the customer (i.e. it has been signed by all necessary signees and fully approved by Wasa Kredit), you will receive a pingback with order status `ready_to_ship`. You should now ship the order items to the customer.
2. When the order items are shipped to the customer, call the Update Order Status operation by issuing a PUT to `/orders/{order_id}/status/{status}` with the status `shipped`.
3. As a confirmation you will now receive a pingback with order status `shipped`.

## Cancel an order

To cancel an order, call the Update Order Status operation by issuing a PUT to `/orders/{order_id}/status/{status}` with the status `canceled`. Orders that have already been shipped cannot be canceled.
