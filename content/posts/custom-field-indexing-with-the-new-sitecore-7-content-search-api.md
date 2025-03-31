---
author: ap
category:
  - sitecore
  - sitecore-7.1
date: "2014-02-01T13:55:06+00:00"
guid: http://laubplusco.net/?p=624
tag:
  - indexing
title: Custom Field indexing with the new Sitecore 7 content search API
url: /custom-field-indexing-new-sitecore-7-content-search-api/

---
_Sitecore 7 comes with it's new Content Search API. The title of this post is self explanatory for what i'm about to share._ **Scenario**

I assume I have a website that contains information about different publishers books. These books are imported into Sitecore in XML format. I assume that on the website a user should be able to find content from these documents by performing a search. So I want to create an index that will index the contents of these books. Later I want to perform searches on Authors, Book names, Publishers, and clear text. A possibility is to index the pages of the books separately, so when the results are returned you know not only the book where the specific word or phrase has been found but also the exact page.

My XML looks like this.

```xhtml
<catalog>
   <book id="bk101">
      <author>Gambardella, Matthew</author>
      <title>XML Developer's Guide</title>
      <genre>Computer</genre>
      <price>44.95</price>
      <publish_date>2000-10-01</publish_date>
      <description>An in-depth look at creating applications
      with XML.</description>
   </book>
   <book id="bk102">
      <author>Ralls, Kim</author>
      <title>Midnight Rain</title>
      <genre>Fantasy</genre>
      <price>5.95</price>
      <publish_date>2000-12-16</publish_date>
      <description>A former architect battles corporate zombies,
      an evil sorceress, and her own childhood to become queen
      of the world.</description>
   </book>
</catalog>
```

This XML has been imported into a a Multiline Text Field on a Sitecore Item. (there can be many items that have catalogs XML imported into).

**Step1 - Configure the index**

Manually add a CustomIndexConfiguration.config file in App\_Config/Include folder. (\* it is mandatory that the Sitecore.ContentSearch.Lucene.DefaultIndexConfiguration.config file is before this file).

```xhtml
 <sitecore>
    <contentSearch>
      <configuration type="Sitecore.ContentSearch.LuceneProvider.LuceneSearchConfiguration, Sitecore.ContentSearch.LuceneProvider">
        <indexes hint="list:AddIndex">
          <index id="book_index" type="Sitecore.ContentSearch.LuceneProvider.LuceneIndex, Sitecore.ContentSearch.LuceneProvider">
            <param desc="name">$(id)</param>
            <param desc="folder">$(id)</param>
            <!-- This initializes index property store. Id has to be set to the index id -->
            <param desc="propertyStore" ref="contentSearch/databasePropertyStore" param1="$(id)" />
            <strategies hint="list:AddStrategy">
              <!-- NOTE: order of these is controls the execution order -->
              <strategy ref="contentSearch/indexUpdateStrategies/syncMaster" />
            </strategies>
            <commitPolicyExecutor type="Sitecore.ContentSearch.CommitPolicyExecutor, Sitecore.ContentSearch">
              <policies hint="list:AddCommitPolicy">
                <policy type="Sitecore.ContentSearch.TimeIntervalCommitPolicy, Sitecore.ContentSearch" />
              </policies>
            </commitPolicyExecutor>
            <locations hint="list:AddCrawler">
              <crawler type="Books.Infrastructure.BookCrawler, Books">
                <Database>master</Database>
                <Root>{110D559F-DEA5-42EA-9C1C-8A5DF7E70EF9}</Root>
              </crawler>
            </locations>
          </index>
        </indexes>
      </configuration>
    </contentSearch>
  </sitecore>
```

Note that my index name is _book\_index._

You can see that i have added a custom crawler called _BookCrawler_ which inherits from _SitecoreItemCrawler_ class. For this example I only had to override the _DoAdd_ method in your BookCrawler.

```c#
protected override void DoAdd(IProviderUpdateContext context, SitecoreIndexableItem indexable)
    {
      Operations = new CustomIndexOperations();

      if (indexable.Item.Template.BaseTemplates.FirstOrDefault(t => t.ID == Constants.Tempaltes.BooksData) == null)
        return;
      BookFactory bookFactory = new BookFactory();

      XDocument doc = XDocument.Parse(indexable.Item.Fields[Constants.Field.Books].Value);
      var catalog = doc.Element("catalog");
      foreach (XElement descendant in catalog.Descendants("book"))
      {
        var book = bookFactory.Create(indexable.Item, descendant);
        Operations.Add(book, context, index.Configuration);
      }
}
```

The _CustomIndexOperations_ class implements the _IIndexOperations_ interface. The only thing you need to implement is the _Add_ method. Mine looks like this:

```c#
public void Add(IIndexable indexable, IProviderUpdateContext context, ProviderIndexConfiguration indexConfiguration)
    {
      Document document = new Document();
      foreach (IIndexableDataField field in indexable.Fields)
      {
        string name = field.Name.ToLowerInvariant();
        string value = field.Value == null ? string.Empty : field.Value.ToString();
        document.Add(new Field(name, value, Field.Store.YES, Field.Index.NOT_ANALYZED));
      }
      context.AddDocument(document, new CultureExecutionContext(indexable.Culture));
    }
```

My _Book_ class implements the _IIndexable_ interface. In fact this class has, at this point, 2 utilities:

- It is used for indexing
- It is used for searching

To use the _Book_ class for indexing it must implement IIndexable interface.

To use it for searching the properties must be decorated with \[IndexField\].

```c#
[IndexField("Author")]
    public string Author { get; set; }

    [IndexField("Title")]
    public string Title { get; set; }

    [IndexField("Genre")]
    public string Genre { get; set; }

    [IndexField("Price")]
    public double Price { get; set; }

    [IndexField("PublishDate")]
    public string PublishDate { get; set; }

    [IndexField("Description")]
    public string Description { get; set; }
```

 **Searching**

Searching is the easy part. Getting all the Books from the index is done like this:

```c#
var index = ContentSearchManager.GetIndex("book_index");
      using (var context = index.CreateSearchContext())
      {
        var results =  context.GetQueryable<Book>().ToList();
        return results;
      }
```

To search for the books by author I do this:

```c#
public static IEnumerable<Book> GetByName(string name)
    {
      var index = ContentSearchManager.GetIndex("book_index");
      using (var context = index.CreateSearchContext())
      {
        var predicate = PredicateBuilder.True<Book>();
        predicate.Or(book => book.Author.ToLower().Contains(name.ToLower()));
        return context.GetQueryable<Book>().Where(predicate).ToList();
      }
    }
```

That is pretty much it. I hope this blog post helps some of you out there.

If you liked my post please comment. If you have questions, please ask them.
