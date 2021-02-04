## CEX Direct integration guide
### Overview
This guide will help you set up CEX Direct crypto buy solution via IFrame or redirect on your side.
We allow customisation with a *CSS* file to match your website's theme.

### Getting started
To start the flow you will need to build the *calculator* on your side and then launch the IFrame with users' *values*.
*Note, alternatively you may bypass the calculator step and launch the IFrame right away.*

### Building a calculator
You will need the following parameters.

*Required:*
- `placementID` - a unique marker for each placement of a calculator. 
This will allow you to track performance.

*Optional:*
- `placementSecret` - used for order tracking purpose.
- `clientOrderID` - a unique GUID that you can assign to an order originating on your side. Also used for tracking an order.

*Note, `placementID` with agreed preset parameters (i.e. fees, currencies) will be configured for you once agreement is signed.*

*Meanwhile, you may use our sample `placementID` for testing* - 8e29e9b1-f3d0-4dd4-9920-08773ebcf0fa

### Testing a calculator

Steps to do it:
1) Get available coins and rates;
2) Get precisions and rounding rules;
3) Calculate a rate and use the IFrame to place an order;

Now we will go through the process step by step.

### 1. Getting the coins and rates.
Send a call:
```
https://api.cexdirect.com/api/v1/payments/currencies/{placementId} 
```

You will receive array of currency pairs :

```
[
  {
    a: 0.0000990154556955063,
    b: 0.0005,
    c: 1,
    crypto: "BTC",
    cryptoPopularValues: ["0.01", "0.05", "0.1"], fiat: "USD",
    fiatPopularValues: ["100", "200", "500"] 
    },
  {
    a: 0.00010979641127605852,
    b: 0.0005,
    c: 1,
    crypto: "BTC",
    cryptoPopularValues: ["0.01", "0.05", "0.1"], fiat: "EUR",
    fiatPopularValues: ["100", "200", "500"]
  }, 
  {
    a: 0.003135035352303995,
    b: 0.001,
    c: 1,
    crypto: "BCH",
    cryptoPopularValues: ["0.01", "0.05", "0.1"], fiat: "USD",
    fiatPopularValues: ["100", "200", "500"] 
    }
 ];
```

**Values a, b, c will be used to calculate the rate.**
*Note, you can use preset popular values to have an understanding of what users like to purchase in this currency pair.*

### 2. Getting the precision and rounding rules.

Use precisions to correctly round calculation and set min and max values.

Send a call:
```
https://api.cexdirect.com/api/v1/merchant/precisions/{placementId} 
```
You will receive:

```
[
  {
    currency: BCH",
    currencyPrecision: 8, maxLimit: "0",
    minLimit: "0.01",
    type: "crypto", 
    visiblePrecision: 4, 
    visibleRoundRule: "trunk"
  }, 
  {
    currency: "BTC",
    currencyPrecision: 8, 
    maxLimit: "0",
    minLimit: "0.01",
    type: "crypto", 
    visiblePrecision: 4, 
    visibleRoundRule: "trunk"
  }, 
  {
    currency: "USD", 
    currencyPrecision: 2, 
    maxLimit: "1000", 
    minLimit: "50",
    type: "fiat", 
    visiblePrecision: 2, 
    visibleRoundRule: "trunk"
  } 
]
```

### 3. Calculating and placing an order.

- To calculate rate, you should know use input value from user and insert it to the following formulas using values **a,b,c** from step 1:

To calculate an amount of crypto use `(a * fiat - b) / c`

To calculate an amount of fiat use `(crypto * c + b) / a`

- Round the result using data from step 2:

`visiblePrecision` - number of characters after dot;

`visibleRoundRule` - *this is a rule how you should round your calculations:*

*`trunk` - just cut to the specified number;* 

*`bigger` - round up;*

*`smaller` - round down;*

*`dynamic` - the number of zeros entered by user is displayed.*

## Embedding the IFrame

To embed the IFrame on your page you should follow this example:

```
<iframe allow="geolocation" frameborder="0" height="450px" width="600px"
src="https://api.cexdirect.com/?placementId=8e29e9b1-f3d0-4dd4-9920-08773ebcf0fa&calculatorValues={"fiatCurrency":"USD","fiatValue":"250","cryptoCurrency":"BTC","cryptoValue":"0.0240"}&wallet=1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2&clientOrderId=0cf8fc5d-6f03-4217-ab83-ccf0a316afac"
></iframe>
```

## Parameters description:

`allow="geolocation"` - this flag is optional and it gives an access for geolocation of user;

`height and width` - you can control widget size on your page;

`src` - url of iframe with important params.

`placementId` - your configuration of the widget, without it a page won't load;

`calculatorValues` - this is optional parameter, it decides which step to show first. 

- Here you should pass values from your calculator:

`fiatCurrency` - selected fiat currency (USD, EUR etc.);

`fiatValue` - fiat amount;

`cryptoCurrency` - selected crypto currency (BTC, TRX, ETH and etc);

`cryptoValue` - crypto amount;

`wallet` - user's crypto address. Optional. If nothing is passed, a client will be able to enter it later.

