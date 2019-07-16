# Getting started

```
Overview

Here some information to help get you started using the LINK Payment API.

```

LINK Mobility's payment integration platform API consists of a set of functions that you can use to:

* Request payment from a Mobile user and allow the user to choose payment method (Phone/SMS or Card).
* Define your own URL's for user navigation and landingpages.
* Initiate transactions directly when you already know the payment method.
* See transaction status and billing information.
* Define your own callback URL's for receiving status information.
  
Before you can start using the API, you will receive 2 pieces of information that will be provided by LINK Mobility:

1. A Partner identification number.
2. A secret key. (**Important: This key is highly confidential and you must keep it secure at all times!**)

If you want to accept Card payment, you will also need a Merchant account (see below).

## Typical Flow of a Credit Card Payment
![Flow](https://raw.githubusercontent.com/torarnev/docs/master/docs/samples/images/start1.png)