---
author: ap
category:
  - analytics
  - sitecore
  - sitecore-dms
  - tips
date: "2013-11-04T19:17:21+00:00"
guid: http://laubplusco.net/?p=254
tag:
  - lucene
title: 'Sitecore Analytics: Display content by relevance using profile cards'
url: /dms-display-content-relevance-using-profile-cards/

---
For this post to make sense I will need to create a user story. I will not try to create a standard scenario instead I will create a scenario that is easy to understand and easily applicable to this post.

The problem we want to solve is to present some content to the user that is relevant to the content that the user is looking at in any given time.

Scenario: You are building a site for the Law university. The site contains an archive of cases, that are categorized by the law branch they are part of : criminal, financial, tax, company, family law etc.

Let's assume that each case from these categories has a presentation page. (sorry i do not have screenshots but this is an imaginary scenario). Each of these cases has one or more profile cards assigned to it.

Somewhere on the presentation page of a case you want to show a list of cases that are similar to the one you are currently reading about.

What you need to do is to assign profile cards to the sitecore items that you want to filter by relevance. Then you simply get all the cases and sort them by relevance.

```c#
public static IEnumerable<ICase> GetRelatedCases(ICase currentCase)
    {
      return GetCases().Where(c => c.Item.ID != currentCase.Item.ID).OrderBy(c => GetRelevance(c, currentCase));
    }
```

This method gets all the cases that are related to the current one, and orders them by relevance. The GetRelevance method looks like this:

```c#
private static double GetRelevance(ICase caseToCompare, ICase currentCase)
    {
      ContentProfile[] caseToCompareProfiles = GetProfiles(caseToCompare.Item);
      ContentProfile[] currentCaseProfiles = GetProfiles(currentCase.Item);

      double matchResult = 0;

      foreach (ContentProfile currentCaseProfile in currentCaseProfiles)
      {
        ContentProfile compareToProfile = caseToCompareProfiles.FirstOrDefault(p => p.Key == currentCaseProfile.Key);
        if (compareToProfile == null)
        {
          continue;
        }
        double distance = GetProfileCardDistance(compareToProfile, currentCaseProfile);
        matchResult += distance;
      }
      return matchResult;
    }
```

One item can have multiple profile cards. We need to compare all of them:

```c#
private static ContentProfile[] GetProfiles(Item item)
    {
      Assert.IsNotNull(item, "Item cannot be null.");
      TrackingField field = new TrackingField(item.Fields["__Tracking"]);
      ContentProfile[] profiles = field.Profiles;
      return profiles;
    }
```

The get profile card distance returns the [Euclidian Distance](http://en.wikipedia.org/wiki/Euclidean_distance) between the two profile cards:

```c#
 private static double GetProfileCardDistance(ContentProfile contentProfile1, ContentProfile contentProfile2)
    {
      Pattern pattern1 = GetProfilePattern(contentProfile1);
      Pattern pattern2 = GetProfilePattern(contentProfile2);

      SquaredEuclidianDistance distance = new SquaredEuclidianDistance();

      return distance.GetDistance(pattern1, pattern2);
    }
```

```c#
private static Pattern GetProfilePattern(ContentProfile contentProfile)
{
 var patternMethod = new Func<IProfileData, Pattern>(contentProfile.Definition.PatternSpace.CreatePattern);

      return GetProfilePatternFromCache(contentProfile, patternMethod);
}
```

```c#
private static Pattern GetProfilePatternFromCache(ContentProfile contentProfile, Func<IProfileData, Pattern> patternMethod)
    {
      Pattern pattern;
      var cacheKey = String.Format(PROFILE_CACHE_KEY, contentProfile.ProfileID);

      if (HttpContext.Current != null)
      {
        pattern = HttpContext.Current.Items[cacheKey] as Pattern;
        if (pattern != null)
        {
          return pattern;
        }
      }
      pattern = patternMethod(contentProfile);

      if (HttpContext.Current != null)
      {
        HttpContext.Current.Items.Add(cacheKey, pattern);
      }
      return pattern;
    }
```

I have written this code about two years ago and only now i share it so, if this post is not clear enough i will try to make a demo site and post code and sitecore package.
