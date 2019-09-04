# Card Payment Integration

Integration process of Card Fullpayment (3DS) will be explained below.

## Overview
Overview of the transaction flow in sequence diagram:

![card transaction flow](./asset/image/card_transaction_3ds.png)
.

## Integration Step
1. Get card token, via frontend.
2. Send transaction data to API Charge using card token, via backend.
3. (Conditional) if transaction is 3DS, open 3DS redirect_url with popup/redirect, via frontend.
4. Handle notification, on backend.

## 1. Get Card Token
Card `token_id` is representation of customer's card data, that will be used during a transaction. `token_id` should be retrieved using [MidtransNew3ds JS library](https://api.midtrans.com/v2/assets/js/midtrans-new-3ds.min.js) on merchant's website frontend, card data will be securely transmitted by frontend javascript to Midtrans API in exchange of card `token_id`, to avoid risk involved if card data being transmitted to merchant's backend.

> **Note:** This `token_id` is only valid for 1 transaction. For each time card transaction, this step should be re-performed. To persist/save card token, please use **One-click/Two-click** feature.


### Include Midtrans JS
First, include Midtrans JS library to our payment page, by adding this script tag:
```
<script id="midtrans-script" type="text/javascript"
src="https://api.midtrans.com/v2/assets/js/midtrans-new-3ds.min.js" 
data-environment="sandbox" 
data-client-key="<INSERT YOUR CLIENT KEY HERE>"></script>
```

**Important**: Change the following attributes.

| Attribute | Value |
|-----------|-------|
| `data-environment`| Input `sandbox` or `production` (API environment)|
| `data-client-key`| Input **client key** from your account's [Dashboard](https://account.midtrans.com) |

