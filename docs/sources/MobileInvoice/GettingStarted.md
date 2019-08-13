**Mobile Invoice Api - Getting started**

<!-- TOC depthFrom:1 insertAnchor:true -->

- [Get started](#get-started)
- [Using the api](#using-the-api)
- [Creating hmac](#creating-hmac)

<!-- /TOC -->
<a id="markdown-get-started" name="get-started"></a>
# Get started

Before you can start using the API, you will receive 2 pieces of information that will be provided by LINK Mobility:

1. A Partner identification number.
2. A secret key. (***Important:*** _This key is highly confidential and you must keep it secure at all times!_) 

Also you have to have a merchant account for your chosen PSPs. 
* MerchantId: Is required to use a PSP, but note that some of the PSPs require a couple of extra properties.

The LINK Payment system is integrated with several payment gateways, and is continually expanding your options. Currently you can choose between the following:

* Credit/Debet card payments.
* Mobile phone subscription billing via premium SMS

<a id="markdown-using-the-api" name="using-the-api"></a>
# Using the api
> **HTTPS only**  
The Payment API allows HTTPS only, and is tested for security threats annualy.  

API urls

| Environment | BaseUrl |
| ----------- | ------- |
| Production | "https://pay-core.linkmobility.com" |
| Test | "https://test-pay-core.linkmobility.com" |

Generate a HMAC authentication header (See Creating hmac),  
including the HMAC signature in the request.  
The response from LINK Mobility is returned in JSON or XML format.
  
Integrating with the api is easiest using our [nuget](https://www.nuget.org/packages/LinkMobility.PaymentCore.Sdk/) package (which unfortunately is only available in C#).

Otherwise the HMAC header needs to be created, and included in any requests made.


<a id="markdown-creating-hmac" name="creating-hmac"></a>
# Creating hmac
[HMAC (Hashed based Message Authentication Code)](CreatingHmac.md)



What is a merchant account?

What is a Payment Gateway?

