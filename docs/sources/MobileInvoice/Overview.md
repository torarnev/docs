**Mobile Invoice Api - Overview**

<!-- TOC depthFrom:1 insertAnchor:true -->

- [Common user stories](#common-user-stories)
  - [Send invoice by sms to users.](#send-invoice-by-sms-to-users)
  - [Request payment from a Mobile user](#request-payment-from-a-mobile-user)
  - [Initiate transactions directly](#initiate-transactions-directly)
  - [See transaction status and billing information.](#see-transaction-status-and-billing-information)
  - [Define your own callback URL's](#define-your-own-callback-urls)
  - [Define your own URL's](#define-your-own-urls)
- [Typical Flow of a Credit Card Payment](#typical-flow-of-a-credit-card-payment)

<!-- /TOC -->



<a id="markdown-common-user-stories" name="common-user-stories"></a>
# Common user stories

<a id="markdown-send-invoice-by-sms-to-users" name="send-invoice-by-sms-to-users"></a>
## Send invoice by sms to users.

<a id="markdown-request-payment-from-a-mobile-user" name="request-payment-from-a-mobile-user"></a>
## Request payment from a Mobile user
* Allow the user to choose payment method (Phone/SMS or Card).

<a id="markdown-initiate-transactions-directly" name="initiate-transactions-directly"></a>
## Initiate transactions directly 
* when you already know the payment method.

<a id="markdown-see-transaction-status-and-billing-information" name="see-transaction-status-and-billing-information"></a>
## See transaction status and billing information.

<a id="markdown-define-your-own-callback-urls" name="define-your-own-callback-urls"></a>
## Define your own callback URL's 
* for receiving status information.

<a id="markdown-define-your-own-urls" name="define-your-own-urls"></a>
## Define your own URL's 
* for user navigation and landingpages.

<a id="markdown-typical-flow-of-a-credit-card-payment" name="typical-flow-of-a-credit-card-payment"></a>
# Typical Flow of a Credit Card Payment

* Invoice is sent to the API
* Sms is sent to user (inclusing link to landing page)
* User opens url (navigates to landing page)
* User selects paymentmethod (from included in the invoice)
* User is redirected to PSP site for payment.
* Payment done - user is redirected to receipt page.

![Typical credit card payment flow](images/start1.png)