Link: [*More detailed definition*](https://api-docs.midtrans.com/#get-token)

### Get Card Token JS Implementation
To retrieve card `token_id`, we will be using `MidtransNew3ds.getCardToken` function. Implement the following Javascript on our payment page.

```javascript
// card data
var cardData = {
  "card_number": 4811111111111114,
  "card_exp_month": 02,
  "card_exp_year": 2025,
  "card_cvv": 123,
};

// callback functions
var options = {
  onSuccess: function(response){
    // Success to get card token_id, implement as you wish here
    console.log('Success to get card token_id, response:', response);
    var token_id = response.token_id;
    console.log('This is the card token_id:', token_id);
  },
  onFailure: function(response){
    // Fail to get card token_id, implement as you wish here
    console.log('Success to get card token_id, response:', response);
  }
};

// trigger `getCardToken` function
MidtransNew3ds.getCardToken(cardData, options);
```

If all goes well, we will be able to get card `token_id` inside `onSuccess` callback function. It will be used as one of JSON parameter for `/charge` API request.

Note: `token_id` will need to be passed from frontend to backend for next step, it can be done using AJAX via Javascript, or html form POST, etc. Merchant are free to implement.

## 2. Send Transaction Data to API Charge

Charge API request should be done from Merchant's backend. **Server Key** (from your account's [Dashboard](https://account.midtrans.com)) will be needed to [authenticate the request](https://api-docs.midtrans.com/#http-s-header).

### Charge API request
This is example of `/charge` API request in Curl, please implement according to your backend language (you can also check our available [language libraries](http://docs.midtrans.com/en/welcome/pluginlibrary.html)). Input `token_id` retrieved previously.

```bash
# sample charge in CURL
curl -X POST \
  https://api.sandbox.midtrans.com/v2/charge \
  -H 'Accept: application/json' \
  -H 'Authorization: Basic <YOUR SERVER KEY ENCODED in Base64>' \
  -H 'Content-Type: application/json' \
  -d '{
  "payment_type": "credit_card",
  "transaction_details": {
    "order_id": "order102",
    "gross_amount": 789000
  },
  "credit_card": {
    "token_id": "<token_id from Get Card Token Step>",
    "authentication": true,
  }
}'
```

Optional: we can customize `credit_card` object for other [advanced features](https://api-docs.midtrans.com/#credit-card). For example:
```bash
...
  "credit_card": {
    "token_id": "<token_id from Get Card Token Step>",
    "authentication": true, //set true for 3DS, false for non 3DS
    "save_token_id": true //optional, to use one/two click feature
    "installment_term": 12, //optional, to allow installment
    "bank": "bca", //optional, to specify acquiring bank
  }
...
```

Optional: we can customize [transaction_details data](https://api-docs.midtrans.com/#json-object). To include data like customer_details, item_details, etc. It's recommended to send as much detail so on report/dashboard those information will be included.

### Charge API response
We will get the **API response** like the following.
```javascript
{
  "status_code": "201",
  "status_message": "Success, Credit Card transaction is successful",
  "transaction_id": "0bb563a9-ebea-41f7-ae9f-d99ec5f9700a",
  "order_id": "order102",
  "redirect_url": "https://api.sandbox.veritrans.co.id/v2/token/rba/redirect/481111-1114-0bb563a9-ebea-41f7-ae9f-d99ec5f9700a",
  "gross_amount": "789000.00",
  "currency": "IDR",
  "payment_type": "credit_card",
  "transaction_time": "2019-08-27 15:50:54",
  "transaction_status": "pending",
  "fraud_status": "accept",
  "masked_card": "481111-1114",
  "bank": "bni",
  "card_type": "credit"
}
```

If the `transaction_status` is `capture` and `fraud_status` is `accept`, it means the transaction is non 3DS, success, and is now complete.

If the `transaction_status` is `pending` and `redirect_url` exists, it means the transaction is 3DS, and we will need to proceed to next step, opening 3DS authentication page.

## 3. Open 3DS Authentication Page

As part of API response, we now have `redirect_url`. It should be opened (displayed to customer) using [MidtransNew3ds JS library](https://api.midtrans.com/v2/assets/js/midtrans-new-3ds.min.js) on merchant's website frontend.

To open 3DS page we can use `MidtransNew3ds.authenticate` or `MidtransNew3ds.redirect` function. Input `redirect_url` retrieved previously.

### Open 3DS Authenticate Page JS Implementation
```javascript
var redirect_url = '<redirect_url Retrieved from Charge Response>';

// callback functions
var options = {
  performAuthentication: function(redirect_url){
    // Implement how you will open iframe to display 3ds authentication redirect_url to customer
    popupModal.openPopup(redirect_url);
  },
  onSuccess: function(response){
    // 3ds authentication success, implement payment success scenario
    console.log('response:',response);
    popupModal.closePopup();
  },
  onFailure: function(response){
    // 3ds authentication failure, implement payment failure scenario
    console.log('response:',response);
    popupModal.closePopup();
  },
  onPending: function(response){
    // transaction is pending, transaction result will be notified later via POST notification, implement as you wish here
    console.log('response:',response);
    popupModal.closePopup();
  }
};

// trigger `authenticate` function
MidtransNew3ds.authenticate(redirect_url, options);



/**
 * Example helper functions to open Iframe popup, you may replace this with your own method to open iframe
 * PicoModal library is used:
 * <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/picomodal/3.0.0/picoModal.js"></script>
 */
var popupModal = (function(){
  var modal = null;
  return {
    openPopup(url){
      modal = picoModal({
        content:'<iframe frameborder="0" width="100%" height="100%" src="'+url+'"></iframe>',
        width: "75%", 
        closeButton: false, 
        overlayClose: false,
        escCloses: false
      }).show();
    },
    closePopup(){
      try{
        modal.close();
      } catch(e) {}
    }
  }
}());

/**
 * Alternatively instead of opening 3ds authentication redirect_url using iframe,
 * you can also redirect customer using: 
 * MidtransNew3ds.redirect(redirect_url, { callbackUrl : 'https://mywebsite.com/finish_3ds' });
 **/
```

### 3DS Authenticate JSON Response
On the JS callback function, we will get JS object (JSON) of the transaction result like below.

```javascript
{
  "status_code": "200",
  "status_message": "Success, Credit Card transaction is successful",
  "transaction_id": "226ba26f-b050-4fc5-aa25-b7f8169bc67b",
  "order_id": "order102",
  "gross_amount": "789000.00",
  "currency": "IDR",
  "payment_type": "credit_card",
  "transaction_time": "2019-08-27 17:22:08",
  "transaction_status": "capture",
  "fraud_status": "accept",
  "approval_code": "1566901334936",
  "eci": "05",
  "masked_card": "481111-1114",
  "bank": "bni",
  "card_type": "credit",
  "channel_response_code": "00",
  "channel_response_message": "Approved"
}
```

If the `transaction_status` is `capture` and `fraud_status` is `accept`, it means the transaction is success, and is now complete.

> **IMPORTANT NOTE:** To update transaction status on your backend/database, DO NOT solely rely on frontend callbacks! For security reason to make sure the status is authentically coming from Midtrans, only update transaction status based on HTTP Notification or [API Get Status](https://api-docs.midtrans.com/#get-transaction-status).

## 4. Handle HTTP Notification

HTTP notification from Midtrans to Merchant backend will also be triggered on event of `transaction_status` getting updated, to ensure merchant is securely informed. Including if card transaction success or denied. So apart of JSON result above, Merchant backend will be notified by Midtrans.

HTTP POST request with JSON body will be sent to Merchant's **notification url** configured on [dashboard](https://account.midtrans.com) (Settings > Configuration > Notification URL), this is the sample JSON body that will be received by Merchant:

```javascript
{
  "transaction_time": "2019-08-27 17:22:08",
  "transaction_status": "capture",
  "transaction_id": "226ba26f-b050-4fc5-aa25-b7f8169bc67b",
  "status_message": "midtrans payment notification",
  "status_code": "200",
  "signature_key": "9b67a07a82d592bc493f3b775eaae5b1862a8bbaa051cdd31a0a81566c84f2c4d306fe2712c86f8167c4faa594a0e2cbe6ae898491bfaebaf849681ae92264d5",
  "payment_type": "credit_card",
  "order_id": "order102",
  "masked_card": "481111-1114",
  "gross_amount": "789000.00",
  "fraud_status": "accept",
  "eci": "05",
  "currency": "IDR",
  "channel_response_message": "Approved",
  "channel_response_code": "00",
  "card_type": "credit",
  "bank": "bni",
  "approval_code": "1566901334936"
}
```

Refer [here on more details of how to handle HTTP Notification](https://api-docs.midtrans.com/#handing-notifications).

## Finish!

The card payment integration guide is now complete. Below are some further references.

## Description

`transaction_status` value description:

| Transaction Status | Description |
| ------------------ | ----------- |
| `capture` | Transaction successful, fund has been deducted |
| `pending` | Transaction is waiting for further action (3DS) |
| `deny` | Transaction is denied, further check `channel_response_message` or `fraud_status` |

Link: [*More detailed definition of transaction_status*](https://api-docs.midtrans.com/#transaction-status)

Link: [*More detailed definition of fraud_status*](https://api-docs.midtrans.com/#fraud-status)

#### Reference

You can also refer to this sample implementation:
- https://github.com/Midtrans/midtrans-nodejs-client/blob/master/examples/expressApp/views/simple_core_api_checkout.ejs
- https://github.com/Midtrans/midtrans-python-client/blob/master/examples/flask_app/templates/simple_core_api_checkout.html

[Or please refer to this demo](https://anice.win/3ds_new/)
- [Which source code is available here](https://gist.github.com/rizdaprasetya/9d16893578d600a03075939ef74c5c1f)

[Complete Core API documentation](https://api-docs.midtrans.com/)