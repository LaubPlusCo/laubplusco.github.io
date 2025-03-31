---
author: ap
category:
  - analytics
  - sitecore
  - tips
cover:
  alt: EngagementPlan
  image: /wp-content/uploads/2014/05/EngagementPlan.png
date: "2014-06-13T20:36:28+00:00"
guid: http://laubplusco.net/?p=900
title: Sitecore Analytics rules with custom user properties
url: /sitecore-analytics-rules-custom-fields/

---
This post will show how to create an Engagement plan rule that uses custom fields. The idea is that the users have custom fields on their profile related to newsletter distribution groups.

The goal is to create an engagement plan where we have a rule condition saying something like this :

[![Rule](/wp-content/uploads/2014/05/Rule2-300x21.png)](/wp-content/uploads/2014/05/Rule2.png)

The engagement plan using this condition has a user in the initial state, a condition validator and a second state where the user will be put if the condition validates.  An email will be sent after the validation (in this case).

[![EngagementPlan](/wp-content/uploads/2014/05/EngagementPlan1-300x173.png)](/wp-content/uploads/2014/05/EngagementPlan1.png)

The "Unsubscribe Date" is a custom field set on the user profile. "Include time in comparison" is used to include or exclude the time when we compare the dates. That is because the date time is saved in Sitecore format "yyyymmddThhmmss" and for some fields the comparison may need to exclude the time and only compare the dates.

On the user profile in the core database the following custom fields are added:

[![fields](/wp-content/uploads/2014/05/fields-300x94.png)](/wp-content/uploads/2014/05/fields.png)

After defining the custom fields we need to set up the Sitecore rule under this item:

_"/sitecore/system/Settings/Rules/Definitions/Elements"_ [![SitecoreItemRule](/wp-content/uploads/2014/05/SitecoreItemRule.png)](/wp-content/uploads/2014/05/SitecoreItemRule.png)

The "TimeComparisonInclusion" and "UserCustomFields" subitems are just empty standard templates that will be used when editing the rule in the engagement plan Designer.

The text of the UserDateTimeFieldComparer rule is like this:

_"Where the User \[DateField,Tree,root={3A3D6165-A03D-4413-ACC7-1DE75F4E9C42}&setRootAsSearchRoot=true,custom date field\] \[operatorid,Operator,,compares to\] \[StartDate, DateTime,,date\] and \[IncludeTimeInComparison, Tree,root={7CF3EC01-4FEB-4181-8875-87A0818CEF70}&setRootAsSearchRoot=true,include time in comparison \]"_

where the " _{3A3D6165-A03D-4413-ACC7-1DE75F4E9C42}_" and " _{7CF3EC01-4FEB-4181-8875-87A0818CEF70}_" IDs are the " _TimeComparisonInclusion_" and " _UserCustomFields_" item IDs.

This is the Sitecore part of the whole rule. The code that actually performs the verification comes next.

In the UserDateTimeFieldComparerCondition class we have defined two enums that are representing the two constraints in the rule : "TimeComparisonInclusion" and "UserCustomFields".

```c#
internal enum CustomDateFields
  {
    SubscribeDate,
    UnsubscribeDate,
    LastProfileUpdateDate,
    FirstDistributionDate,
    LastDistributionDate,
    NextDistributionDate
  }

  internal enum TimeComparisonInclusion
  {
    IncludeTimeInComparison,
    ExcludeTimeFromComparison
  }
```

The  constructor instantiates two dictionaries that bind the two enum constraints to the sitecore items.

```c#
public UserDateTimeFieldComparerCondition()
    {
      Dictionary<string, CustomDateFields> customFieldsDictionary = new Dictionary<string, CustomDateFields>();
      customFieldsDictionary.Add("{E8FC7037-55C4-44F6-9509-226DF972F11C}", CustomDateFields.SubscribeDate);
      customFieldsDictionary.Add("{DACA57EB-FBDC-4F2D-8D5D-C678713BBD24}", CustomDateFields.UnsubscribeDate);
      customFieldsDictionary.Add("{70D6A55B-66CE-49AE-BFF0-1DC4DDE46979}", CustomDateFields.LastProfileUpdateDate);
      customFieldsDictionary.Add("{CFA221A1-2055-4EFE-A8C6-D793AE699AAF}", CustomDateFields.FirstDistributionDate);
      customFieldsDictionary.Add("{5FCED6CF-41FA-44F5-8FA8-BA4685B45350}", CustomDateFields.LastDistributionDate);
      customFieldsDictionary.Add("{51B13381-079A-42BF-96E9-F9263A669252}", CustomDateFields.NextDistributionDate);
      CustomFieldsSet = customFieldsDictionary;

      Dictionary<string, TimeComparisonInclusion> timeComparisonDictionary = new Dictionary<string, TimeComparisonInclusion>();
      timeComparisonDictionary.Add("{9EB42BEB-8D2D-4996-9D0E-FF804DAD683F}", TimeComparisonInclusion.IncludeTimeInComparison);
      timeComparisonDictionary.Add("{FEA6983B-3936-4A18-BBEB-E772860002A9}", TimeComparisonInclusion.ExcludeTimeFromComparison);
      TimeComparisonInclusions = timeComparisonDictionary;
    }
```

We also define some properties that are used to read the values of the inserted parameters in the rule.

```c#
public string StartDate
    {
      get;
      set;
    }

    public string DateField
    {
      get;
      set;
    }

    public string IncludeTimeInComparison
    {
      get;
      set;
    }
```

The execute method reads the parameters and performs the comparison.

```c#
protected override bool Execute(T ruleContext)
    {
      Assert.ArgumentNotNull(ruleContext, "ruleContext");

      if (string.IsNullOrEmpty(StartDate) || string.IsNullOrEmpty(DateField) || string.IsNullOrEmpty(IncludeTimeInComparison))
        return false;

      string customField;

      try
      {
        customField = CustomFieldsSet[DateField].ToString();
      }
      catch (ArgumentException exception)
      {
        Log.Error("Wrong custom field definition" + DateField, exception, GetType());
        return false;
      }

      string includeTimeInComparison;
      try
      {
        includeTimeInComparison = TimeComparisonInclusions[IncludeTimeInComparison].ToString();
      }
      catch (ArgumentException exception)
      {
        Log.Error("Wrong custom field definition" + DateField, exception, GetType());
        return false;
      }

      DateTime startDate = DateUtil.IsoDateToDateTime(StartDate);
      UserDateFieldComparer dateFieldComparer = new UserDateFieldComparer();
      return dateFieldComparer.Compare(GetOperator(), GetCustomFieldValue(customField), startDate, includeTimeInComparison.Equals(TimeComparisonInclusion.IncludeTimeInComparison.ToString()));
    }

    private DateTime GetCustomFieldValue(string customField)
    {
      User currentUser = Context.User;
      return DateUtil.IsoDateToDateTime(currentUser.Profile.GetCustomProperty(customField));
    }
```

Well, that was it.

Now you have a nice example of how to connect user profile custom fields to an engagement plan.
