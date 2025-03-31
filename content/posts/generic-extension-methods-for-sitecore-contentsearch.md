---
author: alc
category:
  - contentsearch
  - sitecore
  - tips
cover:
  alt: queryableextensions
  image: /wp-content/uploads/2014/11/queryableextensions.png
date: "2014-11-06T08:02:06+00:00"
guid: http://laubplusco.net/?p=1245
title: Generic extension methods for Sitecore ContentSearch
url: /generic-extension-methods-sitecore-contentsearch/

---
I would first like to mention that the code shown in this post is not originally mine. I've been using it quite often lately and made some tweaks to it which is why I think it deserves a quick blog post.

Some of the code originate from my colleague [Uli Weltersbach](https://twitter.com/UliWeltersbach), he wrote a blog post on [how to index base templates in Sitecore](http://reasoncodeexample.com/2014/01/29/indexing-base-templates-sitecore-7-content-search/).

The rest of the code I got from my colleague [Bo Breiting](https://twitter.com/bobreiting). He wrote this blog post about his [extension methods for Sitecore ContentSearch](http://thegrumpycoder.com/post/75297631359/extending-sitecore-contentsearch-with-iqueryable).

Bo's extension methods are great but they did not have a type declaration set explicitly. You would typically want to use them on a SearchResultItem which makes them look like this:

```c#
public static class QueryableExtensions
{
  public static IQueryable IsContentItem(this IQueryable<SearchResultItem> query)
  {
    var id = ItemIDs.ContentRoot.ToShortID().ToString().ToLowerInvariant();
    return query.Where(item => item["_path"].Contains(id));
  }

  public static IQueryable IsContextLanguage(this IQueryable<SearchResultItem> query)
  {
    return query.Where(item => item["_language"].Equals(Context.Language.Name));
  }

  public static IQueryable IsLatestVersion(this IQueryable<SearchResultItem> query)
  {
    return query.Where(item => item["_latestversion"].Equals("1"));
  }

  public static IQueryable HasBaseTemplate(this IQueryable<SearchResultItem> query, ID templateId)
  {
    var id = templateId.ToShortID().ToString().ToLowerInvariant();
    return query.Where(item => item["_basetemplates"].Contains(id));
  }
}
```

Here we have set SearchResultItem as type on the IQueryable parameter so the compiler can infer what implementation of Where to use (the one from IEnumerable or from IQueryable).

But this has an issue when you are using your own derived SearchResultItem class.

When you call the extension methods the return type of the IQueryable will be denominated to a SearchResultItem thus it will not build.

## An example

We have a derived SearchResultItem class as the simple one shown below.

```c#
public class MySearchResultItem : SearchResultItem
{
  [IndexField("MyText")]
  public string MyText { get; set; }
}
```

Then we get our IQueryable<MySearchResultItem>  from our SearchContext and try to call one of the extension methods.

```c#
public class MySearchResultItemRepository
{
  public static IEnumerable<MySearchResultItem> Get()
  {
    using (var searchContext = SearchIndexService.GetContextIndex().CreateSearchContext())
    {
      IQueryable<MySearchResultItem> query = searchContext.GetQueryable<MySearchResultItem>()
        .IsContentItem()
        .IsContextLanguage()
        .IsLatestVersion();
      var results = query.ToArray();
      return results;
    }
  }
}
```

This code will not build since there is no conversion between a MySearchResultItem and a SearchResultItem.

## The fix

To fix this we simply change the extension methods to take a generic type instead and constrain the generic type to be a SearchResultItem as shown below.

```c#
public static class QueryableExtensions
{
  public static IQueryable<T> IsContentItem<T>(this IQueryable<T> query) where T : SearchResultItem
  {
    return query.IsDescendantOf(ItemIDs.ContentRoot);
  }

  public static IQueryable<T> IsContextLanguage<T>(this IQueryable<T> query) where T : SearchResultItem
  {
    return query.Where(searchResultItem => searchResultItem.Language.Equals(Context.Language.Name));
  }

  public static IQueryable<T> IsLatestVersion<T>(this IQueryable<T> query) where T : SearchResultItem
  {
    return query.Where(searchResultItem => searchResultItem["_latestversion"].Equals("1"));
  }

  public static IQueryable<T> HasBaseTemplate<T>(this IQueryable<T> query, ID templateId) where T : SearchResultItem
  {
    var id = templateId.ToShortID().ToString().ToLowerInvariant();
    return query.Where(searchResultItem => searchResultItem["_basetemplates"].Contains(id));
  }

  public static IQueryable<T> IsDescendantOf<T>(this IQueryable<T> query, ID parentId) where T : SearchResultItem
  {
    return query.Where(searchResultItem => searchResultItem.Paths.Any(ancestorId => ancestorId == parentId));
  }
}
```

And now without changing the code in our repository class the code builds and the repository works.

## When would you use this?

This code comes in handy if you do not want to call GetItem() on the returned Documents to get the field values directly from the item.

The consideration to take regarding GetItem() is of course performance. If you have a large set of results then you can potentially save many calls to the database.

Quite often you anyway would like to call GetItem further down your execution path for example to get the item url through the LinkManager.

Working with the SearchResultItem without getting the item have the disadvantage that you only have the field values that is set to _stored.yes_ in your index.

How do you store values in your index then? You need to explicitly define what fields you like to store in your index by patching in the field names in the fieldMapping element in configuration and set the StorageType attribute to yes.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <contentSearch>
      <indexConfigurations>
        <defaultLuceneIndexConfiguration type="Sitecore.ContentSearch.LuceneProvider.LuceneIndexConfiguration, Sitecore.ContentSearch.LuceneProvider">
          <fieldMap type="Sitecore.ContentSearch.FieldMap, Sitecore.ContentSearch">
            <fieldNames hint="raw:AddFieldByFieldName">
              <field
				fieldName="mytextfield"
				storageType="YES"
				indexType="TOKENIZED"
				vectorType="NO"
				boost="1f"
				type="System.String"
				settingType="Sitecore.ContentSearch.LuceneProvider.LuceneSearchFieldConfiguration, Sitecore.ContentSearch.LuceneProvider" />
            </fieldNames>
          </fieldMap>
        </defaultLuceneIndexConfiguration>
      </indexConfigurations>
    </contentSearch>
  </sitecore>
</configuration>
```

This will make your indexes grow in size so choose the right implementation approach for your solution.

If you want to store all field values in your index then check out Uli's blog post about [creating a custom FieldMapper that index all Sitecore fields](http://reasoncodeexample.com/2014/01/30/indexing-all-fields-sitecore-7-content-search/).

If you want to use these extension methods then also remember to patch in a computed index field for the base templates as shown below.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <contentSearch>
      <indexConfigurations>
        <defaultLuceneIndexConfiguration type="Sitecore.ContentSearch.LuceneProvider.LuceneIndexConfiguration, Sitecore.ContentSearch.LuceneProvider">
          <fields hint="raw:AddComputedIndexField">
            <field fieldName="_basetemplates">Sitecore.ContentSearch.ComputedFields.AllTemplates,Sitecore.ContentSearch</field>
          </fields>
        </defaultLuceneIndexConfiguration>
      </indexConfigurations>
    </contentSearch>
  </sitecore>
</configuration>
```

That was it. If you have any nice extension methods for ContentSearch please share them in the comments below.
