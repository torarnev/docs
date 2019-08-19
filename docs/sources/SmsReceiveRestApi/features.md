<a id="markdown-contents" name="contents"></a>
# Contents
<!-- TOC depthFrom:1 insertAnchor:true -->

- [Contents](#contents)
  - [Before you begin](#before-you-begin)
  - [Scope of this document](#scope-of-this-document)
  - [Capabilities of “Common” platform](#capabilities-of-common-platform)
  - [Terms and glossary](#terms-and-glossary)
    - [MO message](#mo-message)
    - [IP addresses](#ip-addresses)
    - [Character encoding](#character-encoding)
    - [KeyValue](#keyvalue)
  - [Receiving MO messages](#receiving-mo-messages)
    - [Parameters](#parameters)
    - [Content parameters](#content-parameters)
    - [Route parameters](#route-parameters)
  - [Examples](#examples)
    - [Example sent to a keyword](#example-sent-to-a-keyword)
    - [Example sent to a subnumber](#example-sent-to-a-subnumber)
  - [Appendix 1](#appendix-1)

<!-- /TOC -->


<a id="markdown-before-you-begin" name="before-you-begin"></a>
## Before you begin

Please make sure you have provided to Link Mobility Support an URL where Common shall
deliver your messages.
Please make an opening in your firewall if necessary so that Common can connect to your
system. For a list of the addresses Common will connect from, see below.

<a id="markdown-scope-of-this-document" name="scope-of-this-document"></a>
## Scope of this document

This document will describe the Application Programming Interface (API) to receive text
messages through the Link Mobility “Common” platform. Sending messages is described in a
separate document.
Common is a REST API. A familiarity with REST APIs is assumed.
Messages will be delivered in JSON format. A basic familiarity with JSON is assumed.

<a id="markdown-capabilities-of-common-platform" name="capabilities-of-common-platform"></a>
## Capabilities of “Common” platform

Common is a high-capacity, high-availability SMS gateway designed to let you send and
receive SMS Text messages. Messages can contain any character in the UTF-8 2-byte
character set.

<a id="markdown-terms-and-glossary" name="terms-and-glossary"></a>
## Terms and glossary

<a id="markdown-mo-message" name="mo-message"></a>
### MO message

Mobile Originated message. Refers to any text message sent from an end-user’s handset to
you.

<a id="markdown-ip-addresses" name="ip-addresses"></a>
### IP addresses

When delivering a message to you, the requests can be coming from several different IP
addresses.
Appendix 1 contains the hostnames and IP addresses that are currently active.
Please configure your firewalls so that these hosts/networks can connect to your systems to
deliver messages.

<a id="markdown-character-encoding" name="character-encoding"></a>
### Character encoding

All communication to and from Common will be in UTF-8 encoding.

<a id="markdown-keyvalue" name="keyvalue"></a>
### KeyValue

Contains parameters and values in a JSON object.

Example
```json
{ "key1":"value1", "key2": "value2" }
```
<a id="markdown-receiving-mo-messages" name="receiving-mo-messages"></a>
## Receiving MO messages

When Common forwards an MO message to you, it will be POST:ed to your service, in either
XML or JSON formats. The format is described in the following table:

<a id="markdown-parameters" name="parameters"></a>
### Parameters

**Parameter**            | **Data type** | **Description** 
------------------------ | ------------- | ---------------
**destination**          | String    | The shortnumber the message was sent to. Prefixed by a two-letter country code.
**subNumber**            | Integer   | If subnumbers are used, the subnumber will be here.
**source**               | String    | The phone number of the sender, in MSISDN format (international format)
**content**              | Content   | Contains information about the content (text) of the message. See content table below.
**operator**             | String    | The telecom operator Common received the message from.
**timestamp**            | DateTime  | The timestamp when Common received the message from the operator. Formatted as RFC3339.
**messageId**            | String    | Common’s internal message ID for this message.
**operatorTimestamp**    | DateTime  | The operator-supplied timestamp for the message.
**operatorMessageId**    | String    | The operator-supplied message ID for the message.
**route**                | Route     | Information on the keyword the message has been matched to, and where the message was routed afterwards. Most fields in this structure are included for debugging purposes, with the exception of “keyword”, which contains the keyword matched. The content of route will differ depending on the type of matching used.
**gateCustomParameters** | KeyValue  | If there are custom parameters required for delivery, they will be here. Usually blank.
**customParameters** | KeyValue  | Information from the SMSC which might be mirrored information from other fields, for debugging.   


<a id="markdown-content-parameters" name="content-parameters"></a>
### Content parameters 
   
**content parameter** | **Data type** | **Description**
--------------------- | ------------- | ---------------
**type**              | String    | The type of message. Will usually be “SMS” or “MMS”.
**userData**        | String    | The content of the message sent by the end-user.
**encoding**        | String    | The encoding of the message. Will usually be “TEXT”.


<a id="markdown-route-parameters" name="route-parameters"></a>
### Route parameters 

**route parameter** | **Data type** | **Description**
------------------- | ------------- | ---------------
**type** | String | Contains information on how the message was routed to you. Contains KEYWORD_ROUTE if the message matched a keyword, or SUBNUMBER_ROUTE if the message matched a subnumber.
**id** | String | The ID of either the subnumber or the keyword matched.
**refId** | String | A reference ID for this keyword or subnumber route. This string can be set by Support if you want to use it for routing internally.
**gateIds** | <List>String | A list of the gate(s) this keyword or subnumber forwards to.
**platformId** | String | The platformId of the platform which received this message. Will usually be “SMSC”.
**platformPartnerId** | String | Your partnerId
**platformServiceType** | String | Can be set to any string by Support. Used to differentiate different types of services on your end, if needed.
**platformServiceId** | String | Can be set to any string by Support. Used to differentiate different services on your end, if needed.
**customParameters** | KeyValue|  Any custom parameters needed for this message. Usually blank.
**number** | String | The shortnumber this routing rule applies to.
**startRange** | String | Only if subnumbers are used. Contains the start of the range of subnumbers matched.
**stopRange** | String | Only if subnumbers are used. Contains the end of subnumbers matched.
**keyword** | String | Only if keyword is used. Contains the keyword that was matched.
**keywordType** | String | Contains the type of matching used for this keyword. Support will advise you on the different types of keyword types when setting up the keyword, if needed.
**active** | Boolean | Whether the keyword is active. Will always be true.
**start** | DateTime | The start date of the keyword. Will always be in the past.
**end** | DateTime | The expiry date of the keyword. Will always be in the future.
**shared** | Boolean | Whether this keyword will forward to multiple services. With the exception of STOP services, this will always be false.
**description** | String | Human-readable description of the service.

<a id="markdown-examples" name="examples"></a>
## Examples

For most purposes, you are mainly interested in the “source”, “route”:”keyword”, and
“content”:”userData” fields. Other fields are included for advanced or debugging purposes.

In these examples, some ID numbers have been replaced with 0.

<a id="markdown-example-sent-to-a-keyword" name="example-sent-to-a-keyword"></a>
### Example sent to a keyword

This example is an example of a user with phone number +4741560067 sending a message
to shortnumber 2333 with the text “Bclt hello”.

```json
{
"destination": "NO- 2333 ",
"subNumber": null,
"source": "+4741560067",
"content": {
"type": "SMS",
"userData": "Bclt hello",
"encoding": "TEXT"
},
"operator": "no.telenor",
"timestamp": "2015- 11 - 17T14:28:40Z",
"messageId": " 0 ",
"operatorTimestamp": "2015- 11 - 17T14:28:40Z",
"operatorMessageId": " 0 ",
"route": {
"type": "KEYWORD_ROUTE",
"id": " 0 ",
"refId": "Internal Keyword Reference",
"gateIds": [
"0"
],
"platformId": "0",
"platformPartnerId": "0",
"platformServiceType": "serviceType",
"platformServiceId": "Internal Service ID",
"customParameters": {},
"number": "NO- 2333 ",
"keyword": "BCLT",
"keywordType": "FIRST_WORD",
"active": true,
"start": "2015- 11 - 17T00:00:00Z",
"end": "2016- 11 - 17T00:00:00Z",
"shared": false,
"description": "BCLT service"
},
"gateCustomParameters": {},
"customParameters": {
"platformPartnerId": " 0 ",
"suggestedOperator": "no.telenor",
"moReferenceId": " 0 ",
"queued": "2015- 11 - 17 15:28:40",
"serviceCentreTimeStamp": "20151117152840",
"platformId": " 0 "
}
}
```


<a id="markdown-example-sent-to-a-subnumber" name="example-sent-to-a-subnumber"></a>
### Example sent to a subnumber

This example shows how the request will look when the message is sent to a subnumber
rather than a keyword. This example is sent from +4741560067 to shortcode 2333,
subnumber 9999999989. The text of the message is “Hello, Dolly!”

```json
{
"destination": "NO-2414",
"subNumber": 9999999989 ,
"source": "+4741560067",
"content": {
"type": "SMS",
"userData": "Hello, Dolly!",
"encoding": "TEXT"
},
"operator": "no.telenor",
"timestamp": "2015- 11 - 18T11:41:23Z",
"messageId": " 0 ",
"operatorTimestamp": "2015- 11 - 18T11:41:23Z",
"operatorMessageId": " 0 ",
"route": {
"type": "SUBNUMBER_ROUTE",
"id": " 0 ",
"refId": "SubnumberRange Definition",
"gateIds": [
"0"
],
"platformId": "0",
"platformPartnerId": "0",
"platformServiceType": "Subnumber messages",
"platformServiceId": "0",
"customParameters": {},
"number": "NO- 2333 ",
"startRange": "9999999980",
"stopRange": "9999999989"
},
"gateCustomParameters": {},
"customParameters": {
"platformPartnerId": " 0 ",
"suggestedOperator": "no.telenor",
"moReferenceId": " 0 ",
"queued": "2015- 11 - 18 12:41:23",
"serviceCentreTimeStamp": "20151118124123",
"platformId": " 0 "
}
}
```

<a id="markdown-appendix-1" name="appendix-1"></a>
## Appendix 1
The following hosts are currently used for outgoing messaging.

**Hostname(s)** | **IP address(es)**
--- | ---
socks1.sp247.net | 195.84.162.34 
socks2.sp247.net | 194.71.165.71
socks3.sp247.net | 195.84.162.16
socks4.sp247.net | 194.71.165.98
socks5.sp247.net | 195.84.162.3
socks6.sp247.net | 194.71.165.122
s1.n-eu.linkmobility.io | 213.242.87.36
s[X].n-eu.linkmobility.io | 213.242.87.37 - 54
s20.n-eu.linkmobility.io | 213.242.87.55
s1.no.linkmobility.io | 213.242.87.68
s[X].no.linkmobility.io | 213.242.87.69 - 86
s20.no.linkmobility.io | 213.242.87.87
s1.s-eu.linkmobility.io | 217.163.95.
s[X].s-eu.linkmobility.io | 217.163.95.197 - 214
s20.s-eu.linkmobility.io | 217.163.95.215
s1.deb.linkmobility.io | 62.67.62.68
s[X].deb.linkmobility.io | 62.67.62.69 - 86
s20.deb.linkmobility.io | 62.67.62.87
s1.c-eu.linkmobility.io | 62.67.62.101
s[X].c-eu.linkmobility.io | 62.67.62.102 - 119
s20.c-eu.linkmobility.io | 62.67.62.1120



