---
author: ap
category:
  - lucene
  - sitecore
  - tips
date: "2013-10-29T18:57:51+00:00"
guid: http://laubplusco.net/?p=249
tag:
  - lucene
title: 'Lucene: Indexing DateTime from Sitecore and querying date ranges'
url: /lucene-indexing-datetime-sitecore-querying-date-ranges/

---
The title should be self explanatory. I need to index some sitecore items that represent some entities that are valid to the public between certain dates. The sitecore fields are date time fields looking like this.

[![Image](http://cook4passion.files.wordpress.com/2013/10/dates.png?w=254)](http://cook4passion.files.wordpress.com/2013/10/dates.png)

I needed a solution for indexing these values in a way that i can do range queries on them. I used the UnixTimeStamp for converting the DateTime field in a UnixTimeStamp field.

```c#
public static string ConvertToUnixTimeStamp(DateTime value)
    {
      var date = new DateTime(1970, 1, 1, 0, 0, 0, value.Kind);
      var unixTimestamp = Convert.ToInt64((value - date).TotalSeconds);
      return unixTimestamp.ToString(CultureInfo.InvariantCulture);
    }
```

Decoding UnixTimeStamp to DateTime :

```c#
public static DateTime ConvertToDateTime(double unixTimeStamp)
    {
      DateTime dtDateTime = new DateTime(1970, 1, 1, 0, 0, 0, 0);
      dtDateTime = dtDateTime.AddSeconds(unixTimeStamp).ToLocalTime();
      return dtDateTime;
    }
```

The fields would then be indexed as numbers and i could filter the results using a TermRangeQuery on the index like this:

```c#
TermRangeQuery trq = new TermRangeQuery(dateFieldName, UnixTimeStampConverter.ConvertToUnixTimeStamp(startDateTime), UnixTimeStampConverter.ConvertToUnixTimeStamp(endDateTime), true, true);
```

I think this solution is quite elegant and easy to use.
