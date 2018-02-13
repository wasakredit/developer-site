# Quick start using the .NET SDK

**Table of content**

* [Authentication](#authentication)
* [Initialize WasaKreditClient](#initialize_wasakreditclient)
* [Create checkout](#create_checkout)
* [Handling custom checkout callbacks](#custom_callbacks)
* [Get order](#get_order)
* [Get order status](#get_order_status)
* [Create order reference](#create_order_reference)
* [Update order status](#update_order_status)
* [Cancel an order](#cancel_an_order)



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

Create a new instance of WasaKreditClient by getting the `Instance` property of the WasaKreditClient and calling
the `Initialize` method passing your `AuthenticationClient` and a boolean representing whether or not you wish
to use the client in test mode or not.

```c#
bool testMode = true;

var wasaKreditClient = WasaKreditClient.Instance;
wasaKreditClient.Initialize(authenticationClient, testMode);
```

## <a name="create_checkout">Create checkout</a>

Create a new checkout request and pass it to the `CreateCheckout` method on your `WasaKreditClient`.

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

Note that the property `order_references` is a collection so that even if you don't want to create an order in your system before creating a checkout you should supply a temporary identifier to be able to connect the Wasa Kredit order with something in your system. You still have the option to add additional identifiers when the final order is created.

The URL that you supply with the `ping_url` property should be an endpoint that is set up to receive a POST message and return an http status code 200 response on success.

You will receive an html snippet which you should embed in your web page, inside of which the Wasa Kredit Checkout widget will handle the payment flow.

When you want to initialize the checkout, just call the global ```window.wasaCheckout.init()```.

```javascript
<script>
    window.wasaCheckout.init();
</script>
```

## <a name="custom_callbacks">Handling custom checkout callbacks</a>

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

The ```onComplete``` event will be raised when a user has completed the checkout process. We recommend that you convert your cart/checkout to an order here if you haven't done it already.

The ```onRedirect``` event will be raised the user clicks the "back to store/proceed"-button.

The ```onCancel``` event will be raised if the checkout process is canceled by the user or Wasa Kredit.

All callback functions will get the ```orderReferences``` parameter passed from the checkout. This parameter consists of an Array of ```KeyValue``` objects.

These are the same values as the ones that was passed to the ```CreateCheckout```-method as the ```order_references``` property and also Wasa Kredit order id.

```javascript
orderReferences = [
    { key: "wasakredit-order-id", value: "9c722707-123a-44e7-9eba-93e3a372d57e" },
    { key: "partner_checkout_id", value: "900123" },
    { key: "partner_reserved_order_number", value: "123456" }
];    
```

## <a name="get_order">Get Order</a>

Wasa Kredit will create an order at some point in the checkout process, when that is depends on the payment option that the customer has chosen. When the order is updated you will receive a POST to the supplied `ping_url` which contains:

```json
{
  "order_id" : "9c722707-123a-44e7-9eba-93e3a372d57e"
}
```

To get the order referenced by the id simply call

```c#
var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";

var order = wasaKreditClient.GetOrder(orderId);
```

### Response

#### GetOrderResponse
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

#### Address

| Name | Type | Description |
|---|---|---|
| CompanyName | *string* | Company name |
| StreetAddress | *string* | Street address |
| PostalCode | *string* | Postal code |
| City | *string* | City |
| Country | *string* | Country |

#### OrderReference
| Name | Type | Description |
|---|---|---|
| Key | *string* | A key to succinctly describe the reference. *Ex: "temp_order_reference"*. |
| Value | *string* | The actual order reference value. |

#### OrderStatus
| Name | Type | Description |
|---|---|---|
| Status | *string* | The status that the order is in at Wasa Kredit. |


## <a name="get_order_status">Get order status</a>

Every time the order is updated you will receive a ping and can fetch the [order](#get_order) or just fetch the status with `GetOrderStatus` and can take action on the status that it's in.

To get the order status by the id simply call

```c#
var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";

var order = wasaKreditClient.GetOrderStatus(orderId);
```

### Response

#### GetOrderStatusResponse
| Name | Type | Description |
|---|---|---|
| Status | *OrderStatus* | The order identifier. |

#### OrderStatus
| Name | Type | Description |
|---|---|---|
| Status | *string* | The current order status at Wasa Kredit. |


The statuses that an order can be in are:

- canceled
  - The purchase was not approved, if you have created an order in your system it can safely be deleted using `order_references`
- pending
  - Wasa Kredit has created the order but it is being processed. You should use customer information and addresses from this order and update the order in your system with using `order_references` to find the matching order. If you have no order in your system you should create one now.
- ready_to_ship
  - The order is approved but has to be shipped to the customer before Wasa Kredit can make the payment to the partner.
- shipped
  - This status is set by the partner when the item(s) have been shipped to the customer.
- completed
  - The order has been shipped and payed.

## <a name="create_order_reference">Create order reference</a>

If orders are created in your system after you have created the checkout you will need to update the Wasa Kredit order with a reference to your order. Using the order id you received in the initial ping you can perform the following:

```c#
string orderId = "9c722707-123a-44e7-9eba-93e3a372d57e",
var request = new AddOrderReferenceRequest {Key = "Quote", Value = Guid.NewGuid().ToString()};

var response = client.AddOrderReference(orderId, request);
```

## <a name="update_order_status">Update order status</a>

After Wasa Kredit has finished their internal checks of the payment you will receive a ping on the supplied ping url. At this time you should check the status of the order and if everything is in order, ship the items and set the status to `shipped`. See an example below:

```c#

var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";

var status = wasaKreditClient.GetOrderStatus(orderId);

if(status != "ready_to_ship")
{
    // you could of course check for other statuses like 'canceled' and take necessary actions related to that status here
    return;
}

var request = new UpdateOrderStatusRequest
{
    OrderId = orderId,
    Status = new OrderStatus {Status = "shipped"}
};

var response = client.UpdateOrderStatus(request);
```

## <a name="complete_order">Complete order</a>

After an order has been shipped by the and the status has been set to `shipped` you will receive a ping when Wasa Kredit has performed any final validations and made the payment to the partner. To validate the status of the order, refer to the following example:

```c#
var orderId = "9c722707-123a-44e7-9eba-93e3a372d57e";

var status = wasaKreditClient.GetOrderStatus(orderId);

if(status == "completed")
{
	// Any steps that you wish to perform on your end
}
```

## <a name="cancel_an_order">Cancel an order</a>

To cancel an order simply call ```UpdateOrderStatus(UpdateOrderStatusRequest)``` with the status `canceled`. Note that this can only be done if the current status is not `shipped` or `completed`. See example below:

```c#
var request = new UpdateOrderStatusRequest
{
    OrderId = "9c722707-123a-44e7-9eba-93e3a372d57e",
    Status = new OrderStatus {Status = "canceled"}
};

var response = client.UpdateOrderStatus(request);
```
