## FAQ General Topic

### Which Credit Card acquiring bank that need BCA MIGS Pilot testing?
BCA, Maybank, BRI. Full Payment and Installment

### Got `Terminal or MID not found` message from log and all card transaction fail, what to do?
MID-TID might not be properly onboarded yet (not yet sync), please ask onboarding team to sync or properly onboard.

### Why Snap Popup doesn’t work on merchant mobile app?
- If merchant mobile app use webview to display snap popup, make sure the app configuration follow these points:
Enable Javascript capability for the webview.
Allow webview to open bank web domain (for reasons that will be explained below).
- If testing on sandbox, allow webview to open Midtrans simulator domain: https://simulator.midtrans.com 
Those configs may be found on app config/manifest. Or specific code when calling webview.

**Reasoning :**

Because a lot of payment methods within snap will redirect customer to bank website, merchant mobile dev need to make sure the app allow webview to open bank website domains. Which might mean whitelisting any domain to be opened, because in case of credit card, customers can use any issuing bank credit card, which the web domain can be anything.

### Can Merchant revoke or cancel an active Snap Token to prevent customer proceed the transaction?
There's no official revoke or cancel API method for Snap token.

However there's some work around: 
- If you know how long you will need an Snap token to active, you can set via `expiry` JSON parameter. Make sure to set `start_time`. Or
- Do another `createTransaction()` with **same order ID** to obtain another token, it will revoke the old token, only the latest 1 should active.

If you just need to close payment popup on customer browser, using frontend javascript you can call `snap.hide()` function to close the payment popup.


### Merchant want to use Bin API ( /v1/bins/bin_number), but getting "Invalid authentication credentials"?
In order to use this API, need to manually request Turbo team to whitelist the specific merchant. Provide Merchant ID & environment.

note: On #pilar Slack channel, search for "Please consult with gaia team regarding the inquiry"

### Merchant want to use Offline Installment payment, how they should implement it?
It’s recommended to have separate payment flow (can be separate payment button, etc) for Offline Installment.

For offline installment merchant have to use `whitelist_bins`, customer can only pay with the whitelisted card only. Hence customer cannot use it to pay using other card for full payment or other banks that are not in whitelist_bins.

It doesn’t have to be different `button`, UX wise merchant can make it as checkboxes “I want to pay with <Offline Installment bank> Installment” for customer to check, etc. As long as it can give merchant backend information that your customers want to pay with offline installment, and merchant backend send the `whitelist_bins` accordingly.

### Is it possible to identify card issuer or brand based on the card number?
Yes, it is. The first 6-digit of the card is called: Bank Identification Number (BIN). e.g: `410505` belong to BNI Visa card, `477377` belong to BCA Visa card, etc.

If you need to identify card issuer/brand for promo purpose, it is advisable to request list of BIN from the Bank that want to do promo with you, or usually they will provide the list of BINs when they offer to do join promotion. Because they will have the most accurate list of BIN for the cards they issued.

Else, if you have no choice but to do it yourself, Midtrans offers BIN API:
https://api-docs.midtrans.com/#bin-api
Which constructed based on BINs provided by Midtrans' partner banks. It might not be 100% accurate for every card available out there. So please proceed with caution.

### Merchant want to use credit card BIN based promo campaign, how they should implement it? 
It’s recommended to have separate payment flow (can be separate payment button, etc) for BIN specific promo.

For BIN specific promo merchant have to use `whitelist_bins`, customer can only pay with the whitelisted card only. Hence customer cannot use it to pay using other card for full payment or other banks that are not in whitelist_bins.

It doesn’t have to be different `button`, UX wise merchant can make it as checkboxes “I want to pay with <BIN specific promo>” for customer to check, etc. As long as it can give merchant backend information that your customers want to pay with BIN specific promo, and merchant backend send the `whitelist_bins` accordingly.

### Merchant want to use payment method specific campaign, how they should implement it?
It’s recommended to have separate payment flow (can be separate payment button, etc) for payment method specific promo.

For payment method specific promo merchant have to use `enabled_payments`, customers can only pay with the specific payment method only. Hence customer cannot use it to pay using other payment method.

It doesn’t have to be different `button`, UX wise merchant can make it as checkboxes “I want to pay with < payment method >” for customer to check, etc. As long as it can give merchant backend information that your customers want to pay with specific payment method, and merchant backend send the `enabled_payments` accordingly.

### Why Merchant got alert that their HTTP notification is failure because of 3xx status code or redirect?

See below.

### Why Midtrans HTTP notification looks "empty" when received on merchant backend / notification handler?
It can be caused by `notification_url` (set by merchant on Dashboard) is ending up in HTTP redirect, when HTTP notification sent by Midtrans notification engine. If HTTP redirect happens, it can cause HTTP POST call to be redirected as HTTP GET, which means it will no longer contains HTTP body which contains the transaction data. Resulting in merchant notification handler getting empty request body, and might throw error. Redirect can be caused by network/reverse proxy/web framework used by merchant.

**To resolve**:

Make sure there is no redirect on the `notification_url` set on dashboard. You can open the url on web browser, then see whether the end url is the same as the original url or not, if different, it means there is a redirect. Use the last url displayed on browser URL bar, after you open the original url as `notification_url`.

### Why Permata VA cannot be customized?
Permata custom VA only available for B2B VA type. Permata B2C VA type cannot be customized.

### Why is Customer’s credit card transaction got denied / rejected?
Reason of denied credit card transaction can be checked on merchant Dashboard, on payments menu, search and click the transactions order id. On “**Transaction failure**” red indicator, click the little `(i)` icon, deny reason will be displayed.

### Customer transactions are denied because of fraud detection system, but merchant trust the customer, can merchant request whitelist?
If the customer is trusted and Merchant wish to whitelist the customer, please provide the list of customer emails and send the request as email to operations.fraud@midtrans.com

