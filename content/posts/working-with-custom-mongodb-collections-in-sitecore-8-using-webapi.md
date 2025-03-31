---
author: alc
category:
  - mongo
  - sitecore
  - sitecore-8
  - tips
  - webapi
date: "2015-01-24T17:01:01+00:00"
guid: http://laubplusco.net/?p=1409
title: Working with custom MongoDB collections in Sitecore 8 using WebApi
url: /working-custom-mongodb-collections-sitecore-8-using-webapi/

---
The introduction of the xDB to the Sitecore eXperience Platform offer a ton of new options for us Sitecore developers. As a bonus we suddenly have a data store for anything at any time instead of either trying to make the data fit into Sitecore items or give up and go with a custom database. Now we can just use mongo for almost anything.

In this post I will show how extremely simple it is to get and set data in a custom mongo collection.

We start out with creating a simple class called Comment that we will use as example:

```c#
public class Comment
{
  public ObjectId Id { get; set; }
  public string Name { get; set; }
  public Guid ContactId { get; set; }
  public string Email { get; set; }
  public string Subject { get; set; }
  public string CommentMessage { get; set; }
  public DateTime Date { get; set; }
}
```

Then we create a repository that can get and set Comment objects in mongo.

```c#
public class CommentRepository
{
  internal const string CollectionName = "comments";
  private readonly MongoCollection _commentsCollection;

  public CommentRepository(string connectionString)
  {
    _commentsCollection = GetCollection(connectionString, CollectionName);
    Assert.IsNotNull(_commentsCollection, "commentsCollection");
  }

  public IEnumerable<Comment> GetAll()
  {
    return _commentsCollection.FindAllAs<Comment>();
  }

  public Comment Get(ObjectId id)
  {
    return _commentsCollection.FindOneAs<Comment>(GetIDQuery(id));
  }

  public bool Set(Comment comment)
  {
    return _commentsCollection.Insert(comment).Ok;
  }

  public IEnumerable<Comment> GetByDateRange(DateTime startDate, int numberOfDays)
  {
    return _commentsCollection.AsQueryable<Comment>().Where(c => c.Date >= startDate && c.Date < startDate.AddDays(numberOfDays)).ToArray();
  }

  protected IMongoQuery GetIDQuery(ObjectId id)
  {
    return Query<Comment>.EQ(c => c.Id, id);
  }

  private static MongoCollection GetCollection(string connectionString, string collectionName)
  {
    var url = new MongoUrl(connectionString);
    return new MongoClient(url).GetServer().GetDatabase(url.DatabaseName).GetCollection(collectionName);
  }
}
```

The connection string to the mongodb containing the comments collection then need to be configured in connectionstrings.config:

```xhtml
<add name="comments" connectionString="mongodb://localhost/laubplusco_sitecore8_comments" />
```

Then we create a CommentApiController for getting and setting comments:

```c#
public class CommentApiController : ServicesApiController
{
  private readonly CommentRepository _commentRepository;

  public CommentApiController()
  {
    _commentRepository = new CommentRepository(ConfigurationManager.ConnectionStrings["comments"].ConnectionString);
  }

  [HttpGet]
  public IHttpActionResult Get(string id)
  {
    var objectID = new ObjectId(id);
    var comment = _commentRepository.Get(objectID);
    if (comment == null)
      return NotFound();
    return new JsonResult<Comment>(comment, new JsonSerializerSettings(), Encoding.UTF8, this);
  }

  [HttpGet]
  public IHttpActionResult GetByDate(DateTime date, int numberOfDays)
  {
    var comments = _commentRepository.GetByDateRange(date, numberOfDays);
    if (comments == null || !comments.Any())
      return NotFound();
    return new JsonResult<IEnumerable<Comment>>(comments, new JsonSerializerSettings(), Encoding.UTF8, this);
  }

  [HttpPost]
  public IHttpActionResult Set(string name, string email, string subject, [FromBody] string commentMessage)
  {
    var comment = new Comment
    {
      Name = name,
      Email = email,
      Subject = subject,
      CommentMessage = commentMessage,
      Date = DateTime.Now,
      ContactId = GetContactId()
    };
    if (_commentRepository.Set(comment))
      return Ok();
    return InternalServerError();
  }

  private Guid GetContactId()
  {
    return Tracker.IsActive ? Tracker.Current.Contact.ContactId : Guid.Empty;
  }
}
```

Finally we map the routes for the controller:

```c#
public class RegisterHttpRoutes
{
  public void Process(PipelineArgs args)
  {
    GlobalConfiguration.Configure(Configure);
  }

  protected void Configure(HttpConfiguration configuration)
  {
    var routes = configuration.Routes;
    routes.MapHttpRoute("CommentApi_Get", "sitecore/api/comments/get/{id}", new
    {
      controller = "CommentApi",
      action = "Get"
    });
    routes.MapHttpRoute("CommentApi_GetByDate", "sitecore/api/comments/getbydate/{date}/{numberOfDays}", new
    {
      controller = "CommentApi",
      action = "GetByDate"
    });
    routes.MapHttpRoute("CommentApi_Set", "sitecore/api/comments/set/{name}/{email}/{subject}", new
    {
      controller = "CommentApi",
      action = "Set"
    });
  }
}
```

That is it! It is so extremely easy to work with mongo db while still being completely reliable.

The Repository that I just created as an example could also have been implemented as a Sitecore.Services.Core.IRepository that has been introduced in Sitecore.Services so it could be used in an EntityService. Read this great [introduction post on EntityService](http://mikerobbins.co.uk/2015/01/06/entityservice-sitecore-service-client/) by [Mike Robbins](https://twitter.com/Sobek1985) if you haven't already heard about this already.

_A comment made by a visitor could also be stored directly on the Contact using the analytics API and not in a custom collection. This has only been made as an example and is not production ready code._
