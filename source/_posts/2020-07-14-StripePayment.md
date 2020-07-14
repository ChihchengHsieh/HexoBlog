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

## Payment Methods

Two typs of the payment method we will be using in our App. 
- [x] Credit Card
- [x] Native
- [ ] WeChat
- [ ] Ali

## Process


#### Payment Process for Credit Card
![](https://miro.medium.com/max/770/1*objISxTIwmg6Yhm2A2aVgQ.png)



The **process** can be simplified to these steps in our app:
- Creating `Order` instance in the frontend, and sending the request to backend for creating this `Order` in the database.
- When backend detect that this payment request will be using credit card, the backend will create an `PaymentIntent` instance, which will give you a `client-secret`. At the same time, the `PaymentIntent.Id` will be saved in `Order` as well.
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
- We also have to add a webhook in the server-side to listen to the webhook when payment succeed.
```csharp
// Handle the event
if (stripeEvent.Type == stripe.Events.PaymentIntentSucceeded)
{
    stripe.PaymentIntent paymentIntent = stripeEvent.Data.Object as stripe.PaymentIntent;

    Order orderFromRepo = await _orderRepository.GetOrderbyStripePaymentIntentId(paymentIntent.Id);

    // Updating information
    orderFromRepo.PaidAmount = paymentIntent.Amount;
    orderFromRepo.PaidCurrency = paymentIntent.Currency;
    DateTime currentTime = DateTime.Now;
    foreach (OrderItem oi in orderFromRepo.OrderItems)
    {
        oi.PaidAt = currentTime;
        oi.OrderStatus = OrderItem.EOrderStatus.PaymentSuccess;
    }

    await _orderRepository.Save();
    Console.WriteLine("PaymentIntent was successful!");
}
```

#### Payment Process for Native Pay
Pending


## Dependencies

#### Dart Packages:
- [[stripe_payment]](https://pub.dev/packages/stripe_payment)

#### NuGet:
- [[Strip.net]](https://github.com/stripe/stripe-dotnet)


## Ref

- [[Listen to WebHook]](https://stripe.com/docs/webhooks/build)
- [[Stripe Doc]](https://stripe.com/docs)
- [[Stripe API]](https://stripe.com/docs/api)
- [[Clear Elaboration]](https://medium.com/@hamza39460/stripe-payments-in-flutter-cb2f9cb053d1)
- [[Clear Elaboration II]](https://medium.com/flutter-community/build-a-marketplace-in-your-flutter-app-and-accept-payments-using-stripe-and-firebase-72f3f7228625)