### Customer card denied by bank / blocked on OTP page, customer asked the bank, the bank tell customer that the card is having no issue. What should merchant do?
If the card is blocked within the OTP/3ds page of the card issuer/bank. Customer should contact card issuer/bank call center. Customer should provide the screenshot/message of the 3DS page issue to card issuer/bank. Please note that some card issuer/bank might mistakenly check only if the card have offline payment issue or not, they might not check from online/3DS perspective whether it’s able to transact online or not. customer should mention they are unable to pay on 3DS enabled online merchant, so the call center can understand.

Payment Gateway unfortunately have no direct control over the issue, because the block happen on Card Issuer (and their network) side. Customer should explain the issue to Card Issuer.

### Customer stuck on 3DS / OTP screen. What is happening?
3DS / OTP page is directly served by card issuer/bank's website. The issue is very likely caused by the issue, downtime or maintenance on the website.

Customer should contact card issuer/bank call center. Customer should provide the screenshot/message of the 3DS page issue to card issuer/bank. Please note that some card issuer/bank might mistakenly check only if the card have offline payment issue or not, they might not check from online/3DS perspective whether it’s able to transact online or not. customer should mention they are unable to pay on 3DS enabled online merchant, so the call center can understand.

Payment Gateway unfortunately have no direct control over the issue, because the issue happen on Card Issuer (and their network) side. Customer should explain the issue to Card Issuer.

### Customer does not receive 3DS / OTP, so he can’t proceed with payment or can proceed but transaction become non 3DS. What’s the issue? What should merchant do?
In case of OTP could not be received by customer, the issue is between card issuer’s (bank’s) 3DS (OTP) page and the customer phone. This can be caused because for example:
- Card issuer 3DS page is currently down / under maintenance, 
- Card is blocked by card issuer because fraud attempt or incorrect OTP submitted too many times, 
- Customer phone is unreachable by the SMS service, customer phone don’t have enough credit to receive SMS, etc,
- Card issuer is having issue and downgrading the transaction to non 3DS.

Merchant should inform customer to contact their card issuer support center, they should explain in details that they are unable to do online transaction and did not receive 3DS / OTP. Also inform the error message displayed on the page if any. Make sure they explain about the issue is for online transaction, because in some cases card issuer support center might only check the card issue for offline transaction and tell customer the card is fine and able to transact.

### Merchant don’t want to pass real "customer_details" data to Midtrans, is there any consequences? how should merchant do? 
There is no legal or business constraint if merchant want to do that. But there might be technical consequences in terms of how Midtrans Fraud Detection System (FDS) works.
Few sample cases are:
- FDS have blacklist database of online fraudster, if customer is a fraudster, and merchant pass fake email, fraud might not be detected.
- FDS estimate fraud behavior based on transaction count for each email, phone number, and other unique data. If merchant sent static dummy email, it can get blocked after certain amount of transaction, To avoid this issue merchant should use dynamic dummy data, that is unique for each user. For example merchant can mask customer@email.com to MmtMPHTlqVZyE0@dummycompanyemail.com 

In most cases, merchant can just not sent the customer data, if the data is not mandatory. E.g: Gopay transaction do not require customer email or phone number at all.

### While doing VA transaction, Merchant got error 505 “Unable to create va_number for this transaction.” What's the issue and what should merchant do?
The issue might happen because after several attempts bank unable to allocate VA number for the transaction at that moment. For error code 5xx. Please try to resend the request.

### Customer transaction is blocked by Fraud Detection System, but Merchant are sure the customer are legitimate and should be allowed to do transaction. What should merchant do?
Merchant can check the FDS deny reason by searching the transaction’s order id on Midtrans Dashboard. Click the order id for the order details, then click the (i) icon on the red/yellow highlighted text. Deny reason should be displayed.

If the customer are trusted and verified by Merchant, and Merchant think their transaction should be accepted, Merchant can request to whitelist specific customer by customer email. Please provide the customer email(s) to Midtrans FDS team via email to operations.fraud@midtrans.com  and explain they should be whitelisted. Our FDS team will respond promptly.

### Merchant is switching their system configuration from Midtrans Sandbox environment to Midtrans Production environment (or switching between different MID/Account), but when making credit card transactions they encounter error message "Credit card token is no longer available". What should merchant do?
The issue happen because the server key and client key mismatch. 
Please make sure both the client & server key used is coming from the same MID/account
Please make sure both the client & server key used is coming from the same environment (Sandbox/Production) of that MID/account.

### What is Pending, Settlement, Expire transaction status for asynchronous payment?
On asynchronous payment such as Bank Transfer, Go Pay, BCA Klikpay, etc. Payment code/id is created then should be displayed to the customer, along with the payment instruction (if you enable customer email notification or use Snap product). Next, customer need to complete the real payment (transfer the fund) using the payment provider website/Application/ATM etc.

The available payment statuses are:

- `Deny`: Payment provider rejected the payment code/id creation.
- `Pending`: Payment specific code for customer to pay is created. Customer need to complete payment at the app/bank/payment provider website/Application/ATM etc.
- `Expire`: Customer fail to pay at bank/payment provider within the specified expiry time.
- `Cancel`: Transaction is canceled by trigger from Merchant.
- `Settlement`: Customer payment is successfully confirmed by bank/payment provider.
- `Refund`: Transaction is refunded by trigger from Merchant.

### Should merchant refer to `status_code` API response as transaction status?
`status_code` only refer to the result of current API action (it behave like HTTP status code). For example `status_code: 200`, the meaning depend on which API action context being executed. 

In case of create transaction API action, `200` means the API able to create and receive payment successfuly. In case of get status API action, `200` means the API get status action is success, resulting in transaction found, regardless of the status of the transaction. It does not refer to the transaction status, and never intended as one.

If Merchant want the reference of the most recent transaction status, it is recommended to check the `transaction_status` instead of `status_code`, which will have consistent value regardless of any API action.

