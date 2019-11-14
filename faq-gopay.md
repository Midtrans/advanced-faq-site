<!-- Copy pasted from faq-general, FAQ with tag \#gopay and partner-gopay-pos-->

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