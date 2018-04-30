---
title: Sending proactive messaging in Chatbot using Microsoft bot framework
date: 2018-04-29 14:15:26 +0530
comments: 'true'
undefined:
- sending proactive messaging in chat bot
- proactive messaging
- microsoft bot framework
- proactive messaging using azure function and queue
---
**Proactive messaging** can be useful in many scenario for example

If you have to send some notification proactively to user, or if you have to initiate some conversation with user at some predefined time irrespective of whether user is currently interacting with the bot or not, or if you have to initiate a survey with user, you can send the survey proactively etc...

For sending proactive messaging we need to have Conversation reference which contains all the necessary information to initiate a proactive messaging with user.

The simplest way to send proactive messages is to create a Web app bot in [Azure portal](https://portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryFeaturedMenuItemBlade/selectedMenuItemId/home/searchQuery/bot/resetMenuId/)

![](/uploads/2018/04/29/New bot portal.PNG)

and then Just click create button and fill the necessary detail.

Now, at the time of choosing the template choose Proactive template and click select and create, after filling all the other fields.

![](/uploads/2018/04/29/bot.PNG)

So When you create a web app bot by choosing the Proactive template, behind the scene several Azure resources which are required are automatically created and added to your resource group. By default, these Azure resources are already configured to enable a very simple proactive messaging scenario.

So following are the resources which are created by portal automatically

* **Azure Storage** -> Used to create the queue which will have a default queue named 'bot-queue'. Azure storage is also used to store bot data in azure table storage.
* **Azure Function App**  -> A queueTrigger Azure Function that is triggered whenever there is a message in the queue. It communicates to Bot Service by using Direct Line. This function uses bot binding to send the message as part of the trigger’s payload. Our example function forwards the user’s message as-is from the queue.
* **Bot Service**  -> This will be your bot where you will write the logic how you want to handle the scenario. By default  it contains the logic that receives the message from user, adds the message to the Azure queue. And since we have a trigger function running, the moment something gets added to azure queue the queue trigger azure Function sends back the message it received via trigger's payload.

> _Now lets see the code .._

So now when you get the ActivityTypes.Message you simply send it to some dialog (ProactiveDialog) where you will add the user text and Conversation reference to the azure queue.

    if (activity.GetActivityType() == ActivityTypes.Message)
                {
                    await Conversation.SendAsync(activity, () => new ProactiveDialog());
                }

You need to maintain Message class which keeps ConversationReference  and Text.

     public class Message
        {
            public ConversationReference RelatesTo { get; set; }
            public String Text { get; set; }
            public bool IsTrustedServiceUrl { get; set; }
        }

ProactiveDialog.cs

    public async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> argument)
            {
                var message = await argument;
                    // Create a queue Message
                    var queueMessage = new Message
                    {
                        RelatesTo = context.Activity.ToConversationReference(),
                        Text = message.Text,
                        IsTrustedServiceUrl = MicrosoftAppCredentials.IsTrustedServiceUrl(message.ServiceUrl)
                    };
    
                    // write the queue Message to the queue
                    await AddMessageToQueueAsync(JsonConvert.SerializeObject(queueMessage));
    
                    await context.PostAsync($"You said {queueMessage.Text}. Your message has been added to a queue, and it will be sent back to you via a Function shortly.");
                    context.Wait(MessageReceivedAsync);
            }
    
            public static async Task AddMessageToQueueAsync(string message)
            {
                // Retrieve storage account from connection string.
                var storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["AzureWebJobsStorage"]); // If you're running this bot locally, make sure you have this appSetting in your web.config
    
                // Create the queue client.
                var queueClient = storageAccount.CreateCloudQueueClient();
    
                // Retrieve a reference to a queue.
                var queue = queueClient.GetQueueReference("bot-queue");
    
                // Create the queue if it doesn't already exist.
                await queue.CreateIfNotExistsAsync();
    
                // Create a message and add it to the queue.
                var queuemessage = new CloudQueueMessage(message);
                await queue.AddMessageAsync(queuemessage);
            }