### Merchant’s developer tested failure scenario within Go-Pay simulator, but nothing happens, transaction status still pending, what happens?
That is expected, in production mode a failure of payment within Go-Jek App will be contained only within the app, and will allow customer to retry payment. So failure is not notified to Midtrans or Merchant. Transaction status will remain pending, to allow retry attempt from customer. If customer fail to do successful payment until expiry-time exceeded (default expiry is 15 minutes) the transaction status will then change to `EXPIRE` and cannot be paid.

### For MIGS acquiring (and facilitator agreement type), if my customer says their card is deducted but Midtrans says the transaction is failure, what to do next?
<!-- TODO Translate to english -->
Untuk case mismatch status dengan acquiring bank yg menggunakan MIGS (yg biasanya hanya terjadi kalau sisi MIGS terjadi timeout), silahkan dari Merchant cek ke portal MIGS menggunakan akun yg di dapat dari pihak bank. Kemudian cek/search apakah id tersebut transaksinya success. ID yg bisa di search di portal MIGS adalah `transaction_id` pada response API Midtrans.

Kemudian jika memang transaksi success di portal MIGS, silahkan tentukan transaksi akan di cancel/refund, atau dibiarkan success. Jika dibiarkan success, silahkan infokan ke Midtrans agar kami menyamakan status transaksi menjadi success.

### Merchant is 3DS, card is 3DS, but 1st time transaction fail and no OTP/3DS displayed, what to do?
Search PAPI graylog by txn id, find for keyword `error` or keyword `exception`, for example you may find
```
done executing verify to netcetera with response={"session_id":"00259985-2a42-4196-bdf4-50c872c4ef71","error":{"message":"The input value for the element 'merchant.url' is missing.","code":"INP-MISS","element":"merchant.url"}}
```
It means Merchant URL is required but empty, ask merchant to complete their Dashboard Config.

### Using Midtrans iOS Mobile SDK, when submitting to Apple App Store merchant got warning about deprecated `UIWebView`, what to do?
Note: Only applicable if you are using Midtrans iOS SDK, specifically under version v1.16.0

Recently, Apple introduced a new App Submission warning stating that they are formally deprecating `UIWebView` in favor of `WKWebView`. We wanted to let you know that Midtrans iOS SDK has been updated to use `WKWebView` on our latest version of Midtrans iOS SDK `v1.16.0` to meet the new Apple App Submission requirement. Please ensure update to this latest version next time you plan to submit your app to App Store.

### Customer fail to be redirected to gojek:// deeplink on mobile app, what to do?
If merchant is using android app webview to open the deeplink url, webview need to be configured to allow open deeplink to other app 

Please make sure that the webview allow opening `gojek://` deeplink protocol. 

**Android**

On Android please refer to: https://stackoverflow.com/a/32714613 . You need to modify your web view shouldOverrideUrlLoading functions as follows:
```java
 @Override
 public boolean shouldOverrideUrlLoading(WebView view, String url) {
        LogUtils.info(TAG, "shouldOverrideUrlLoading: " + url);
        Intent intent;

        if (url.contains("gojek://")) {
            intent = new Intent(Intent.ACTION_VIEW);
            intent.setData(Uri.parse(url));
            startActivity(intent);

            return true;
        } 
 }
 ```

**iOS**

On iOS, you will need to add `LSApplicationQueriesSchemes` key to your app's `Info.plist`

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
<string>gojek</string>
</array>
```

**Web Browser & Progressive Web App (PWA)**

If customer is transacting via Mobile Web Browser & PWA and Gojek App fail to be opened, please make sure that you are not trying to open `gojek://` deeplink via Javascript. Some web browser **may block** link opening/redirection via Javascript, because browser consider it as malicious pop-up.

**Don't do** this, via javascript:
```javascript
window.open("gojek://gopay/merchanttransfer?tref=RHHM5IIFEIZCAUEWYDFITLBW", '_blank');
```

Instead please **do this**, allow customer to click the deeplink URL themselves, for example via HTML link element (`a` tag):
```html
<a href="gojek://gopay/merchanttransfer?tref=RHHM5IIFEIZCAUEWYDFITLBW" rel="noopener" target="_blank">Click to Pay with Gopay</a>
```
So browser will allow opening the deeplink URL, because it recognize it as valid user click.

**React Native**

On React Native try whitelisting the deeplink via the `originWhitelist`, for example:

```
<WebView
    {...this.props}
    bounces={false}
    originWhitelist={["https://*", "http://*", "gojek://*"]}
    allowFileAccess={true}
    domStorageEnabled={true}
    javaScriptEnabled={true}
    geolocationEnabled={true}
    saveFormDataDisabled={true}
    allowFileAccessFromFileURLS={true}
    allowUniversalAccessFromFileURLs={true}
  />
```
If it doesn't work try `onShouldStartLoadWithRequest`, for example:

```
<WebView
    ...
    onShouldStartLoadWithRequest={this.openExternalLink}
    ...
  />
```

then implement function to handle `gojek://`, for example:

```javascript
import { WebView, Linking } from 'react-native';

openExternalLink= (req) => {
  const isHTTPS = req.url.search('https://') !== -1;

  if (isHTTPS) {
    return true;
  } else {
    if (req.url.startsWith("gojek://")) {
      return Linking.openURL(req.url);
    } 
    return false;
  }
}
```
or try these references:
- https://facebook.github.io/react-native/docs/linking#opening-external-links
- https://stackoverflow.com/questions/54248411/react-native-deep-link-from-within-webview
- https://stackoverflow.com/questions/56800122/err-unknown-url-scheme-on-react-native-webview
- https://stackoverflow.com/questions/35531679/react-native-open-links-in-browser


