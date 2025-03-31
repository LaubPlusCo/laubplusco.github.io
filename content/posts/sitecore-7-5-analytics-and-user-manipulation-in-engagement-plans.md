---
author: ap
category:
  - analytics
  - sitecore
date: "2014-11-05T13:14:05+00:00"
guid: http://laubplusco.net/?p=1215
title: Sitecore 7.5 Analytics and user manipulation in Engagement plans
url: /sitecore-7-5-analytics-api-xdb-api/

---
I have recently had the interesting and challenging task of upgrading a Sitecore 7.2 installation to the newest Sitecore 7.5.  In the project we were extensively using Analytics code which had to be updated. That is where the fun began.

In our solution we had to manipulate users in engagement plans. Users had to be added, removed or moved between engagement plan states from the code. In order to do that we had to change the code to adapt it to the new API. Here are the changes.

**1\. Add a Sitecore user to an engagement plan.** _Before_

```c#
VisitorManager.AddVisitor(user.Name, engagementPlanIdStartState);
```

 _Now_

It can be done in two different ways. Assuming you have a list of User Names that you want to add to an engagement plan you can do it this way.

```c#
List<string> contactNames = new List<string> { user.Name };
bool addVisitor = AutomationContactManager.AddContacts(contactNames, engagementPlanIdStartState);
```

The idea behind this was to perform a bulk-import of users/contacts into an Engagement Plan from an external source. For other scenarios we have to use the new API that adds Contacts to the Engagement Plan using their ContactId.

```c#
Tracker.Current.Session.Identify(UserNameText.Text);
var managerInContext = Tracker.Current.Session.CreateAutomationStateManager();
managerInContext.EnrollInEngagementPlan(EngagementPlanId, StateId);

```

A great help in how do I identify my user was this blog [post](http://goo.gl/SGduqD) by Nick Wesselman.

After the first line of code the Contact has been identified. If it is a new Contact and does not exist in the Contacts Collection in xDB it will be saved in the shared session state.

If it exists in the xDB then it will be locked to the shared session state.

The third line will put the Contact Entry in the Engagement plan (in the shared session state, thus no changes in xDB). Only when the xDB is synchronized the results will be visible and the Contact will exist in the Engagement Plan state.

**2. Removing a user from an engagement plan** _Before_

```c#
VisitorManager.RemoveVisitor(userName, stateId);
```

 _Now_

```c#
Tracker.Current.Session.Identify(UserNameText.Text);
var managerInContext = Tracker.Current.Session.CreateAutomationStateManager();
managerInContext.RemoveFromEngagementPlan(EngagementPlanId));
```

 **3\. Moving a contact from an engagement plan state to another one** _Before_

```c#
VisitorManager.MoveVisitor(userName, stateId, engagementPlanStartState);
```

 _Now_

```c#
Tracker.Current.Session.Identify(UserNameText.Text);
var managerInContext = Tracker.Current.Session.CreateAutomationStateManager();
managerInContext.MoveToEngagementState(EngagementPlanId, StateId);
```

The explanation related to user identification and the whole process described in the Add action applies to the remove and move user actions.

I hope that this post will save someone some time. Ask away and if I can I will answer.
