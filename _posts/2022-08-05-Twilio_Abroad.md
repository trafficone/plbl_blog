---
layout: post
title: "Twilio Abroad"
date: 2022-08-05
author: Jason Schlesinger
tags: phone, software, Japan
---
When I moved abroad, I decided to keep my US number. It made sense to me to port my number over to Twilio, so I can keep it without needing to keep it on me.

Twilio charges about $1/month for each phone number, and a few cents for SMS and calls. They're a service to help offload business' telephone needs, but as an individual the prices are eminently reasonable.
## Twilio Setup
If you don't have a Twilio account, you're going to need to sign up for one at <https://www.twilio.com/try-twilio>
### Phone Number 
Twilio has a form you can print out to port your number, you will need to provide them with your information, as well as information given to you by your provider. 
Voice calls -> SIP Inbound TwiML Bin
Text messages -> SMS Handler Function 
### TwiML Bin
Twilio will dial my SIP phone for 30 seconds. If no answer, it will pull TwiML from the **action** URL.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Dial timeout="30" action="Failover URL">
    <Sip><!-- SIP URI user@yourdomain.sip.twilio.com --></Sip>
  </Dial>
</Response>
```
My Failover TwiML is below, but I also push a notification when the URL is loaded.
This is accomplished using *functions*
```xml
<Response>
    <Say>Please leave a message. Beep.</Say>
    <Record timeout="30" transcribe="true"/>
</Response>
```
### SIP Phone
Twilio supports Programmable SIP Domains that you can connect a soft phone to for incoming/outgoing calls. 

Before creating your SIP Domain, create your credentials in a new Credential List. 

Then create your SIP domain, pick a URI, and add your credentials to the to the credentials list. Then, if you want, specify how you want to handle incoming calls. These will be SIP calls only, but I just forward them on to my softphone like everything else.

Enable SIP registration and again put in your credential list.

Now you're done!

### Functions
If you're familiar with [AWS Lambda](https://aws.amazon.com/lambda/) then Twilio functions are a Twilio wrapper on top of it. When a message, call, or other Twilio trigger comes in, it fires off a Node.js function to handle the trigger. I use Pushover to forward my SMS messages to my mobile phone, but you can do anything.

My Function is very barebones, I essentially copied the example and added a call to Pushover.
```javascript
const Pushover = require('node-pushover');

exports.handler = function(context, event, callback) {
  const APPID = process.env.PO_APPID;
  const USERKEY = process.env.PO_USERKEY;
  var po = new Pushover({ token: APPID });
  const incomingMessage = event.Body;
  const fromNumber = event.From;
  console.log("Message from " + fromNumber + " forwarded");
  let twiml = new Twilio.twiml.MessagingResponse();
  // A callback function is defined:
 return po.send(USERKEY, 'SMS_Forwarded', 'From ' + fromNumber + ': ' + incomingMessage, function (err, res) {
    if(err){ 
        console.log(err);
        return;
    }
    // This callback is what is returned in response to this function being invoked.
    return callback(null, twiml);

  });

}
```

## Setup Other Software
I use Pushover to receive notifications, and MizuDroid as my softphone to connect to SIP. For iPhone users SessionTalk Softphone seems to be highly rated, but I've never used it.

### Pushover 
I love using Pushover, for a one-time license fee of just $5 I can send as many notifications to my phone as I desire. I'm a big fan of [ntfy](https://github.com/dschep/ntfy) for long running shell commands, and it's nice to just get a ping on my phone when they finish.

Sign up, get a user key, and create an API key for your Twilio handler. API keys get a free 10,000 notifications/month, so you'll likely not go over it.

### Softphone
You should be able to point your softphone to your sipdomain.sip.twilio.com and register using the credentials from your Credential List.

## Concerns
Twilio isn't designed for regular folks to muck about on, there's some pretty serious regulations regarding handling of sensitive data, and contacting people without prior affirmative consent. This varies from region to region, but this is the reason I don't discuss outbound 
