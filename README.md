# TelyPay Payment API Documentation V1
## Introduction
Welcome to TelyPay's Payment API documentation. This guide is intended for developers who are looking to integrate TelyPay's payment gateway into their applications. TelyPay offers a straightforward, robust, and secure method for processing transactions.

The API base URL is: https://payment.telypay.com

## Pre-requisites
Before you start integration, you need to have your API key and Merchant ID which is available once you register as a business at https://invoice.telypay.com. After registration, you can copy your API key and Merchant ID from the developer menu.

## EndPoints
We are providing different endpoints to communicate with our payment gateway.
|  | Endpoint | URL |
| --------------- | ----------------- | ----------------- |
| 1  | Payment Request | __`POST:`__ `/v1/payment/request` |
| 2  | Payment Refund | __`POST:`__ `/v1/refund/request` |
| 3  | Get Transaction by Id | __`GET:`__ `/v1/transaction` |
| 4  | Get Transaction List | __`GET:`__ `/v1/transaction/list` |

### 1. Payment Request
To initiate a payment request, you need to call the endpoint below. If the request initiates successfully, in the response we will provide `redirectUrl` to which you need to redirect the user to proceed to the transaction, where the user will provide the card information. When the user completes the transaction, we will POST you the response of the transaction on your provided `callback` URL automatically.

##### URL
__`POST:`__ `/v1/payment/request`

##### Parameters

| Parameter | Type | Description | Sample Value |
| --------------- | ----------------- | ----------------- | ----------------- |
| ApiKey | header | Registered business API key | TP-SK799J6DX62095B4827F6D96F0C3S9D4 | 
| Content-Type | header | Content type | `application/json` |
| merchantId | body | Registered business Merchant Id | 999366903 |
| orderId | body | Unique order id | 2254857641 |
| orderDescription | body | Order description | - |
| amount | body | Order amount in OMR | 2.456 |
| callback | body | Callback URL where we will POST transaction information | https://xyzCompany.com/callback |


#### Sample Request (curl)
```
curl --location --globoff '<<API base URL>>/payment/request' \
--header 'ApiKey: XXXXXXXXXXXXXXXXXXXXXX' \
--header 'Content-Type: application/json' \
--data '{    
  "merchantId": XXXXXXXXXX,
  "orderId":"your unique order ref",
  "orderDescription":"This is a description",
  "amount": 1.5,
  "callback": "https://yourcompany.com/callback"
}'
```

#### Sample Request (C#)

If you are integrating in C#, You need to install __RestSharp__ nuget package in your project.

```
string _body = @"{" 
+ @"    ""merchantId"": XXXXXXXXXX,
+ @"    ""orderId"": ""12345678"",
+ @"    ""orderDescription"": ""This is a description"",
+ @"    ""amount"": 2.55,
+ @"    ""callback"": ""https://yourcompany.com/callback""
+ @"}";

var options = new RestClientOptions("<<API base URL>>")
{
  MaxTimeout = -1,
};
var client = new RestClient(options);
var request = new RestRequest("/v1/payment/request", Method.Post);
request.AddHeader("ApiKey", "XXXXXXXXXXXXXXXXXXXXXX");
request.AddHeader("Content-Type", "application/json");
var body = _body;
request.AddStringBody(body, DataFormat.Json);
RestResponse response = await client.ExecuteAsync(request);
Console.WriteLine(response.Content);
```


#### Payment Request Response
```
{
    "message": "Success",
    "result": 200,
    "body": {
        "tranRef": "TST2313501591268",
        "orderId": "Your unique order ref",
        "orderDescription": "Ths is a description",
        "amount": "1.500",
        "callback": "https://yourcompany.com/callback",
        "redirectUrl": "https://secure-oman.paytabs.com/payment/wr/5E99A88F82E4116AC8D9E3762BBB199033920195EC0083435DAA8123",
        "trace": "PMNT0505.646A6D95.000036QQ"
    }
}
```

Once you get the response, you need to redirect the user on `redirectUrl` as received in response, which is actually the payment page, once redirected, we will __POST__ the transaction response (model as below) to your `callback` URL. Your call-back URL must be a POST URL.

It is important to save the `tranRef` value received in the response for future reference. The `tranRef` serves as a unique identifier for the transaction and will be required in case a refund needs to be processed.  

#### Transaction Response
```
{
    "BusinessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419"
    "TranRef": "TST2313501591268",
    "TranType": "Sale",
    "OrderId": "your unique order ref",
    "Description": "This is a description",
    "Currency": "OMR",
    "Amount": "1.5",
    "Callback": "https://yourcompany.com/callback"
    "Trace": "PMNT0505.646A6D95.000036QQ",
    "Date": "2023-05-21T14:30:00Z",
    "PaymentStatus": "A",
    "PaymentMessage": "Authorised",
    "PrevTranRef": null
}
```

