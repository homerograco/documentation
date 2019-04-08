# Open Payment Gateway - OPG

This documentation guides you through optile's Open Payment Gateway (OPG) features for frontend checkout and backend use cases. It provides important information about integration scenarios, testing possibilities, and references.

## First Steps

Every potential checkout webpage includes information about the purchase, payment amount, customer details, plus a few additional details. This information is always issued from your server-side system via HTTP POST Request. This is what we call a LIST Request.

To test these features, you need an account for our website. If you do not have an account, please email support@optile.net and we will create one for you. You can start experimenting with OPG Sandbox as soon as you get your optile account details.

Try your first LIST Request by following these 4 steps:

1. Configure your OPG Sandbox
* In the portal, go to the Dashboard and open the Provider Contracts section
* Choose Test Adapter "TESTPSP". Make sure you add a valid URL eg (http://www.dummyvalidurl.com)
* Select "Countries and networks", eg Germany (DE), France (FR), and payment methods VISA, MasterCard, SEPA Direct Debit, and PayPal
* In the country-methods matrix at the end of the dialog, select PayPal to be available in France but not Germany
* Access the Routing App from the Merchant Portal, and activate the routes per country by choosing a routing strategy.

2. Generate a Sandbox Payment Token
You must generate a Token to authenticate your system against the Payment Gateway API.

To generate a Sandbox token:

* Go to the portal dashboard, open "Tokens" and generate a Sandbox Payment Token. It will be used later to authenticate your system against the Payment Gateway.
* Copy and save the value immediately. For security reasons, it will not be shown again and cannot be recovered. You can create a new one if necessary.

3. Install a tool for manual testing
We suggest that you use one of the browser plugins listed in Tools for Manual Testing so you can submit JSON POST requests easily by hand.

4. Do a LIST Request
Use the following values to submit your first LIST Request with the chosen tool:

* URL: https://api.sandbox.oscato.com/api/lists
* HTTP Method: POST
* Basic Authentication User Name: [Your merchant code]
* Basic Authentication Password: [Your sandbox payment token]
* Header Content Type:  application/vnd.optile.payment.enterprise-v1-extensible+json
* Header Accept:  application/vnd.optile.payment.enterprise-v1-extensible+json
* Request Body Content: Copy and paste the content on the side to the tool that you're using. This content is just an example; make sure that it stays in JSON Format.

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