So this will add the user text and ConversationReference to the azure storage queue and hence azure function will be triggered and it will send back the message it received via trigger's payload. And this time you will get ActivityTypes.Event where you can write your logic.

you can simply reply a text to the same conversation like below.

    if (activity.GetActivityType() == ActivityTypes.Message)
                {
                    await Conversation.SendAsync(activity, () => new ProactiveDialog());
                }
                else if (activity.Type == ActivityTypes.Event)
                {
                    IEventActivity triggerEvent = activity;
                    var message = JsonConvert.DeserializeObject<Message>(((JObject)triggerEvent.Value).GetValue("Message").ToString());
                    var messageactivity = (Activity)message.RelatesTo.GetPostToBotMessage();
    
                    var client = new ConnectorClient(new Uri(messageactivity.ServiceUrl));
                    if (message.IsTrustedServiceUrl)
                    {
                        MicrosoftAppCredentials.TrustServiceUrl(messageactivity.ServiceUrl);
                    }
                    var triggerReply = messageactivity.CreateReply();
                    triggerReply.Text = $"This is coming back from the trigger! {message.Text}";
                    await client.Conversations.ReplyToActivityAsync(triggerReply);
                }

Or you can transfer the control to some other dialog like survey, or some notification etc like below.

    else if (activity.Type == ActivityTypes.Event)
                {
                    IEventActivity triggerEvent = activity;
                    var message = JsonConvert.DeserializeObject<Message>(((JObject)triggerEvent.Value).GetValue("Message").ToString());
                    await StartSurvey(message);
                }
    
    private async Task StartSurvey(Message message)
            {
                var messageActivity = message.RelatesTo.GetPostToBotMessage();
    
                using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, messageActivity))
                {
                    var botData = scope.Resolve<IBotData>();
                    await botData.LoadAsync(CancellationToken.None);
                    var task = scope.Resolve<IDialogTask>();
    
                    IDialog<object> dialog = new LearnerDialog();
                    switch (message.Text)
                    {
                        case "new":
                            dialog = new newSurvey();
                            break;
                        case "old":
                            dialog = new oldSurvey();
                            break;
                    }
                    task.Call(dialog.Void<object, IMessageActivity>(), null);
    
                    await task.PollAsync(CancellationToken.None);
                    //flush dialog stack
                    await botData.FlushAsync(CancellationToken.None);
                }
            }

Done (Y)

You can simply publish your bot and can see it in action.

Once you have the ConversationReference of  any user you can send a proactive message any time. But it depends on client, like if you can send a message to the skype  anytime once you have ConversationReference but on webchannel you need an active conversation in order to send proactive message.

In order to send a proactive message to an skype user you can simply add the below message to the azure queue and it will trigger the survey proactively.

    {
      "relatesTo": {
        "user": {
          "id": "29:1Hdz_Pw*******Gh0AeWGh5CU-pCwRknjEDzxtGevxI",
          "name": "Harsh Raj"
        },
        "bot": {
          "id": "28:************,, 2-4ef4-9a38-22625ff08a38",
          "name": "SurveyBot"
        },
        "conversation": {
          "id": "29:1Hdz_PwYyxvU2H5Gh0AeWGh5CU-pCwRknjEDzxtGevxI"
        },
        "channelId": "skype",
        "serviceUrl": "https://smba.trafficmanager.net/apis/"
      },
      "text": "new",
      "isTrustedServiceUrl": true
    }

There are other ways also to send the proactive messag like using custom api which can be called with above payload.

you can check these links for more detail

[https://docs.microsoft.com/en-us/azure/bot-service/bot-service-concept-templates](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-concept-templates "Bot service template")

[https://docs.microsoft.com/en-us/azure/bot-service/nodejs/bot-builder-nodejs-proactive-messages](https://docs.microsoft.com/en-us/azure/bot-service/nodejs/bot-builder-nodejs-proactive-messages "https://docs.microsoft.com/en-us/azure/bot-service/nodejs/bot-builder-nodejs-proactive-messages")

**Thank you**