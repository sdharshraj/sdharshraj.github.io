---
title: 'Migrating to, or creating new QnAMaker with microsoft bot framework '
comments: 'true'
description: This post will help you to create new QnAMaker service by registering
  a cognitive service from azure and then creating a new knowledge base in qna maker
  using the registered cognitive service.
date: 2018-05-20 12:50:00 +0530
tags: QnAMaker, new QnAMaker, QnAMaker with bot framework
---
In This post i will help you to create new QnAMaker service by registering a cognitive service from azure and then creating a new knowledge base in qna maker using the registered cognitive service. And also if you are migrating from old QnAMaker (which you are using as a dialog in microsoft bot framework) to new QnAMaker you need to do certain changes in you code to make it work otherwise you may find 400 error or 404 error etc.

So lets register a new QnA cognitive service from azure portal first.

In order to do that you need to go to the [Azure Portal](portal.azure.com "Azure Portal") and then search for QnAMaker service and there you can see the detail about QnAMaker service.

![](/uploads/2018/05/20/qnaAzure.png)

Or for a shortcut you can directly go [here ](https://portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker "QnAMaker service")to register [QnAMaker service](https://portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker "QnAMaker").

![](/uploads/2018/05/20/qnaAzure2.png)

Then you need to fill the necessary details like name, subscription etc.

Now you need to go to the [qnamaker.io](https://www.qnamaker.ai/Create "QnAMaker portal") in order to create a new knowledgebase. There you need to sign in with the same account as you have there in azure portal.

![](/uploads/2018/05/20/qnaPortal.png)