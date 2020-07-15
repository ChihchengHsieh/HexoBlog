---
title: Stripe Payment for .NET and Flutter
tags:
    - Flutter
    - .NET
    - Stripe
categories: App Development
description: The stripe implementation on .NET and flutter. Payment methods includes Native Pay and Credit Card.
photos:
    - https://stripe.com/img/v3/home/social.png
---

<!--more-->

# Payment Methods

Two typs of the payment method we will be using in our App. 
- [x] Credit Card
- [x] Native
- [ ] WeChat
- [ ] Ali

# Process


## Payment Process for Credit Card
Basically, the user will need a `PaymentIntendId` to pay. This PaymentIntent instance will firstly be created in the server side with the *Amount* and *Currency* you want to request from customers. After the PaymentIntent is created by the Stripe PaymentIntent API, your server side will get the Id of PaymentIntent. This id can retrieve the information of PaymentIntent in the client side, and request the customer to paid for the `$amount currency` you set.

The **process** can be simplified to these steps in our app:
- User select `PaymentMethod` as *NativePay*.
- Creating `Order` instance in the frontend, and sending the request to backend for creating this `Order` in the database. At the same time, server will:
    - Calculate the total price for payment.
    - Checking in stock quantity for every orderItem. 
    - When backend detect that this payment request will be using credit card, the backend will create an `PaymentIntent` instance, which will give you a `client-secret`. At the same time, the `PaymentIntent.Id` will be saved in `Order` as well.
    - Set the `PaymentStatus` to **Unpaid**.
```csharp
if (orderEntity.PaymentMethod == Order.ESuppportPaymentMethod.CreditCard)
{
    stripe.PaymentIntentCreateOptions options = new stripe.PaymentIntentCreateOptions
    {
        Amount = (long)orderEntity.OutputTotalPrice,
        Currency = orderEntity.OutputCurrency,
    };

    stripe.PaymentIntentService service = new stripe.PaymentIntentService();
    stripe.PaymentIntent paymentIntent = service.Create(options);
    orderEntity.StripeClientSecret = paymentIntent.ClientSecret;
    orderEntity.StripeCreditCardPaymentIntentId = paymentIntent.Id;
}
```
- Send the `client-secret` from server-side to client-side.
- Using the `client-secret` to retrieve the information of **PaymentIntent** in the client-side.
- With the helps from [[stripe_payment package]](https://pub.dev/packages/stripe_payment), we can confirm the payment like this:
```dart
  Future<PaymentIntentResult> payByCreaditCard(
    BuildContext context,
    String name,
    Order order,
  ) async {
    PaymentIntentResult result = await StripePayment.confirmPaymentIntent(
      PaymentIntent(
        clientSecret: order.stripeClientSecret,
      ),
    );
    bool success =
        result.status == "succeeded" && result.paymentIntentId != null;
    if (!success) {
      throw Exception("Card Payment fail, with status ${result.status}");
    }
    return result;
  }
```
- After the payment process in doen in the client-side, call `CompletePaymentProcessForOrder` end-point in the server to set the `PaymentStatus` to **Pending**
```csharp
// In Swtich
case Order.ESuppportPaymentMethod.CreditCard:
    bool invalidCardPayment = string.IsNullOrWhiteSpace(paidInfo.CreditCardPaymentIntentId) || paidInfo.CreditCardPaymentIntentId != orderFromRepo.StripeCreditCardPaymentIntentId;
    if (invalidCardPayment) return BadRequest();
    if (orderFromRepo.PaymentStatus == Order.EPaymentStatus.Unpaid)
    {
        orderFromRepo.PaymentStatus = Order.EPaymentStatus.Pending;
    }
    break;
```
- We also have to add a webhook in the server-side to listen to the webhook when payment succeed. One we got the StripeEvent with type of success, we can set the `PaymentStatus` to **Success**
```csharp
// Handle the event
if (stripeEvent.Type == stripe.Events.PaymentIntentSucceeded)
{
    stripe.PaymentIntent paymentIntent = stripeEvent.Data.Object as stripe.PaymentIntent;

    Order orderFromRepo = await _orderRepository.GetOrderbyStripePaymentIntentId(paymentIntent.Id);

    // Do something with the order...

    await _orderRepository.Save();
    Console.WriteLine("PaymentIntent was successful!");
}
```
- To test the webhook follow the [[Official Instruction]](https://stripe.com/docs/webhooks/test). And don't forget to match your API version (Which can be set in the [[Dashboard]](https://dashboard.stripe.com/developers)) and your server-side version, or may get some errors like this: 
```
{"Received event with API version 2019-02-11, but Stripe.net 37.16.0 expects API version 2020-03-02. We recommend that you create a WebhookEndpoint with this API version. Otherwise, you can disable this exception by passing `throwOnApiVersionMismatch: false` to `Stripe.EventUtility.ParseEvent` or `Stripe.EventUtility.ConstructEvent`, but be wary that objects may be incorrectly deserialized."}
```


## Payment Process for Native Pay
The Native is easier and more nature to set up with Flutter.

- User select `PaymentMethod` as *NativePay*.
- Creating order instance in client-side.
- Sending the Order to server for approvement. The service will: 
  - Calculate the total price.
  - Checking if the item still has enought quantity in stock.
  - Set `PaymentStatus` to **Unpaid** 
- Request Native Pay to Stripe, which will return a `Token` instance.
- Send the `tokenId` to server-side and save with the `Order`. 
- Update the `PaymentStatus` to **Success** (No pending status for Native Pay).
  
```csharp
// In Swtich
case Order.ESuppportPaymentMethod.Native:
    bool invalidNativePay = string.IsNullOrWhiteSpace(paidInfo.NativePayTokenId);
    if (invalidNativePay) return BadRequest();
    orderFromRepo.StripeNativePaymentTokenId = paidInfo.NativePayTokenId;
    if (orderFromRepo.PaymentStatus == Order.EPaymentStatus.Unpaid)
    {
        orderFromRepo.PaymentStatus = Order.EPaymentStatus.Success;
    }
    break;
```

# Dependencies

## Dart Packages:
- [[stripe_payment]](https://pub.dev/packages/stripe_payment)

## NuGet:
- [[Strip.net]](https://github.com/stripe/stripe-dotnet)


# Ref
- [[Listen to WebHook]](https://stripe.com/docs/webhooks/build)
- [[Stripe Doc]](https://stripe.com/docs)
- [[Stripe API]](https://stripe.com/docs/api)
- [[Clear Elaboration]](https://medium.com/@hamza39460/stripe-payments-in-flutter-cb2f9cb053d1)
- [[Clear Elaboration II]](https://medium.com/flutter-community/build-a-marketplace-in-your-flutter-app-and-accept-payments-using-stripe-and-firebase-72f3f7228625)