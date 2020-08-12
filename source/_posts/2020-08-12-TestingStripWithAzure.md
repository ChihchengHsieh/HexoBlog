---
title: Testing the stripe with Azure server side.
tags:
    - .NET
    - Flutter
categories: App Development 
photos:
    - https://images.ctfassets.net/fzn2n1nzq965/3AGidihOJl4nH9D1vDjM84/9540155d584be52fc54c443b6efa4ae6/homepage.png?fm=jpg
---

# Setting up the WebHook in the Stripe console.

You have to set up your end point in the stripe console. Since we're using the PaymentIntent for requesting the payment, we require all the activities related to the payment intent to send a notification to the webhook.


![](https://trello.com/1/cards/5f338a1c53a3ee542c3c68bd/attachments/5f33b9daad5fd74a428e5aa3/previews/5f33b9daad5fd74a428e5aaf/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyMjI4MDAsImV4cCI6MTU5NzIyODIwMCwicmVzIjoiNWYzMzhhMWM1M2EzZWU1NDJjM2M2OGJkOjVmMzNiOWRhYWQ1ZmQ3NGE0MjhlNWFhMzo1ZjMzYjlkYWFkNWZkNzRhNDI4ZTVhYWYiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.PPzi3gB9y5acksfErJBy9IRVtcw6z8X6xzMKnSNfTC0)


![](https://trello.com/1/cards/5f338a1c53a3ee542c3c68bd/attachments/5f33ba4bc2309688fb2aa523/previews/5f33ba4cc2309688fb2aa59c/download?signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTcyMjI4MDAsImV4cCI6MTU5NzIyODIwMCwicmVzIjoiNWYzMzhhMWM1M2EzZWU1NDJjM2M2OGJkOjVmMzNiYTRiYzIzMDk2ODhmYjJhYTUyMzo1ZjMzYmE0Y2MyMzA5Njg4ZmIyYWE1OWMiLCJhdWQiOiJUcmVsbG8iLCJpc3MiOiJUcmVsbG8ifQ.-LkEPaVqr4Mql44xKSrMTuOk0KUxbxSS51hs03r75og)

