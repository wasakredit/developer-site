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

Note that the property `order_references` is a collection so that even if you don't want to create an order in your system before creating a checkout you should supply a temporary identifier to be able to connect the Wasa Kredit order with something in your system. You still have the option to add additional identifiers when the final order is created.

The URL that you supply with the `ping_url` property should be an endpoint that is set up to receive a POST message and return an http status code 200 response on success.

You will receive an html snippet which you should embed in your web page, inside of which the Wasa Kredit Checkout widget will handle the payment flow.

When you want to initialize the checkout, just call the global ```window.wasaCheckout.init()```.

```javascript
<script>
    window.wasaCheckout.init();
</script>
```

### Handling custom checkout callbacks

Optionally, you're able to pass an options object to the ```init```-function. Use this if you want to manually handle the onComplete, onRedirect and onCancel events.

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

The ```onComplete``` event will be raised when a User has completed the checkout process. We recommend that you convert your cart/checkout to an order here if you haven't done it already.

The ```onRedirect``` event will be raised the user clicks the "back to store/proceed"-button. The default behaviour will redirect the user to the ```confirmation_callback_url``` passed into the ```CreateCheckout```-request.

The ```onCancel``` event will be raised if the checkout process is canceled by the user or Wasa Kredit.

All callback functions will get the ```orderReferences``` parameter passed from the checkout. This parameter consists of an Array of ```KeyValue``` objects.

These are the same values as the ones that was passed to the ```CreateCheckout```-request as the ```order_references``` property.

```javascript
orderReferences = [
	{ key: "partner_checkout_id", value: "900123" },
	{ key: "partner_reserved_order_number", value: "123456" }
];    
```

## Handling order status changes

Wasa Kredit will create an order at some point in the checkout process, when that is depends on the payment option that the customer has chosen. When the order is updated you will receive a POST to the supplied `ping_url` which contains:

```json
{
  "order_id" : "9c722707-123a-44e7-9eba-93e3a372d57e"
}
```

Using this id you can now GET from `/orders/{order_id}` which will return the following:

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

Every time the order is updated you will receive a ping and can fetch the order in the same fashion and can take action on the status that it's in.
The statuses that an order can be in are:

- canceled
  - The purchase was not approved, if you have created an order in your system it can safely be deleted using `order_references`
- pending
  - Wasa Kredit has created the order but it is being processed. You should use customer information and addresses from this order and update the order in your system with using `order_references` to find the matching order. If you have no order in your system you should create one now.
- ready_to_deliver
  - The order is approved but has to be delivered to the customer before Wasa Kredit can make the payment to the partner.
- delivered
  - This status is set by the partner when the item(s) have been delivered to the customer.
- completed
  - The order has been delivered and payed.

## Create order reference

If orders are created in your system after you have made the POST to `/checkouts` you will need to update the Wasa Kredit order with a reference to your order. Perform a POST call to `/orders/{order_id}/order-references` with the body:

```json
{
  "key" : "succint_description_of_the_reference",
  "value" : "123456"
}
```

## Complete order

1. Perform a GET call to `/orders/{order_id}/status`. This will just return the status that your order is in. If it is `ready_to_deliver` the partner should send the item(s) to the customer.
2. Perform a PUT call to `/orders/{order_id}/status/{status}` with the status `delivered` when the item(s) have been shipped.
3. You will receive a ping when Wasa Kredit updates the state of the order, perform a GET call to `/orders/{order_id}/status` to check the status. If it is set to `completed` all payments have been made and the item(s) should be delivered to the customer.

## Cancel an order

Simply perform a PUT call to `/orders/{order_id/status/{status}` with the status `canceled`. This can only be done on orders that don't have status `delivered` or `completed`
