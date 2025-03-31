---
author: alc
category:
  - errors
  - sitecore
  - sitecore-ecm
cover:
  alt: queuedistinct
  image: /wp-content/uploads/2014/03/queuedistinct.png
date: "2014-03-07T12:11:40+00:00"
guid: http://laubplusco.net/?p=778
title: How to stop Sitecore ECM from spamming duplicate recipients
url: /stop-sitecore-ecm-spamming-duplicate-recipients/

---
In the current version of ECM the same mail message will be sent several times to the same user if the user has registered several times to the same target audience with a different user but the same email.

The issue is really that the email recipients are not filtered by distinct emails before the newsletter is dispatched.

Personally I would have thought that ECM ensured that emails was distinct before it dispatches the mail but this is not the case

To our luck it is easy to fix even though the fix overrides some standard functionality with modified code based on reflection.

## Make ECM email recipients distinct

When dispatching a mail message ECM uses the pipeline called _DispatchNewsletter_ found in the _Sitecore.EmailCampaign.config_ file.

The messages are placed in queue for being sent in the processor called _Sitecore.Modules.EmailCampaign.Core.Pipelines.DispatchNewsletter.QueueMessage_.

Using reflection the processor code looks like this:

```c#
  public class QueueMessage
  {
    public void Process(DispatchNewsletterArgs args)
    {
      if (args.IsTestSend || args.SendingAborted || args.DedicatedInstance)
        return;
      if (!args.Queued)
      {
        try
        {
          args.ProcessData.State = SendingState.Queuing;
          using (new SecurityDisabler())
            AnalyticsHelper.QueueMessage(args.Message, args.Message.SubscribersNames, args.TotalNumReceivers);
        }
        catch (Exception ex)
        {
          args.AbortSending(ex, true);
        }
      }
      else if (new MessageStatistics(args.Message).QueuedSecondLevel == 0)
        this.MoveToSecondLevel(args.Message);
    }

    private void MoveToSecondLevel(MessageItem message)
    {
      Item contentDbItem = ItemUtilExt.GetContentDbItem(message.PlanId);
      if (contentDbItem == null)
        throw new EmailCampaignException("An engagement plan for the '{0}' message could not be found.", new object[1]
        {
          (object) message.ID
        });
      else
        AnalyticsHelper.SetAutomationStateNumber(AnalyticsHelper.GetPlanState(contentDbItem, "Recipient Queued").ID.ToGuid(), 0, 100);
    }
  }
```

Notice the highlighted line, it is the value _args.Message.SubscribersNames_ which we want to make distinct on their emails.

The _args.Message.SubscribersNames_ property is read-only so we cannot modify it in a processor before _QueueMessage_ instead we need to override the whole processor which is a bit of a bore if Sitecore chooses to update it later on.

Another fun fact is that _args.Message.SubscribersNames_ actually contain the subscriber user names. So to make these distinct on email we need to load in all the user profiles and then make them distinct by email and finally return the filtered user names.

```c#
  public class QueueMessagesDistinct
  {
    public void Process(DispatchNewsletterArgs args)
    {
      if (args.IsTestSend || args.SendingAborted || args.DedicatedInstance)
        return;
      if (!args.Queued)
      {
        try
        {
          args.ProcessData.State = SendingState.Queuing;
          using (new SecurityDisabler())
          {
            var subscriberNames = GetDistinctContacts(args.Message.SubscribersNames);
            var totalRecipients = subscriberNames.Count;
            AnalyticsHelper.QueueMessage(args.Message, subscriberNames, totalRecipients);
          }
        }
        catch (Exception ex)
        {
          args.AbortSending(ex, true);
        }
      }
      else if (args.Message.MessageType == MessageType.Trickle || new MessageStatistics(args.Message).QueuedSecondLevel == 0)
        MoveToSecondLevel(args.Message);
    }

    private List<string> GetDistinctContacts(IEnumerable<string> subscriberNames)
    {
      var contacts = subscriberNames.Select(Membership.GetUser);
      var emails = contacts.Select(s => s.Email.ToLowerInvariant()).Distinct();
      return emails.Select(Membership.GetUserNameByEmail).ToList();
    }

    private void MoveToSecondLevel(MessageItem message)
    {
      var contentDbItem = ItemUtilExt.GetContentDbItem(message.PlanId);
      if (contentDbItem == null)
        throw new EmailCampaignException("An engagement plan for the '{0}' message could not be found.", new object[1]
        {
          message.ID
        });
      AnalyticsHelper.SetAutomationStateNumber(AnalyticsHelper.GetPlanState(contentDbItem, "Recipient Queued").ID.ToGuid(), 0, 100);
    }
  }
```

The only consideration in using this approach is tracking since some recipients will be filtered out but then again the email should be the unique identifier and not the user name.

I cannot really understand why ECM always rely on users, it should always be the email which is used as a unique identifier and not Sitecore users. If the email then could be mapped to a user then this information could be used. As it is implemented now the same user can exist in several domains which smells bad.

I hope the ECM team will read this post and include this fix in later versions of ECM. It seems very unprofessional for a company to send the same newsletter twice or more to the same user. There might be some cases where you want this behavior (even though I can't think of any). The solution could then be to have a checkbox on the mailmessage item determining if it only should be dispatched to distinct emails.

Please share.