All possible `PaymentStatus` are explained at the end of the document.


#### Error Response Model
```
{
    "message": "Duplicated Request",
    "result": 409,
    "body": null
}
```

### 2. Refund Request
To initiate a refund, API details are below. We will create a new transaction with the type "Refund"
and with the same model of the transaction, a response will be sent back.

##### URL
__`POST:`__ `/v1/refund/request`

##### Parameters

| Parameter | Type | Description | Sample Value |
| --------------- | ----------------- | ----------------- | ----------------- |
| ApiKey | header | Registered business API key | TP-SK799J6DX62095B4827F6D96F0C3S9D4 | 
| Content-Type | header | Content type | `application/json` |
| merchantId | body | Registered business Merchant Id | 999366903 |
| transRef | body | Unique transaction reference which need to refund | TST2313501591268 |
| amount | body | Order amount in OMR | 2.456 |


#### Sample Request (curl)
```
curl --location --globoff '<<API base URL>>/refund/request' \
--header 'ApiKey: XXXXXXXXXXXXXXXXXXXXXX' \
--header 'Content-Type: application/json' \
--data '{
    "merchantId": XXXXXXXXXX,
    "transRef":"TST2313501591268",
    "amount": 1
}'
```

#### Sample Request (C#)

If you are integrating with C#, you need to install __RestSharp__ nuget package in your project.
```
string _body = @"{" 
+ @"    ""merchantId"": XXXXXXXXXX,
+ @"    ""transRef"": ""TST2314101600392"",
+ @"    ""amount"": 2.55
+ @"}";

var options = new RestClientOptions("<<API base URL>>")
{
  MaxTimeout = -1,
};
var client = new RestClient(options);
var request = new RestRequest("/v1/Refund/request", Method.Post);
request.AddHeader("ApiKey", "XXXXXXXXXXXXXXXXXXXXXX");
request.AddHeader("Content-Type", "application/json");
var body = _body
request.AddStringBody(body, DataFormat.Json);
RestResponse response = await client.ExecuteAsync(request);
Console.WriteLine(response.Content);
```

#### Sample Response
```
{
    "message": "Success",
    "result": 200,
    "body": {
        "BusinessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419"
        "TranRef": "TST2314101600398",
        "TranType": "Refund",
        "OrderId": "22435634534",
        "Description": "This is a description",
        "Currency": "OMR",
        "Amount": "1.5",
        "Callback": "https://yourcompany.com/callback"
        "Trace": "PMNT0506.646A734C.000036D4",
        "Date": "2023-05-21T16:45:00Z",
        "PaymentStatus": "A",
        "PaymentMessage": "Authorised",
        "PrevTranRef": "TST2313501591268"
    }
}
```

#### Error Response Model
```
{
    "message": "Unable to process your request",
    "result": 422,
    "body": null
}
```

### 3. Get Transaction
To get transaction by reference Id, you can use below API endpoint.

##### URL
__`GET:`__ `/v1/transaction`

##### Parameters

| Parameter | Type | Description | Sample Value |
| --------------- | ----------------- | ----------------- | ----------------- |
| ApiKey | header | Registered business API key | SK799J6DX62095B4827F6D96F0C3S9D4 | 
| Content-Type | header | Content type | `application/json` |
| merchantId | query string | Registered business Merchant Id | 999366903 |
| transRef | query string | Unique transaction reference | TST2313501591268 |

#### Sample Request (curl)
```
curl --location '<<API base URL>>/v1/transaction?tranRef=TST2313501591268&merchantId=999366903' \
--header 'ApiKey: XXXXXXXXXXXXXXXXXXXXXX'
```

#### Sample Request (C#)

If you are integrating with C#, you need to install __RestSharp__ nuget package in your project.
```
var options = new RestClientOptions("<<API base URL>>")
{
  MaxTimeout = -1,
};
var client = new RestClient(options);
var request = new RestRequest("/v1/transaction?tranRef=TST2313501591268&merchantId=999366903", Method.Get);
request.AddHeader("ApiKey", "XXXXXXXXXXXXXXXXXXXXXX");
RestResponse response = await client.ExecuteAsync(request);
Console.WriteLine(response.Content);
```

#### Sample Response
```
{
    "message": "Success",
    "result": 200,
    "body": {
        "BusinessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419"
        "TranRef": "TST2313501591268",
        "TranType": "Sale",
        "OrderId": "22435634534",
        "Description": "This is a description",
        "Currency": "OMR",
        "Amount": "1.5",
        "Callback": "https://yourcompany.com/callback"
        "Trace": "PMNT0506.646A734C.000036D4",
        "Date": "2023-05-21T16:45:00Z",
        "PaymentStatus": "A",
        "PaymentMessage": "Authorised",
        "PrevTranRef": null
    }
}
```

