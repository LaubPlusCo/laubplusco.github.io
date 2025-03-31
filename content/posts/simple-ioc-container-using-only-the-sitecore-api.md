---
author: alc
category:
  - best-practices
  - sitecore
date: "2015-04-07T20:58:25+00:00"
guid: http://laubplusco.net/?p=1445
title: Simple IoC container using only the Sitecore API
url: /simple-ioc-container-using-only-the-sitecore-api/

---
Lately there has been a lot of discussions on twitter about using different third party frameworks for Inversion of Control using Dependency Injection in Sitecore solutions.

The arguments in these discussions can be almost religious, either for or against using a specific third party IoC framework. There are two extremes, on one side the developers who have just found IoC and now evangelize it wonders to the world and then on the other side we have developers who are either scared of IoC or have seen the hell it is when IoC frameworks are misused.

Personally I am somewhere in the middle of these extremes.

I too was a shirt and tie IoC evangelist for a short period of time. It was back when I first worked with Microsoft Unity some 5-6 years ago. It amazed me how IoC could be twisted to make your code really expressive and easy to read using only very few lines of code.

I quickly discovered the downside on large solutions, maintaining the code became a living nightmare. Stepping into specific implementations was not directly available and when you haven't seen the code for some time or perhaps not written it yourself then you first had to untangle where the control over which types was. It was hard detective work almost each time something had to be changed. This is not good especially when the website does nothing more than some basic presentation.

Using third party IoC frameworks in general made our code less maintainable so we went away from using these if we did not have a very good reason at Pentia. Looking in the rear-view mirror it was a really tough and expensive experience we gained from our work with IoC containers but it is invaluable today.

Our approach now is to stay true to the Sitecore API and not introduce any third party frameworks for IoC except in the very rare cases where they really are the right tool for the job. And in these cases we keep them isolated and not as the foundation for the whole solution.

> In hindsight it seems wrong to create a tight coupling with a third party framework and all your components just to gain loose coupling between your own components.

Loose coupling can be achieved in many other ways than using Inversion of Control and when you really need the pattern in a Sitecore solution, it is easy to implement without introducing any third party framework such as Ninject, Windsor, Unity, etc.

A basic IoC implementation that cover 29 out of 30 use cases can be achieved with very few lines of code using the Sitecore API.

These lines of code and the pattern is used everywhere in Sitecore. You know it from all Sitecore pipeline processors, providers, event handlers and so on in configuration and SheerUI command types in items.

Dependency Injection in raw form is no more than a container in which you can insert mappings between a name and a type.

In Sitecore there are two apparent ways of keeping a container that can contain your mappings, either using configuration or items (or mongo if you want an x-IoC container).

## Using Sitecore configuration as DI container

For this simple example we use the Sitecore configuration section and Sitecore.Configuration namespace as our container.

To keep the example really simple lets consider the following simple configuration patch.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <typeMappings>
      <mapping name="ProductRepository" type="LaubPlusCo.IoC.ProductRepository, LaubPlusCo.IoC"/>
    </typeMappings>
  </sitecore>
</configuration>
```

We have created a new custom configuration element in the Sitecore configuration section called typeMappings that contain a mapping called ProductRepository.

Now in code we can write a simple generic TypeResolver that look like this:

```c#
namespace LaubPlusCo.IoC
{
  public class TypeResolver
  {
    public static T Resolve<T>(string typeName, object[] parameters = null)
    {
      var xpath = "typeMappings/mapping[@name='" + typeName + "']";
      return (T)(parameters == null ?
        Sitecore.Reflection.ReflectionUtil.CreateObjectFromConfig(xpath) :
        Sitecore.Reflection.ReflectionUtil.CreateObjectFromConfig(xpath, parameters));
    }
  }
}
```

If we have an interface for our ProductRepository that look like this:

```c#
public interface IProductRepository
{
  object Get();
  void Set(object product);
}
```

Then we can resolve the implementation defined in configuration like this:

```c#
private IProductRepository productRepository = TypeResolver.Resolve<IProductRepository>("ProductRepository");
```

In this example we implicitly require the type defined in the ProductRepository to implement to IProductRepository interface. This could have been defined in a custom configuration element instead.

We instantiate the type using the class called ReflectionUtil from the Sitecore API. This class has been around at least since Sitecore 5.0.

ReflectionUtil.CreateObjectFromConfig calls a method on ReflectionUtil called CreateObject(..). This method can be used directly with the typename if you want to create objects from types written in text fields on items or similar (mongo).

### Simple life time management

IoC frameworks typically have some degree of life time management so the mapped types are not instantiated each time they are resolved.

If you want your ProductRepository to be instantiated only once per process then you would wrap the resolving in a singleton pattern.

```c#
    private static IProductRepository _productRepository;
    public static IProductRepository ProductRepository
    {
      get { return _productRepository ?? (_productRepository = TypeResolver.Resolve<IProductRepository>("ProductRepository")); }
    }
```

 _This is typically a really bad idea to do in a web application if you want it to scale since static is per process._

A  common and good life time management requirement is to only instantiate the object once per thread or rather once per request in a web app.

This can be done like this:

```c#
    private const string ProductRepositoryRequestCacheKey = "ProductRepository";
    public static IProductRepository ProductRepository
    {
      get
      {
        var productRepository = (IProductRepository)HttpContext.Current.Items[ProductRepositoryRequestCacheKey];
        if (productRepository != null)
          return productRepository;
        productRepository = TypeResolver.Resolve<IProductRepository>("ProductRepository");
        HttpContext.Current.Items[ProductRepositoryRequestCacheKey] = productRepository;
        return productRepository;
      }
    }
```

What about speed then? That is actually a really good question. I will do a [speed comparison in a later blog post](/comparing-di-containers-and-sitecore-reflectionutil/ "Comparing DI frameworks with Sitecore ReflectionUtil"). I just had to get this one out quick to get me started on some blogging again. It has been some really slow two months for me.

What would you think is the fastest?

To resolve a type using MS Unity, Ninject or ReflectionUtil?

Share your thoughts in the comments below or tweet it [@AndersLaub](https://twitter.com/AndersLaub) #SitecoreIoC

_After posting I received some feedback about calling the frameworks and my implementation Inversion of Control and not Dependency Injection. This is true, it is IoC using Dependency Injection which is just one form of IoC. Read more [here](http://martinfowler.com/articles/injection.html) thanks for the feedback._
