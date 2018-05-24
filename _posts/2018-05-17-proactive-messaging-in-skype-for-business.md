---
title: Proactive messaging in Skype for business
comments: 'true'
description: description
date: 2018-05-17 19:04:22 +0530
tags:
- skype for business
---
In my last [post](https://sdharshraj.github.io/2018/04/29/sending-proactive-messaging-in-chatbot-using-microsoft-bot-framework.html "post") I have already shared an approach on how to send proactive message in chat bot.

So in this post i will just tell you what all things you require to send the proactive messaging in SKype for business.

So in the previous post we had some payload for the conversation reference and some other values which we were storing into the queue and then a queue trigger function invoke the Bot with ActivityTypes.Event.

But this time we'll take slightly different approach, we will create an custom api which will expect ConversationReference as requestbody and based on that ConversationReference we will initiate a dialog or simply a proactive message.