### Merchant developer encounter `javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure` when trying to connect to Midtrans API url, what to do?
This usually caused by outdated Java client. Please check the Java version, web framework version, and OS version used to connect. Please make sure you are not using outdated version, and stay updated, for example if your versions are: java version 1.7, web framework version Spring 3.1  and OS version windows 7. Please update. Java version 7 are no longer officially supported by Oracle (https://java.com/en/download/faq/java_7.xml ). Other than that Spring & OS version is also outdated. Using outdated platforms make your system vulnerable to security threats, which is not a suitable environment for handling payments.

### Does Midtrans provide SSL certificate or root certificate for Merchant to download?
Merchant **should not pin/download Midtrans' SSL certificate manually** (storing it locally as file) for your system to verify connection. Please let the HTTP client auto-verify SSL certificates (the default behaviour). That is not best practice and might break your future integration if our certificate is updated/upgraded.

### Does Midtrans provide IP address of the API domain to be whitelisted from Midtrans side (outbound request from Merchant to Midtrans)?
Midtrans API endpoint is distributed and protected with multiple layers of security, including Cloudflare, **so it will not have any specific IP address**. Please whitelist our API domain name instead:

```
api.midtrans.com
app.midtrans.com
```
> Always use the domain name to contact our API — never an IP address.


Or if you really need to, you can try to whitelist these IP: https://www.cloudflare.com/ips/

However, you are responsible to always update the list yourself incase it got updated, else it might break your integration with Midtrans.

For inbound request from Midtrans to Merchant, we provide IP address to whitelist:
Please refer to: https://docs.midtrans.com/en/reference/address.html

### Merchant developer are using React/Vue/Angular (or any frontend framework) PWA, does Midtrans have any compatible library?
Please note that those framework is frontend JS frameworks.

Midtrans CoreApi product require backend only integration, you are free to use anything you want in the frontend, as long as it able to display the data returned by CoreApi on backend.

Midtrans Snap product require backend & frontend integration. On the backend, those famework doesn't matter, you still can use [our library](https://beta-docs.midtrans.com/en/technical-reference/library-plugin) based on your backend tech stack. While on the frontend you [can follow this answer below](#merchant-developer-use-react-js-frontend-framework-and-unable-to-use-midtransminjs-snapjs-what-to-do)

### Merchant developer use React JS frontend framework, and unable to use midtrans.min.js / snap.js, what to do?
Please be aware that React (or other frontend framework) does play nicely with regular JS library inside `<script>` tag. Both are same frontend based JS anyway, so they can still access each other. So include the midtrans.min.js / snap.js as `<script>`. `Veritrans` object still available as global `window.Veritrans` object inside react. `Snap` object still available as global `window.Snap` object inside react. Please refer to https://github.com/facebook/create-react-app/issues/3007#issuecomment-357863643 

### Merchant tried to send request to Midtrans API, but always get blocked by CORS Policy on the browser, what's the issue?
For now that is expected you will get CORS issue when calling `/transactions` endpoint from frontend (at least until our Snap API team decided to allow CORS). Please send the API request securely from backend.

Because for security purpose, **you should not call API which require Server Key authorization from Frontend**. You are in risk of exposing your Server Key to public (which should be kept secret). Server Key on frontend code are easily accessible from client side. Server Key should be used from backend (server). You can send the frontend HTTP request to your backend first, which your backend should securely add the Authorization header, then send the request to the API.

### Customer complains the refund is not yet received, but on Midtrans Dashboard the status has already `refund`, can you check?
Unfortunately, access to fund status checking is not granted to Tech Support Team, we can only check the status technical wise. The actual fund checking will need to be inquired to Business Operations Team (which may reconcile with Go-Pay team as needed). To ensure you have correct point of contact and faster response regarding fund status, please follow below steps:

---------------------------
Here are some things that can be done if there are any mismatch in the transaction using the GO-PAY payment method:

1. If the customer complains that there is no refund after 3 working days from a refund submission to Midtrans, merchant needs to provide information to Midtrans Business Team and the Business Operations Team (bizops@midtrans.com) with the following detailed information:
	- Order ID
	- Transaction date
	- Gross amount
	- Refund amount
	- Screenshot of customer GO-PAY payment history, if any

2. If there is any mismatch found in the transaction on the customer’s payment proof and transaction data at MAP, then you can contact the Business Team and the Business Operations Team (bizops@midtrans.com) by providing the following information:
	- Order ID
	- Transaction Time
	- Transaction Amount
---------------------------

### Merchant use Midtrans provided CMS plugin/module, but found that it does not exactly suit their specific needs, can Midtrans help change the plugin/module?
Midtrans provides easy-to-install popular CMSes plugin/module to cover general merchant use cases plus some advanced features, and will try our best to cover use cases that are generally needed by most merchants. 

However, please note that any of the CMS module is developed as-is and without any warranty (MIT licensed, see the License file). No warranty of it will cover all specific merchant needs, due to each merchant use case can be very different and custom. There is no one-fits-all solution for those. 

Midtrans publish and open source code of the modules. Merchant’s developer team should be able to further develop or use it as basis for their customization needs, because fulfilling merchant’s specific use cases should be their responsibility. 

Midtrans however is fully responsible to payment API service (not CMS plugin/module, but API), which you can let us know if there’s feedback for us to review related to the API. Midtrans will also consider implementing feedback for CMS plugin/module if the feedback can be beneficial to general use cases.

### Our customer/team tried to pay using a Debit Card, but we are getting responses that the transaction is denied, what is the issue?
Please note that not all debit card (especially in Indonesia) is activated by the card issuer (bank) for online payment. That can vary depends on card issuer (bank) regulation, most major Indonesian banks still doesn’t activate debit card for online transaction by default, because of security reasons. In this case, customer (card holder) should ask their card issuer (bank) whether their debit is activated for online transaction or not, before attempting to make online transactions.

Additionally, some customer service of card issuer (especially in Indonesia) also don’t really understand the difference between online and offline transactions, they would often mistakenly inform customer that the debit card is available for transactions. Customer should explains the issue in detail providing which website they try to transact, and evidence (screenshot, web url), etc if available. So the card issuer can better understand the case.

### How does Midtrans ensure customer card data is securely being transmitted to Midtrans and not compromised by 3rd party?
All the data transmission coming from customer device to Midtrans should be encrypted over the network layer via SSL/HTTPS. That means data transmission is end-to-end encrypted (customer-to-midtrans), and secure from any 3rd party. Only Customer and Midtrans side can see the real value of data being transmitted. Unless a 3rd party have direct control over the user’s device (which means already compromised anyway) or he able to decrypt SSL/HTTPS.

In order to ensure the customer-to-midtrans encryption are properly implemented, merchants are required to use official Midtrans provided javascript library for card transaction (midtrans.min.js, snap.js, or Mobile SDK) and strictly prohibited to record card credentials to their own system, unless PCI DSS certified. 
If you have proof of how a 3rd party can possibly break this security, feel free to contact us, we will connect you to our Infosec team.

Some auditors may see credentials transmitted in plain text, it is because the audit happen on the customer device itself. So ofcourse the data are expected to be visible from customer device. Auditor should try to check from non customer device, for example from network layer as 3rd party between midtrans and customer. Auditor will see the data is encrypted from 3rd party.

Additionally HTTPs GET method encrypt any GET query / request credentials (via TLS/HTTPS), it may expose the destination web domain to proxy, but **will not** expose any parameter.

### Merchant have reported that the transaction amount on the Snap pop up can be tampered/modified/changed from customer side, is it safe?
Amount that may be tampered/modified on Snap is strictly only on the front-end side. Back-end wise you can check on the Midtrans Dashboard that the amount remain the original value, and amount being charged to customer by bank is also still the original value.

Tampering on the front-end side is common things and can possibly done via Browser Dev Tools like inspect-element tool, and only affect the front-end of that specific customer. Which does not posses any potential risk and is outside of Midtrans or Merchant control. 

If you have proof of how this can possibly posses significant risk, feel free to contact us, we will connect you to our Infosec team.

### Merchant is using Gopay `callback_url` but customer does not redirected to expected url/deeplink what is wrong?
For Gopay transaction, merchant can specify `callback_url`, customer will be redirected to `callback_url` after attempting Gopay payment within GOJEK app, wether the result is failure or success. If customer did not get redirected properly, please check the following points:
- **Is customer making payment on GOJEK app via QR Code?**
Making payment by scanning QR will not result in redirect. Only `gojek://` deeplink method will result in redirect.

- **Do merchant use `http/https` protocol as the url?**
Make sure to add trailing slash `/`  at the end of the url. For example `https://myshop/finish_payment/`. Gopay will automatically append `?<some-query>` at the end of the url, some web framework unable to handle `?` directly appended to your url like `https://myshop/finish_payment?order_id=123`, so you have to ensure to add `/`.

- **Do merchant use app deeplink protocol as the url?**
Make sure merchant app already handle that deeplink url, for example `slack://finis_payment/`, make sure the `slack` app can handle `/finish_payment` as deeplink

- **Do the callback_url trigger any redirect?**
Sometime that url trigger redirect to another url, or you have internal redirect rule within your network/device. Please check that url.

\#gopay \#mobile \#snap

### How long is Gopay transaction will be available to be paid after being created (after pending transaction status)?
By default expiry for Gopay transaction is 15 minutes. However this can be customized by sending additional JSON parameter during transaction creation. Merchant can send `custom_expiry` ([Core API](https://api-docs.midtrans.com/#charge-features)) or `expiry` ([Snap API](https://snap-docs.midtrans.com/#json-objects)) parameter.

It is **not recommended to set expiry below 15 minutes**, because Midtrans' expiry scheduler only reliably expire transaction with 15 minutes or more expiry, 15 minutes is also might be subject to some delay on batch processing of periodic expire transactions. If merchant want the transaction to expire in real time or less than 15 minutes, they can utilize API [cancel](https://api-docs.midtrans.com/#cancel-transaction) or [expire](https://api-docs.midtrans.com/#expire-transaction) instead. Which merchant can trigger at anytime on a `pending` transaction.

\#gopay

### Can merchant retrieve/store Gopay deeplink url in mobile app?
For Snap payment product:
Payment UI is managed by Midtrans
- If merchant are using Snap.js or redirect_url to display transaction, the Gopay deeplink/QR url is currently not retrievable from merchant side.
- If merchant are using Midtrans Mobile SDK (native Android/iOS), deeplink are retrievable using the SDK.

For Core API payment product:
Integration is API based, so deeplink/QR url can be retrieved by merchant directly as API response and can be stored by merchant however they like.

\#gopay \#mobile

### Can merchant force Snap to always show deeplink or QR for Gopay transaction?
Yes, Snap can be configured to specifically show QR, deeplink, or automatically guess the device type. On `snap.js`, there is option `options.gopayMode` that can be used by Merchant. To configure Snap to always show QR for example,

If using `snap.js` popup mode, add `gopayMode` on second parameter when calling `snap.pay`:
```javascript
snap.pay('<SNAP_TRANSACTION_TOKEN>', {
  gopayMode: "qr" 
  // possible value gopayMode: `qr`, `deeplink`, `auto`
})
```

If using Snap's `redirect_url`, append `?gopayMode=deeplink` after the url:
```
https://app.midtrans.com/snap/v2/vtweb/c9e25cd7-1b89-4fc9-8cb8-ab0342eac21f?gopayMode=qr
```

\#gopay \#snap

### Why is customer Gopay deducted while the transaction recorded as failure/expire on Midtrans Dashboard?
In the very rare case of Gopay system already deduct customer’s Gopay but experiencing issues that may result in failure to notify Midtrans (and Merchant) about the transaction status, Gopay system will auto-sync transaction on their end by refunding the payment. This mechanism intended to sync up transaction status between Merchant-Midtrans-Gopay to failure state. Merchant can always refer to status on Midtrans, as the most accurate (and final) status. Merchant may advise customer to re-check their Gopay balance periodically to ensure that their balance is refunded, as the refund can be instant or might take a while depends on Gopay internal process. If customers still does not receive any refund, Merchant can email bizops[at]midtrans.com with following information: Order ID, Transaction date, Gross amount.

If the customer wish to proceed transaction, please create new transaction.

> NOTE: **Do not** deliver good/service to customer, if transaction status on Midtrans is not `settlement`/success.

\#gopay

### When can I refund GoPay transaction?
After transaction is settlement. You can instantly refund it, up to maximum of 45 day after settlement.

There is fund related limitation though, you can only refund if your Midtrans account have sufficient payable amount on it. Usually a transaction become payable within 5 days or less (depend on business agreement, please check with @sandy). This is to prevent refund exceeding the total amount received.

So for example you received a transaction worth IDR 100K, and IDR 0 payable today, you can't immediately refund. Then after 5 days, let's say your payable already increased to IDR 100K, you can refund any number of transactions as long it does not exceed that payable amount.

\#gopay \#refund

### Merchant/Customer tried failure payment scenario on E-Wallet transaction, but transaction status does not updated to failure?

If you as merchant are declining payment, via API `/cancel` the transaction status will become failure.

But if customer is trying to pay then it fails within the E-Wallet app, they have chance to for example topup their E-Wallet and retry the payment. It is expected that the status will remain as `pending`. Until it eventually become `settlement` or `expire` because of time limit.

So E-Wallet may contains the failure within their own app and does not emmit failures to merchant/Midtrans, in order to give customer chance for retries. From business perspective it also increase your payment success rate and may increase revenue.

\#gopay \#refund

### How to use Core API **register card** endpoint from frontend?
Please follow [this demo](https://gist.githack.com/rizdaprasetya/cecce986cb3c71ca0ec1a404d3063105/raw/4fadc5e425b4ddc770a005e33383fd1a8e481134/index.html ':include :type=iframe width=100% height=400px')

### Merchant use Mobile SDK but unable to get Snap Token, how to resolve it?
Typically the Mobile SDK transaction flow is: 
1. Merchant mobile dev config & setup Midtrans Android/iOS SDK especially the environment, clientKey & merchantServerURL
2. Given correct implementation, SDK will do API request to merchantServerURL to retireve Snap transaction token
3. Backend implementation of that merchant server will forward the API request to Midtrans' Snap API endpoint, adding HTTP auth header from serverKey
4. Snap API will respond with token, merchant server need to print/output the API response as-is
5. SDK will auto parse the API response, and once token retrieved, payment page will be started.

Things to do:
- Make sure that on 1# you have config the correct sandbox environment, merchantServerUrl & clientKey.
- Make sure that on app, the [SDK implementation is correctly following the docs](https://mobile-docs.midtrans.com/)
- Make sure to implement merchant server / backend.
- Make sure that on backend / 3# you have config the correct serverKey and API endpoint to correct sandbox config.
- Incase the issue persists, please share any error messages recorded on log, either from the Mobile or backend.
- Check the backend log to see if it's able to get API response from Snap API, sometime API can reject invalid request. Provide the log to us if needed to check. Or at least the order_id of transaction, so we can crosscheck it with our API log.

### How can merchant disable debug log on Android Mobile SDK?

Merchant can disabled it by setting the value of `enableLog` to `false` . Here is the sample codes:
```java
SdkUIFlowBuilder.init()
        .setClientKey(CLIENT_KEY)
        .setContext(CONTEXT)
        .setTransactionFinishedCallback(new TransactionFinishedCallback() {
                    @Override
                   public void onTransactionFinished(TransactionResult result) {
                   }
                })
        .setMerchantBaseUrl(BASE_URL)
        .enableLog(false) // this is to disable logging
        .buildSDK();
```
\#mobile


### Can merchant set enabled payments list on Android SDK?
It is recommended to define `enabled_payments` JSON field on merchant server (backend), during API request to Midtrans Snap API.

Alternatively, merchant can define enablePayment object as List
```
List<String> enabledPaymentList = new ArrayList<>();
```
and set to transactionRequest object with method `setEnablePayments(enabledPaymentList)`
example code:
```java
List<String> enabledPaymentList = new ArrayList<>();
enabledPaymentList.add("gopay");
enabledPaymentList.add("credit_card");

final UUID idRand = UUID.randomUUID();
TransactionRequest transactionRequest = new TransactionRequest(idRand.toString(), 200000);
transactionRequest.setEnabledPayments(enabledPaymentList);

midtransSDK.setTransactionRequest(transactionRequest);
midtransSDK.startPaymentUiFlow(CONTEXT);
```

### How to display specific payment channel via mobile SDK client code?
It is recommended to specify payment channel from merchant backend / server. Before forwarding request to Snap API, you can modify the JSON payload to add `enabled_payments` parameter. For example, add this to the JSON:

```
...
"enabled_payments": ["credit_card", "mandiri_clickpay", "cimb_clicks",
    "bca_klikbca", "bca_klikpay", "bri_epay", "echannel", "permata_va",
    "bca_va", "bni_va", "other_va", "gopay", "indomaret",
    "danamon_online", "akulaku"]
...
```

If you really need it on client/front end side, try adding this config:

Android Java try utilizing `setEnabledPayments`

iOS Objective C
```
MidtransConfig.shared.customPaymentChannels = @[@"credit_card",@"gopay",@"bank_transfer"];
```

iOS Swift
```
MidtransConfig.shared().customPaymentChannels = ["alfamart"]
```

### Merchant is using Android SDK, but transaction is failing because 3DS is not enabled, what is wrong?
If merchant is creating Snap transaction token via Backend and try to start payment screen with Android SDK, there is known issue, which Android SDK may override the transaction as non 3DS.

As workaround try to implement these code before `startPaymentUiFlow` :
```java
// create CreditCard config object and set as 3DS enabled
CreditCard creditCard = new CreditCard();
creditCard.setAuthentication(Authentication.AUTH_3DS);

// create dummy transaction request to attach CreditCard config
TransactionRequest transactionRequest = new TransactionRequest("",0);
transactionRequest.setCreditCard(creditCard);
// apply the config to SDK
midtransSDK.setTransactionRequest(transactionRequest);
// start the payment UI
midtransSDK.startPaymentUiFlow(this, YOUR_TRANSACTION_TOKEN_FROM_BACKEND);
```

Another symptom of the issue: 
Merchant may encounter card payment success on Sandbox but fail on Production. This is because the transaction is non 3DS, Sandbox account by default allow 3DS and non 3DS transaction, while on Production bank may allow only 3DS transaction for their account.

### Does Midtrans support Flutter, React Native, or other hybrid / non-native mobile framework?
Midtrans does not directly provide official SDK for hybrid mobile framework, because currently we focus on supporting native Android & iOS experience via our [Mobile SDK](https://mobile-docs.midtrans.com).

Rest assured, our payment products are compatible to be used on Flutter, React Native, or other hybrid mobile framework platform. Some of our merchant have been able to do so.

The simplest and easiest method is to utilize **webview** (or similar method to display HTML page), developer can display (via webview) the HTML page of Snap payment (HTML which utilize snap.js).

So developer need to setup web page which integrated with snap.js to display payment, and use webview within the app to display it as payment page.

Alternatively developer can also utilize Core API, which is JSON-based REST API, that should be able to be integrated on virtually any framework/platform.

### During 3DS card transaction frontend callback what does `"status_message": "Failed to generate 3D Secure token"` means?
The 3DS process is not properly completed by customer. It can be various reasons, commonly:
- Customer input wrong OTP/3DS. Which he can retry.
- Customer don't have access to the OTP/3DS device.
- Card issuer decided the transaction is unauthorized. e.g: fraud, stolen card, etc.
- Card issuer 3DS verification system is having issue, maintenance, or downtime. Customer can try to reach card issuer contact center.

Since the issue is between customer and card issuer, merchant & Payment Gateway can only observe.

### Using 3DS Card transaction flow v1 Core API, how to ensure transaction is 3DS end to end?
Parameters that can be check to make sure transaction is 3DS:

1# Get Token Process (endpoint `/v2/token`):
Make sure `redirect_url` present on the response. Sample:
```javascript
{
  status_code: "200",
  status_message: "Credit card token is created as Token ID.",
  token_id: "481111-1114-dc7d77d5-237b-4a7f-a26c-75c4a7aa4e91",
  bank: "bni",
  redirect_url: "https://api.sandbox.veritrans.co.id/v2/token/redirect/481111-1114-dc7d77d5-237b-4a7f-a26c-75c4a7aa4e91",
  hash: "481111-1114-xxx"
}
```
2# Callback from 3DS iframe:
Response object should have `status_message: "Success, 3D Secure token generated"` and `eci` field, as proof the OTP/3DS page has been successfully completed by customer and verified by card issuer. Sample:
```javascript
{
  token_id: "481111-1114-592cfc60-9056-4c6d-bb4d-323fb0ebd97e",
  status_code: "200",
  status_message: "Success, 3D Secure token generated",
  eci: "05"
}
```

ECI for **non-3DS** transaction is `07` or `00` (bad value)
https://support.midtrans.com/hc/en-us/articles/204161150-What-is-ECI-on-3DS-protocol-

3# Charge process (endpoint `/v2/charge`)
There will be `eci` value as well.
ECI for **non-3DS** transaction is `07` or `00` (bad value)

### Can you explain the implementation details of Credit Card 3DS transaction?

Please refer to this sample implementation:
- https://github.com/Midtrans/midtrans-nodejs-client/blob/master/examples/expressApp/views/core_api_credit_card_frontend_sample.ejs
- https://github.com/Midtrans/midtrans-python-client/blob/master/examples/flask_app/templates/core_api_credit_card_frontend_sample.html

Or please refer to this demo:
- https://anice.win/3ds_new/

Which source code is available at:
- https://gist.github.com/rizdaprasetya/9d16893578d600a03075939ef74c5c1f

Sequence diagram:

![card transaction flow](./asset/image/card_transaction_3ds.png)

### Can you explain the flow of Recurring/One Click transaction?
Please refer to below sequence diagram:

![one click flow](./asset/image/recurring_one_click_sequence.png)

### Can you explain the flow of Recurring/One Click transaction with Register Card API?
Please refer to below sequence diagram:

![one click via register card flow](./asset/image/recurring_via_register_card.png)

### Can you explain the flow of Two Click transaction?
Please refer to below sequence diagram:

![two click flow](./asset/image/two_click_sequence.png)

### Can you explain the flow of Gopay transaction?
Please refer to below sequence diagram,

**Deeplink Mode**:

![Gopay Deeplink flow](./asset/image/gopay_coreapi_deeplink.png)

**QR Mode**:

![Gopay QR flow](./asset/image/gopay_coreapi_qr.png)

### Can you explain the flow of Bank Transfer / VA transaction?
Please refer to below sequence diagram:

![Bank Transfer VA flow](./asset/image/va_coreapi.png)

### Can you explain the flow of Recurring transaction using Snap?
Please refer to below sequence diagram:

![snap recurring flow](./asset/image/snap_recurring_sequence.png)

### Merchant updated iOS SDK from v1.14.7 and below, but old implementation did not work after update. How to resolve it?

- Older SDK require config of `CC_CONFIG.secure3DEnabled = ...`, newer SDK no longer requires it, please remove that config. Then add this config `CC_CONFIG.authenticationType = MTAuthenticationType3DS`
- Please make sure that your backend/merchant-server will also accept that changes. Older SDK will generate request that have JSON keys `"secure" : true`, newer SDK will have `"authentication" : "3ds"`. Make sure there are no type checking or similar that are rejecting the JSON.

### There are some missing field/property on the JSON response/notification, it seems the JSON is not consistent, is this expected? 
In Midtrans, we are following [Google JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml). According to the style guide it is recommended if a property doesn't have any value (`null`) then it should be removed. Reference: https://google.github.io/styleguide/jsoncstyleguide.xml#Empty/Null_Property_Values

So yes it is expected to have few missing JSON field/property, it means the value is `null` for that field/property. Please adjust your implementation accordingly to accomodate this behaviour.

### There are new field/property added on the JSON response/notification, it break merchant implementation, is this expected?
Yes it is expected.

To ensure "forward compatibility", as JSON based API common practice, please allow new fields/properties to be added on JSON based API communication, thus including API response and HTTP notification. Ensure your backend are able to ignore and does not break when encountering new fields/properties. Depends on JSON parser library you are using, those parser usually have `JsonIgnoreProperties` flag or similar that can be utilized.

Please adjust your implementation accordingly to accomodate this behaviour.

### How to make sure card transaction is 3DS on iOS
Make sure to use these config on the client code:

```
 CC_CONFIG.secure3DEnabled = YES;
 CC_CONFIG.authenticationType = MTAuthenticationType3DS;
```

Also remove `CC_CONFIG.acquiringBank` if you don't need to specify any bank. 

### Why on iOS device Gopay deeplink is taking customer to different app and not Gojek?
Some apps might interfere with `gojek://` app deeplink url and taking over customer to their app. This behavior is caused by the app, if you find this keep happening please report the intefering app to us and Gojek team will raise the issue to iOS app store for further investigation. As temprorary please inform customer to uninstall the app that causing the interference.

\#gopay

### Why Merchant unable to do other VA transaction on Snap / Mobile SDK?
Please check during Snap token creation, if merchant send `enabled_payments` parameter.
Sending 
```
"enabled_payments": ["other_va"]
``` 
may not work, please change to
```
"enabled_payments": ["other_va","bni_va"]
``` 
or 
```
"enabled_payments": ["other_va","permata_va"]
```
Because behind the scene `other_va` will need to utilize BNI or Permata VA for the transaction.

### How to debug network / API call on Midtrans iOS SDK to check for any API validation error?
Please enable network logging using:
```[[MidtransNetworkLogger shared] startLogging];```
You can provide us the network log result for any issue.

### Can Midtrans show/deduct MDR directly to customer during payment?
Business and regulation wise, MDR as the name suggest, Merchant Discount Rate is merchant responsibility to the payment provider (bank / card network principal). MDR should be charged to merchant, not customer. So it should not be directly charged to customer. 

However merchant may manage on their own the fee charged to customer, based on that. Merchant may define additional service fee or include it to the final item price to customer. 

Technical wise, for example merchant can add fee as additional `item_details` when requesting payment to Midtrans API. e.g:
```json
...
"item_details": [
  {
    "id": "ITEM1",
    "price": 7000,
    "quantity": 2,
    "name": "Apple Fruit"
  },
  {
    "id": "miscfee",
    "price": 250,
    "quantity": 1,
    "name": "Service Fee"
  }
]
...
```

### For failure card transaction, there are `reversal transaction` entry in dashboard, what does it means?

When a card transaction is failure because of acquiring bank timeout (no response after a while), Midtrans will make sure customer fund does not get deducted, by sending reversal command to bank. If customer fund deducted during the timeout, the reversal will makesure the fund will be reverted to customer. If fund is not deducted, then the reversal will do nothing. You should not be alarmed when you see this `reversal transaction` entry in dashboard.

### Merchant is using Midtrans' Shopify integration, sometimes item stock quantity become negative (item oversell), what happened?

This behaviour can happen on some specific case if the item stock is low, and there are multiple customers trying to checkout the same item (usually during promo / flash sale period). If the order is not yet paid, the stock quantity is not yet reserved for that customer. When multiple customer pay their orders within short time, all of the orders can be accepted on Shopify side, causing negative stock quantity. This case happens due to the limitation of Shopify platform integration model with external payment gateway.

For now there are no work around of this behaviour, and Shopify team confirm it as expected behaviour.

The explanation from Shopify Technical Merchant Support Team can be referred below:

> This is actually expected behaviour for an offsite gateway. Because of the way these gateways interact with Shopify, we're currently unable to apply our Cart Hold Policy. 
This stock discrepancy occurs because once the request to move to the gateway is triggered in the checkout we check for available stock. If stock is available, then we permit the customer to proceed to the transaction externally to the Shopify checkout. 
However if it’s the case where multiple orders come in at once, they all have the ability of making it to the external gateway provided there is still at least 1 stock available when they click Complete Payment, this is because we can't decrement inventory levels until the initial customer has returned from the gateway with a successful transaction. 
Then because other customers were permitted to proceed to the gateway, any successful transactions they place will also decrement inventory even if it's already at zero due to the fact there was still at least one in stock when they began payment and the order was permitted to proceed. 

> It is a factor that our support developers are working on and they're constantly testing for a new method of offsite gateways integrating with our inventory services, but as it currently stands this is something that can happen with use of an external payment gateway which is not directly connected to an online checkout

### Why do Google Analytics within Snap payment page is different than the one configured from Snap Preference menu?
When merchant is using `Snap.js`, Snap payment page are loaded within iframe inside merchant's website. The custom Google Analytics UA id merchant set on `Snap Preference` menu on Midtrans Dashboard are loaded dynamically within the Iframe. Merchant might not immediately find it on the HTML, because it is dynamically loaded via Ajax.

As evidence that merchant's custom Google Analytics code is successfully used, open `Chrome Dev Tools` before merchant's payment page is loaded, continue checkout until Snap page/iframe is opened . Go to `network` tab. On the filter or search field type the UA id, e.g: `UA-xxxxxxx-x`. Analytics request sent to Google Analytics server with custom UA id will be found there. It means merchant UA id is correctly being used.

Midtrans also have another built-in Midtrans' UA id within the iframe html. That the purpose is to track Snap pageviews count globally across any merchants using snap, for analytics and debugging purpose.

Merchant can let Midtrans know if they have any concern regarding this.

\#snap \#google-analytics

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

.