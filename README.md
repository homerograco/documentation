# Open Payment Gateway - OPG

This documentation guides you through optile's Open Payment Gateway (OPG) features for frontend checkout and backend use cases. It provides important information about integration scenarios, testing possibilities, and references.

## First Steps

Every potential checkout webpage includes information about the purchase, payment amount, customer details, plus a few additional details. This information is always issued from your server-side system via HTTP POST Request. This is what we call a LIST Request.

To test these features, you need an account for our website. If you do not have an account, please email support@optile.net and we will create one for you. You can start experimenting with OPG Sandbox as soon as you get your optile account details.

Try your first LIST Request by following these 4 steps:

### 1. Configure your OPG Sandbox
* In the portal, go to the Dashboard and open the Provider Contracts section
* Choose Test Adapter "TESTPSP". Make sure you add a valid URL eg (http://www.dummyvalidurl.com)
* Select "Countries and networks", eg Germany (DE), France (FR), and payment methods VISA, MasterCard, SEPA Direct Debit, and PayPal
* In the country-methods matrix at the end of the dialog, select PayPal to be available in France but not Germany
* Access the Routing App from the Merchant Portal, and activate the routes per country by choosing a routing strategy.

### 2. Generate a Sandbox Payment Token

You must generate a Token to authenticate your system against the Payment Gateway API.

To generate a Sandbox token:

* Go to the portal dashboard, open "Tokens" and generate a Sandbox Payment Token. It will be used later to authenticate your system against the Payment Gateway.
* Copy and save the value immediately. For security reasons, it will not be shown again and cannot be recovered. You can create a new one if necessary.

### 3. Install a tool for manual testing

We suggest that you use one of the browser plugins listed in Tools for Manual Testing so you can submit JSON POST requests easily by hand.

### 4. Do a LIST Request
Use the following values to submit your first LIST Request with the chosen tool:

* URL: https://api.sandbox.oscato.com/api/lists
* HTTP Method: POST
* Basic Authentication User Name: [Your merchant code]
* Basic Authentication Password: [Your sandbox payment token]
* Header Content Type:  application/vnd.optile.payment.enterprise-v1-extensible+json
* Header Accept:  application/vnd.optile.payment.enterprise-v1-extensible+json
* Request Body Content: Copy and paste the content on the side to the tool that you're using. This content is just an example; make sure that it stays in JSON Format.

Example LIST Request

```json
    {
      "transactionId": "tr101",
      "country": "DE",
      "customer": {
        "number": "42",
        "email": "john.doe@example.com"
      },
      "payment": {
        "amount": 19.99,
        "currency": "EUR",
        "reference": "Shop 101/20-03-2016"
      },
      "style" : {
        "hostedVersion":"v3"
      },
      "callback": {
        "returnUrl": "https://dev.oscato.com/shop/success.html",
        "cancelUrl": "https://dev.oscato.com/shop/cancel.html",
        "notificationUrl":"https://dev.oscato.com/shop/notify.html"
      }
    }
```

Example LIST Response

```json
    {
    "links": {
      "self": "https://api.sandbox.oscato.com/pci/v1/5a4f43dabc12312dfef5752blt5et3mnk2abc123u0fm79bud2"},
    "timestamp": "2018-01-05T09:22:31.355+0000",
    "operation": "LIST",
    "resultCode": "00000.11.000",
    "resultInfo": "4 applicable and 0 registered networks are found",
    "returnCode": {
      "name": "OK",
      "source": "GATEWAY" },
    "status": {
      "code": "listed",
      "reason": "listed" },
    "interaction": {
      "code": "PROCEED",
      "reason": "OK" },
    "identification": {
      "longId": "5a4f43d7148b512dfef5752blt5et3mnk226k45uu0fm79bud2",
      "shortId": "02694-35736",
      "transactionId": "id_h0010" },
    "networks": {
      "applicable": [{
        "code": "MASTERCARD", "..." }]
    },
    "..."
```

Find out more about authentication and version headers in API Access.

For your request you should receive a successful HTTP 200 response with a list of available payment methods for this transaction (attribute networks.applicable).

If you configured your OPG Sandbox as described above, Carte Bleue will not show in this list.

### 5. Visualize the LIST

Follow the steps below to visualize the content of a generated LIST with a default style, using optile's Hosted Payment Page:

* Examine the LIST Response and take the content of the links.self attribute. That's the URL of the LIST Resource:
https://api.sandbox.oscato.com/pci/v1/59722056cb4280f9550078e3lffpph128vffgueil7chn475bs
* Append it as query parameter listUrl to the following URL of the Hosted Payment Page, and open it in a browser:
https://resources.sandbox.oscato.com/paymentpage/v3/responsive.html?listUrl=https://api.sandbox.oscato.com/pci/v1/59722056cb4280f9550078e3lffpph128vffgueil7chn475bs

Note: You have 30 minutes to make a successful CHARGE. After 30 minutes, you will get an error and you should create a new LIST Session (That's also why the link above will not work).

The payment page should look like the example picture below:

![Payment Page](https://github.com/homerograco/documentation/blob/master/payment_page.png)

This payment page has all necessary capabilities to validate and process payment account input. For example, try using the following test account and then click on the Pay button:

```
Card number: 5500000000000004
Expiry Date: any date in the future
Security Code: any 3 digit number
Account Holder: John Doe
```

Using this input above will trigger instant validation, and clicking on Pay will trigger a frontend CHARGE request and should take you to the success URL defined in the LIST request. Alternatively, you can perform the CHARGE request from your backend which we will simulate together in the next step.

### 6. CHARGE request

The LIST response as generated in step 4. contains information about the current payment session, including all available payment networks and their attributes as seen in the array  `networks.applicable`. In this example every network has the same attributes, but for our testing we are now only going to look at `links.operation` from Mastercard:

```json
"operation": "https://api.sandbox.oscato.com/pci/v1/59722056cb4280f9550078e3lffpph128vffgueil7chn475bs/MASTERCARD/charge"
```

Notice that this operation endpoint starts with a reference to the current LIST long ID and ends in `/MASTERCARD/charge`, which means that for the Mastercard network the next operation after listing it is a charge on the customer account. This operation URL is the endpoint to which you need to submit the CHARGE request via `POST`.

In the case of Mastercard, the charge cannot happen without a customer account. So, the CHARGE in this case will carry also a JSON body containing the collected customer account. We can use the same data as done in the step 5. with the hosted payment page, but now mapping them into the appropriate attributes as expected by the payment API.

The CHARGE request will then be done as exemplified in the right pane, using the same authentication and headers as previously done with the LIST request.

A successful CHARGE response should contain a "charged" status code and a redirect URL to the success page as defined in the LIST request.

### Play Around
Make another LIST Request, with an incremented `transactionId` (it's technically not required but it is good practice to keep them unique) and change the attribute `country` to `FR`. Submit again. Now you should see a similar list in the response, but this time with PayPal because we configured it earlier for France only.

Also, you can copy the logo URL (attribute `links.logo`) of one of the networks and open it in a browser. The logo of that payment method will show.

Take one of the credit cards in the list and open the URL from `links.localizedForm` in a browser. You will see a snippet of a simple HTML form without any styling. If you like, you can open the HTML source code. This snippet is another building-block provided by OPG to generate the payment page.

To see an out-of-the-box example of how a payment page is rendered, see our Demo. You can also play around with the country parameter that should go into the LIST Request as well as several parameters offered by our AJAX Integration JavaScript library.

From here on we recommend you to understand our Integration Scenarios and choose the one that better fits your business model. We also have more LIST use cases and backend use cases designed to cover specific checkout flows to fit your application.