#### Error Response Model
```
{
    "message": "Unauthorised",
    "result": 401,
    "body": null
}
```

### 4. Get Transaction List
To get multiple transactions, you can use below API endpoint.

##### URL
__`GET:`__ `/v1/transaction/list`

##### Parameters

| Parameter | Type | Description | Sample Value |
| --------------- | ----------------- | ----------------- | ----------------- |
| ApiKey | header | Registered business API key | SK799J6DX62095B4827F6D96F0C3S9D4 | 
| Content-Type | header | Content type | `application/json` |
| merchantId | query string | Registered business Merchant Id | 999366903 |
| type | query string | Transaction type i.e., Sale, Refund. You can provide both comma separated | `sale,refund` |
| MaxSize | query string | Maximum records in one response page | 50 |
| MinSize | query string | Minimum records in one response page | 2 |
| MinPage | query string | Page number to be fetched | 1 |

#### Sample Request (curl)
```
curl --location '<<API base URL>>/v1/transaction/list?tranRef=TST2313501591268&merchantId=999366903&type=sale&maxsize=3&minsize=1&minpage=1' \
--header 'ApiKey: XXXXXXXXXXXXXXXXXXXXXX'
```

#### Sample Request (C#)

If you are integrating with C#, you need to install __RestSharp__ nuget package in your project.
```
var options = new RestClientOptions("<<API base URL>>")
{
  MaxTimeout = -1,
};
var client = new RestClient(options);
var request = new RestRequest("/v1/transaction/list?tranRef=TST2313501591268&merchantId=999366903&type=sale,refund&maxsize=3&minsize=1&minpage=1", Method.Get);
request.AddHeader("ApiKey", "XXXXXXXXXXXXXXXXXXXXXX");
RestResponse response = await client.ExecuteAsync(request);
Console.WriteLine(response.Content);
```

#### Sample Response
```
{
    "message": "Success",
    "result": 200,
    "body": {
        "totalCount": 15,
        "data": [
            {
                "tranRef": "TST2316201626708",
                "tranType": "Sale",
                "orderId": "12345678",
                "businessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419",
                "description": "This is a description",
                "currency": "OMR",
                "amount": "15.000",
                "callback": "https://yourcompany.com/callback",
                "trace": "PMNT0205.142664B5.00005BC8",
                "date": "2023-06-11T18:38:45.5306171",
                "paymentStatus": "D",
                "paymentMessage": "Card security code (CVV) mismatch",
                "prevTranRef": null
            },
            {
                "tranRef": "TST2316201626677",
                "tranType": "Sale",
                "orderId": "17945689",
                "businessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419",
                "description": "This is a description",
                "currency": "OMR",
                "amount": "2.456",
                "callback": "https://yourcompany.com/callback",
                "trace": "PMKT0506.6482FA2F.0010595C",
                "date": "2023-06-11T16:45:35.4552097",
                "paymentStatus": "A",
                "paymentMessage": "Authorised",
                "prevTranRef": null
            },
            {
                "tranRef": "TST2313501591268",
                "tranType": "Refund",
                "orderId": "0000",
                "businessId": "626B3C9E-298C-4E2D-A8A5-CE39542CD419",
                "description": "This is a description",
                "currency": "OMR",
                "amount": "1",
                "callback": null,
                "trace": "PMNT0106.647CF265.000032DC",
                "date": "2023-06-04T20:47:01.6812138",
                "paymentStatus": "E",
                "paymentMessage": "Amount greater than available balance",
                "prevTranRef": "TST2315201613740"
            }
        ]
    }
}
```


#### Error Response Model
```
{
    "message": "Unauthorised",
    "result": 401,
    "body": null
}
```


## Payment Staus
 Below are the possible statuses for any transaction i.e., `Request`, `Refund`

| PaymentStatus | Description |
| --------------| -------------- |
| A | Authorised |
| H | Hold (Authorised but on hold for further anti-fraud review) |
| P | Pending (for refunds) |
| V | Voided |
| E | Error |
| D | Declined |
| X | Expired |

## Test Card
We are providing a test VISA Card as below to test the integration with TelyPay payment gateway.

| Card Number | CVV | Month | Year |
| --------------- | --------------- | --------------- | --------------- |
| 4111 1111 1111 1111 | 123 | Any | Any |


## Contact Us
For any inquiries or assistance regarding our API, we are dedicated to providing prompt and reliable support. Our team of experts is readily available to address your questions, offer technical guidance, and help troubleshoot any issues you may encounter. Please feel free to reach out to our dedicated support contact via email at techsupport@telypay.com or phone at 0096893614406. We value your feedback and are committed to ensuring a seamless and successful integration of our API into your applications. Your satisfaction is our priority, and we strive to deliver exceptional support to empower your development journey.


Happy Coding :)