`merchantUri` - a link that defines where the "Return to" button at the end of the flow will direct a user.

`useremail` - user's email address. Optional. If nothing is passed, a client will be able to enter it later.

`country` - user's country. Optional. If nothing is passed, a client will be able to choose it later.

`state` - user's state, only in case `country` is US. Optional. If nothing is passed, a client will be able to choose it later.

`clientOrderId` - optional unique GUID, passed to be able to get an order status (if such 'clientOrderId' already exists, flow will be crashed).

**This will start an order flow for user in the IFrame.**

*Note, if you don't pass `calculatorValues` to the IFrame or `calculatorValues` data is invalid , widget will load its own calculator. 

If `calculatorValues` is valid, widget will move users to step of filling email and country.

## Redirecting a flow to cexdirect.com

If you are willing to redirect your customers to cexdirect.com instead of setting up the IFrame on your side you should use a following link:

``` 
https://cexdirect.com/buy?placementId=
```
You may add all the above listed parameters: `calculatorValues` , `wallet` , `merchantUri` , `clientOrderId`, `useremail` , `country` , `state`

## Getting a list of supported countries and US states.

Request Method: `GET`

Request URL: https://api.cexdirect.com/api/v1/payments/countries/placementId

## Link encryption

IFrame or redirect link can be encrypted using the following guidelines:

**Header**

```
{
  "alg": "RS256",
  "typ": "JWT"
}

```
**Payload example**
```
{
  "calculatorValues": {
    "fiatValue": "120.55",
    "fiatCurrency": "USD",
    "cryptoValue": "0.4000034",
    "cryptoCurrency": "BTC"
  },
  "placementId": "77777777-7777-7777-7777-777777777777",
  "wallet": "wallet",
  "clientOrderId": "clientOrderId",
  "merchantUri": "https://website.com",
  "userEmail": "test@test.com",
  "country": "US",
  "state": "CA"
}
```
**Signature**
```
RS256(secret, base64urlEncoding(header) + '.' + base64urlEncoding(payload))
```
Secret will be provided upon the request.

## Tracking orders (optional)

If you want to track an order within your system you may use the following instructions.
**Note, to track an order you have to pass `clientOrderId` for each order generated on your side.**

- Firstly, you would need your `placementID` and secret. As example:

```
// placementId =  "8e29e9b1-f3d0-4dd4-9920-08773ebcf0fa"
  // placementSecret =  "af087b108d1bc50735951f81a4d39126"
const crypto = require('crypto');
const nonce = new Date().getTime();
```

- Then, you'll need to Generate a signature.
```
function generateSignature([placementId, placementSecret, nonce.toString()]) 
  { 
    const hash = crypto.createHash("sha512"); 
    params.forEach(value => hash.update(value)); 
    return hash.digest("hex").toUpperCase();
  }
```
- Run a POST request.
```
{
const signature = generateSignature([placementId, placementSecret, nonce.toString()]);
  //POST https://api.cexdirect.com/api/v1/orders/getOrderInfo
  const req = {
    "serviceData":
      {
        "placementId": "<< placementId >>", 
        "deviceFingerprint": "deviceFingerprint", 
        "signature": signature,
        "signatureType": "msignature512", 
        "nonce": nonce
      }, 
     "data": 
      {
        "clientOrderId": "<< clientOrderId >>" 
      }
}
```
- Here's how successful response looks like.
```
const res = 
{
  code: 200,
  nonce: 1574339717108,
  data:
  {
    placementId: '77777777-7777-7777-7777-777777777777', 
    status: 'settled',
    userEmail: 'test@test.com',
    createdAt: 1574246991678,
    orderId: '332d1824-db60-4456-9ee4-99323f47167a', 
    clientOrderId: '346682a2-cafe-4aeb-8eed-2c8d93187207'
  } 
};
```

## Possible status values

`"new"` - Order is created, Email/Country are filled in;

`"new-returned"` - Order is created, user who previously used our service is asked to verify it with email confirmation;

`"email-returned"` - Order is created, user who previously used our service has confirmed his email;

`"verification-ready"` - User clicks Next on Verification step;

`"verification-in-progress"` - Shown for a brief period when verification is in progress. Transition status; 

`"verification-rejected"` - User's verification has been rejected;

`"verification-failed"` - Verification services are down/not responding;

`"processing-acknowledge"` - User has been requested to provide additional data (basic ID info such as name, surname);

`"processing-ready"` - Additional data is filled and sent;

`"processing-3ds"` - Payment 3DS screen;

`"processing-rejected"` - Payment has been rejected by service provider;

`"processing-failed"` - Payment has failed on the banking side;

`"refund-in-progress"` - Transition status of an order being failed in processing, only in case a card has been charged;

`"refunded"` - Refund is done;

`"email-confirmation"` - Email confirmation stage; 

`"crypto-sending"` -  Transaction cheque with a loading Transaction ID;

`"crypto-sent"` - Transaction cheque with Transaction ID;

`"settled"` - order has been completed;

`"crashed"` - manual check is needed (refund failed, settlement failed, rate has changed exceeding allowed percentage).
