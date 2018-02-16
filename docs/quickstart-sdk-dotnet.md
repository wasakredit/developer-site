# Quick start using the .NET SDK

**Table of content**

* [Authentication](#authentication)
* [Initialize WasaKreditClient](#initialize_wasakreditclient)
* [Create checkout](#create_checkout)
* [Initialize checkout](#initialize_checkout)
* [Handling custom checkout callbacks](#custom_callbacks)
* [Handling order status changes via pingbacks](#pingbacks)
* [Add order references](#add_order_reference)
* [Complete an order](#complete_order)
* [Cancel an order](#cancel_order)

---

## <a name="authentication">Authentication</a>

Create a new instance of AuthenticationClient by passing in the client id and client secret provided from WasaKredit.

It is a good idea to create the AuthenticationClient as a singleton during your application initialization,
this will allow the AuthorizationClient to store your access token in memory for reusability between client requests.

```c#
var authenticationClient = AuthenticationClient.Instance;
authenticationClient.SetClientCredentials("[your client id here]", "[your client secret here]");
authenticationClient.GetAccessToken();
```

## <a name="initialize_wasakreditclient">Initialize WasaKreditClient</a>

Get the singleton instance of WasaKreditClient by getting the `Instance` property of the WasaKreditClient. Call the `Initialize` method passing your `AuthenticationClient` and a boolean representing whether or not you wish
to use the client in test mode or not.

```c#
bool testMode = true;

var wasaKreditClient = WasaKreditClient.Instance;
wasaKreditClient.Initialize(authenticationClient, testMode);
```

## <a name="create_checkout">Create checkout</a>

Create a new checkout request and pass it to the `CreateCheckout` method on your `WasaKreditClient`.

### Create checkout example

```c#
var request = new CreateCheckoutRequest
{
  PaymentTypes = "leasing", // Optional
  OrderReferences = new [] // Optional but strongly recommended.  
            {
                new OrderReference
                {
                    Key = "temp_order_number",
                    Value = "12345"
                }
            }
  CartItems = new List<CartItem>
            {
                new CartItem
                {
                    ProductId = "ez-32131",
                    ProductName = "Kylskåp EZ3",
                    PriceExVat = new Price
                    {
                        Amount = "10000.0",
                        Currency = "SEK"
                    },
                    Quantity = 1,
                    VatAmount = new Price
                    {
                        Amount = "2500.00", // Vat per item
                        Currency = "SEK"
                    },
                    VatPercentage = "25",
                    ImageUrl = "http://image.com"
                }
            },
  ShippingCostExVat = new Price
  {
      Amount = "250.00",
      Currency = "SEK"
  },
  CustomerOrganizationNumber = "2222222-2222", // Optional
  PurchaserName = "Anders Svensson", // Optional
  PurchaserEmail = "email@example.com", // Optional
  PurchaserPhone = "07001234567", // Optional
  BillingAddress = new Address // Optional
            {
                City = "Göteborg",
                CompanyName = "Star Republic AB",
                Country = "Sweden",
                PostalCode = "41116",
                StreetAddress = "Eklundsgatan 9"
            },
  DeliveryAddress = new Address // Optional
            {
                City = "Göteborg",
                CompanyName = "Star Republic AB",
                Country = "Sweden",
                PostalCode = "41116",
                StreetAddress = "Eklundsgatan 9"
            },
  RecipientName = "Anders Svensson", // Optional
  RecipientPhone = "07001234567", // Optional
  RequestDomain = "https://www.wasa-partner.se", // Optional, but needed to resize iframe automatically.
  PingUrl = "https://www.wasa-partner.se/payment-callback/" // Optional, but needed for status updates.
};

var response = wasaKreditClient.CreateCheckout(request);
```

Note that the `OrderReferences` property is a collection. Even if you don't want to create an order in your system at before creating a Wasa Kredit checkout, you have the possibility to supply a temporary identifier to be able to match the Wasa Kredit order with some reference in your system. You also have the option to add additional reference identifiers at a later time, for example when your final order is created (see [Add order references](#add_order_reference)).

The URL that you supply with the `PingUrl` property should be an endpoint that is set up to receive a POST message and return an http status code 200 response on success.

The return object of the `CreateCheckout` method is a html snippet which you should embed in your web page, inside of which the Wasa Kredit Checkout widget will handle the payment flow.

## <a name="initialize_checkout">Initialize checkout</a>

After creating a Wasa Kredit Checkout by calling the `CreateCheckout` method and embedding the resulting html snippet in your web page, the checkout must be initialized by calling the global javascript function `window.wasaCheckout.init()`.

```javascript
<script>
    window.wasaCheckout.init();
</script>
```

### <a name="custom_callbacks">Handling custom checkout callbacks</a>

Optionally, you're able to pass an options object to the `init` javascript function. Use this if you want your own custom handling of the onComplete, onRedirect and onCancel checkout events.

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

The `onComplete` event will be raised when a user has completed the checkout process. We recommend that you convert your cart/checkout to an order here if you haven't done it already.

The `onRedirect` event will be raised the user clicks the "back to store/proceed"-button.

The `onCancel` event will be raised if the checkout process is canceled by the user or Wasa Kredit.

All callback functions will get the `orderReferences` parameter passed from the checkout. This parameter consists of an Array of `KeyValue` objects.

These are the same values as the ones that was passed to the `CreateCheckout` method as the `OrderReferences` property and also Wasa Kredit order id.

## <a name="pingbacks">Handling order status changes via pingbacks</a>

When calling the `CreateCheckout` method, Wasa Kredit will create an order. When the order is created or when the order status is updated, you will receive a POST to the supplied `PingUrl`, which contains the following body:

```json
{
  "order_id" : "9c722707-123a-44e7-9eba-93e3a372d57e",
  "order_status": "initialized"
}
```

For further information about the order status flow, see the [order flow chart](https://developer.wasakredit.se/order-flow-chart/).

Using the Wasa Kredit order id (`order_id`), provided in the pingback body, you are able to get the entire order object by calling the [GetOrder](#get_order_method) method.

### Get order example

```c#
var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";
var order = wasaKreditClient.GetOrder(orderId);
```

The response object from the `GetOrder` method is quite extensive. For further information about this response object see [GetOrderResponse](#get_order_response)

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

### Best practises

The preferred point in time to create the order in your system is when receiveing a pingback with order status "pending".

To match the Wasa Kredit order against your internal cart/checkout/order, call the `GetOrder` method and use the `OrderReferences` property in the return object.

## <a name="add_order_reference">Add order references</a>

To be able to match your internal cart/checkout/order against the Wasa Kredit order you are able to provide an unlimited set of order references. Order references might be provided in two ways.

1. In the Create Checkout request.
2. By calling the `AddOrderReference` method. This might be done at anytime as long as you have the Wasa Kredit order id. *Notice that this operation will add the additional order references to any previous order references.*

The order reference object is a collection of key-value pairs where the key is the reference identifier (i.e. describes the type of reference) and the value is the actual reference id.

Order referenses are added one at a time by calling the `AddOrderReference` method.

### Add order reference example

```c#
string orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";
string quoteId = "fce34f82-de23-4f29-b19b-5e01bc4a3cf6";
var request = new AddOrderReferenceRequest { Key = "Quote", Value = quoteId };
var response = client.AddOrderReference(orderId, request);
```

## <a name="complete_order">Complete an order</a>

1. When an order is ready to be shipped to the customer (i.e. it has been signed by all necessary signees and is fully approved by Wasa Kredit), you will receive a pingback with order status `ready_to_ship`. You should now ship the order items to the customer.
2. When the order items are shipped to the customer, call the `UpdateOrderStatus` method (see example below) with the status `shipped`.
3. As a confirmation you will now receive a pingback with order status `shipped`.

## <a name="cancel_order">Cancel an order</a>

To cancel an order, call the `UpdateOrderStatus` method (see example below) with the status `canceled`. Orders that have already been shipped cannot be canceled.

### Update order status example

```c#

var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";

var request = new UpdateOrderStatusRequest
{
    OrderId = orderId,
    Status = new OrderStatus {Status = "shipped"}
};

var response = client.UpdateOrderStatus(request);
```

## Responses

### <a name="get_order_response">GetOrderResponse</a>

| Name | Type | Description |
|---|---|---|
| CustomerOrganizationNumber | *string* | The organization number of the customer who made the purchase. |
| BillingAddress | *Address* | ... |
| DeliveryAddress | *Address* | ... |
| OrderReferences | *List[**OrderReference**]* (required) | A list containing order reference objects. |
| PurchaserEmail | *string* | The email of the person performing the purchase. | 
| RecipientName | *string* | The name of the person who should receive the order. |
| RecipientPhone | *string* | The phone number of the person who should receive the order. |
| Status | *OrderStatus* | The status that the order is in at Wasa Kredit. |
| CartItem | *List[Cart Item]* | A list of the items purchased as Cart Item objects. |

### Address

| Name | Type | Description |
|---|---|---|
| CompanyName | *string* | Company name |
| StreetAddress | *string* | Street address |
| PostalCode | *string* | Postal code |
| City | *string* | City |
| Country | *string* | Country |

### OrderReference

| Name | Type | Description |
|---|---|---|
| Key | *string* | A key to succinctly describe the reference. *Ex: "temp_order_reference"*. |
| Value | *string* | The actual order reference value. |

### OrderStatus

| Name | Type | Description |
|---|---|---|
| Status | *string* | The status that the order is in at Wasa Kredit. |