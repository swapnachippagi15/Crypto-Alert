# Crypto-Alert


![pxfuel.com.jpg](https://cdn.steemitimages.com/DQmWK666zGg6sdm9MJNcyYdbEn2YKXrkCzLxauFZuUzwVE2/pxfuel.com.jpg)



Last June I took a loan from celsius.network because of some financial aid for my family. Previously I never bothered about Bitcoin price as I was HODLER of it, but as soon as I took the loan I was more scared even there was a small price hiccup. 
 

![InkedWhatsApp Image 2021-07-31 at 9.43.55 PM_LI.jpg](https://cdn.steemitimages.com/DQmXYVXkZE7w3uZcAHirb2EyUhpi2BEbYBKzKRnCGitmARV/InkedWhatsApp%20Image%202021-07-31%20at%209.43.55%20PM_LI.jpg)

 
>REFERAL ALERT 
If you would like to take loan against your crypto give celsius a chance,  use this signup promo https://celsiusnetwork.app.link/149806af89 you get $40 and I get $40  , more about it [here](https://celsius.network/download-app#referral)

As a  result I googled for the online alerts for bitcoin price ,then I found cryptocurrencyalerting.com , but there is a catch I don't get a free phone call alert service , instead got only email. For outside of US and Canada you need to have a pro plan to get either a call or a sms.So I thought why not create a cron job and make a call when the price met the conditions.

# let's discuss about Tech Stack

* I chose NodeJS to work as cron job, which will constantly will query the bitcoin price from an API and check whether the conditions are met .
* For the price of bitcoin I chose the  [Coingecko API](https://www.coingecko.com/en/api). 
* In order to make a phone call programmatically , I chose [twillo ](https://www.twilio.com/) . You can have a trail account without a credit card and they provide you some free credits with which you can buy a number and spend minutes/calls made , [more about here](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)

# DEPLOYMENT STACK 

[Deta.sh](https://www.deta.sh/) is a free cloud provider where we run our cron job. There is no paid plan yet in deta.sh, so you can have your code run at free of cost.Feel free to explore more free hosting and service providers from [Free-for-dev](https://free-for.dev/#/)

# LET'S START CODING YOUR CRYPTO_ALERT

### 1. Creating an account
First create a free account at deta.sh, once you had the account it will be directed automatically to this [web.deta.sh](https://web.deta.sh/)  endpoint.
### 2. Setting up local Development 
Follow the instruction provided at deta.sh docs [here](https://docs.deta.sh/docs/cli/install) for installing and setting up based on your OS.Once you setup the CLI then login to your deta.sh account from the terminal , more about that is [here](https://docs.deta.sh/docs/cli/install#logging-into-deta-via-the-cli)

### 3. Deploy a sample CRON job
Now go through the documentation provided [here](https://docs.deta.sh/docs/tutorials/cron-guide) to simply create a cron job running in you dashboard.This basic flow will give you an over view about the platform, till now we haven't gone through our code yet, feel confortable using deta.sh as it's a great platform for the prototypes like this.

***name your sample app as BITCOIN_ALERT while creating,, because we later change this sample code given at `index.js` to our logic***
 

> minimum time interval is 1 min while running the cron job

### 4. Make a Twilio account
> REFERAL ALERT 
use the this referal link [www.twilio.com/referral/eES7kf](https://www.twilio.com/referral/eES7kf) during sign up to get $10 and I get $10 

Go trough this [link](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account) , for the limits and description about your free trail account and how to make one if not created yet.

> In trail account you need to verify your phone number if you would like to recieve a call or sms.once you verify you will be able to recieve calls that made from code.

After your signup you will be redirected to the [twilio console ](https://www.twilio.com/console).
#### 5. Create a new twilio project
learn how to make one [here](https://support.twilio.com/hc/en-us/articles/360011177133-View-and-Create-New-Projects-in-Twilio-Console).

#### 6. Get your Account SID and Auth token
In your dashboard you will get Account SID and auth token which are needed to make a call/ send a sms from the code.

![Twillio.png](https://cdn.steemitimages.com/DQmYr8pdMCDkTZwJT599frDy4Z31RjvRjr39va7NWU3rbEE/Twillio.png)


### 7. Setting up Envoirment variable [.env]
All the values that are required in code are provided from the file called `.env`.This file holds your API keys and Auth tokens and other data needed for the programme.Please make you keep this file sensitve, and do not upload files to a public github repo.You can get this file [here](https://raw.githubusercontent.com/swapnachippagi15/Crypto-Alert/main/.env)

`.env`
```
PROJECT_KEY=<Deta project key>
DETABASE=<Database name>
MARGIN_CALL=<Your bitcoin margin call price in USD>
LIQUIDATION_CALL=<Your bitcoin liq call price in USD>
TWILIO_SID=<your twillo sid>
TWILIO_AUTH_TOKEN=<your twillo auth token>
TWILIO_PHONE_NUMBER=<Twillo phone number with country code>
TWILIO_MESSAGE_LIQ=Bitcoin price is at liquidation call
TWILIO_MESSAGE_MARGIN=Bitcoin price is at margin call
PHONE_NUMBER_WITH_COUNTRY_CODE=<Phone numbet to which you want to recieve call to, with Countrycode>
```
> There should not be space after equals sign
SECRET=123 //no space after equals
SECRET= 123 //space after equals

### 8.Applying our logic
In previous step [3] you made a deta project which have a sample code at the `index.js` file.Now all you need to do is to replace the entire source code/content  of the file to the node.js code below.I made comments in the code itself to give you an idea what that piece of logic is trying to do.

#### 8.1 Updating index.js file

```
const { app, Deta } = require("deta");
const axios = require("axios");


// add your Project Key
const deta = Deta(process.env.PROJECT_KEY);
// 
// Envorment Varaibles
const db = deta.Base(process.env.DETABASE);
const accountSid = process.env.TWILIO_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;
const client = require("twilio")(accountSid, authToken);
const marginCall = process.env.MARGIN_CALL;
const liquidationCall = process.env.LIQUIDATION_CALL;
const PhoneNumber=process.env.PHONE_NUMBER_WITH_COUNTRY_CODE;

let btcprice = 0;

// define a function to run on a schedule
// the function must take an event as an argument
app.lib.cron(async (event) => {
  console.log("CRON running @ " + new Date().toLocaleTimeString());
  const resp = await getBitcoinPrice();
  const data = resp.data;
  console.log("Current BTC price ===>", data.bitcoin.usd);
  btcprice = parseInt(data.bitcoin.usd);
  //btcprice = 23000; this variable holds the btc price at that moment

  // hascalled is the key in the database, which stores if phone call made for liquidation/Margin conditions when the predefined criteria met.
  // Suppose if BTC price met Margin condition then a call will be made , once its made,the code is not going 
  // to make a another call in the very next run even the condition satisfied.

// querying the DB for the key `hascalled`
  let hasCalled = await db.get("hascalled");

  console.log("values from db for the key named hascalled is  -->", hasCalled);
  if (hasCalled == null) {
    // Runs for the very first time

    // initally set margin and liq to false for the key 'hascalled' in db
    const insertedkey = await db.put(
      { margin: false, liq: false },
      "hascalled"
    );

    hasCalled = insertedkey;
  }

  //_BTCValidation_Block_Margin
  if (hasCalled.margin == false) {
     
    // This block will get executed only if there was no liquidation call not made yet
    await _BTCValidationMargin();
  }

  //_BTCValidation_Block_Liquidation
  if (hasCalled.liq == false) {
     // This block will get executed only if there was no liquidation call not made yet
    await _BTCValidationliquid();
  }


});

 
const _BTCValidationMargin = async () => {
  if (btcprice < marginCall && btcprice > liquidationCall) {
 
    await getCallfromTwillo(process.env.TWILIO_MESSAGE_MARGIN, PhoneNumber);

    const updateState = await db.update({ margin: true }, "hascalled");
    console.log("DB UPDATED For Margin --->", updateState);
  }
};

const _BTCValidationliquid = async () => {
  if (btcprice < liquidationCall) {
    
    await getCallfromTwillo(process.env.TWILIO_MESSAGE_LIQ, PhoneNumber);

    const updateState = await db.update({ liq: true }, "hascalled");
    console.log("DB UPDATED For Liquid --->", updateState);
  }
};

const getCallfromTwillo = async (msg, _to) => {
  const call = await client.calls.create({
    twiml: `<Response><Say>${msg}</Say></Response>`,
    to: _to,
    from: process.env.TWILIO_PHONE_NUMBER,
  });
  console.log(call.sid);
 
};

const getBitcoinPrice = async () => {
  try {
    return await axios.get(
      "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=USD"
    );
  } catch (error) {
    console.error(error);
  }
};

module.exports = app;


```

#### 8.2 Updating Project dependencies
Our Logic code depends on the two NPM dependencies , `AXIOS` and `TWILIO`.In order to install apply the following commands in the terminal.

```
npm i axios twilio
```

### 9.Finally deploying our logic

open the terminal at the root directory and apply the following command to deploy the local source code to the remote
```
deta deploy
```
learn more about deploy [here](https://docs.deta.sh/docs/cli/commands#deta-deploy).Whenever you change the 


### 10. Check Visor for logs
Deta.sh provides the `Visor` functionality which is a logger kind for the production code.Learn more about visor [here](https://docs.deta.sh/docs/cli/commands#deta-visor)


![DetaDashboard.png](https://cdn.steemitimages.com/DQmPy9gHX3ncYZHV1BjPET3Qkv2biHSW6JJ37YZAGhHHjbf/DetaDashboard.png)

### 11. Project Structure 

📦BITCOIN_ALERT
 ┣ 📂.deta
 ┃ ┣ 📜prog_info
 ┃ ┗ 📜state
 ┣ node_modules
 ┣ 📜.env
 ┣ 📜index.js
 ┣ 📜package-lock.json
 ┗ 📜package.json
 
 
 













