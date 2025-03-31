---
author: alc
category:
  - modular-architecture
  - sitecore
  - sitecore-8
date: "2015-11-30T23:28:23+00:00"
guid: http://laubplusco.net/?p=1538
title: The groundbreaking Sitecore Habitat
url: /the-groundbreaking-sitecore-habitat/

---
Sitecore released a new demo site late September called _[Sitecore Habitat](https://github.com/Sitecore/Habitat)_.

There is nothing new in Sitecore supplying a demo site but this one is nothing less than groundbreaking, it is a real **revolution** in the way that Sitecore teaches developers to work with their product.

In the latter days Sitecore would never explain how to structure your solution. They would merely supply limited demos showing how different functionality can be achieved using the Sitecore API. Nothing about how to structure a large solution that has to be easy to maintain over time.

This has now changed. Sitecore Habitat introduces an architectural approach called **Modular Architecture**.

## The origin of Modular Architecture

Modular Architecture is primarily derived from the work of Uncle Bob ( [Robert C. Martin](http://www.objectmentor.com/omTeam/martin_r.html)). It is an architectural approach that was formalized for Sitecore solutions at [Pentia](http://pentia.net/) where it has been in use since around 2009.

The current version of the architecture, that contain the concept of layers, is used in all solutions made by Pentia since 2012 where we referred to the architecture as _**Component Architecture**_.

The principles in what we called component architecture and what now is called Modular Architecture are the same. For a reference on component architecture see my [slides from SUGCON 2015 on component architecture](http://www.slideshare.net/AndersLaubChristoffe/sugcon-2015-anderslaubchristoffersen-49425398) _Is Modular == Component?_ _**Yes**_, in this case, it is just a case of naming but I bet there is a good reason for this.

The name component architecture would often confuse developers who was not acquainted with the architecture. The word component was understood as a single presentation, **_rendering_** in Sitecore terms, and not as a full **feature** of the solution which was what the name actually intended.

This lead to unfortunate misunderstandings and an unnecessary extra effort had to be put into explaining the meaning of the word component before we could get down to speaking about the important stuff.

Uncle Bob refer to the architectural principles as "Principles of Package and Component Design" in his book [Agile Principles. Patterns and Practices in C#](http://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258) where he writes

> _The term package has been overloaded with many meanings in software. For our purposes, we focus on one particular kind of package, often called a component. A component is an independently deployable binary unit. In .NET, components are often called assemblies and are carried within DLLs_ _Agile Principles, Patterns and Practices, Robert C. Martin, chapter 28 - Packages and components_

Nowadays the word component has been overloaded with many meanings in web development so dragging it a level up and calling it Modular Architecture?

Yeah, that makes sense, I can live with that name even though it will take some time to get used to not calling it component architecture :)

A component, or now rather a module in Modular Architecture, is a feature in the solution and not just a single assembly or VS Project. A module typically consist of template and rendering items, configuration, tests and so forth along with the assembly. With this in mind then the name Modular Architecture makes a lot of sense.

The name, although important, is still secondary to the understanding of the principles behind it.

## Why follow Modular Architecture?

Basically to have more time to work on fun stuff. That is at least my favorite argument. The experience at Pentia has shown that the complexity of support issues goes down significantly so the time spent on support is at an absolute minimum. And I for one rarely enjoy working on support issues, I want to work on new stuff and I have more time for that now. The general quality and maintainability of our solutions ensures that we only work on support cases at Fridays on most of our solutions and Fridays are even only half a working day due to meetings and our play time. Some Fridays there aren't even enough support cases to go around for everyone so we get to work on projects.

Basically we have managed to keep maintenance costs down to a minimum on our own solutions even though maintenance costs still does grow some over time.

The minute you start developing a solution the clock starts ticking and you know that some day in the future the solution will have to die, that is just a fact of life. It starts to build up technical debts from day one due to different decisions taking during development to meet the requirements. The more time pressure, the more likely developers are to take bad decisions that adds technical debt. So on a solution where things go pear-shaped this can end in a vicious cycle where more and more technical debt adds more and more pressure which in turn adds more debt.

One day it will even be more expensive to maintain the solution and add new features than it will be to replace it completely with a new one.

In the worst cases I have seen this point was reached even before the solution was delivered to the customer. I guess you could call it _stillborn solutions_ where development was deadlocked due to poor or completely lacking architecture and sometimes also incompetent implementation partners.

![stillborn_graph](/wp-content/uploads/2015/11/stillborn_graph.jpg)

I have unfortunately witnessed quite a few of these solutions in my role as a professional service consultant at Pentia. It is a real shame since these solutions brings down the reputation and good name of Sitecore and has huge unnecessary costs for everyone involved.

For us developers such solutions are living nightmares that probably has brought many good people down with stress, caused divorces, broken families, addictions and probably also deaths.

Luckily most solutions are not stillborn. They live for quite some time but

> Due to changes in both technology and business requirements, all code will decay over time

The goal of modular architecture or any good solution architecture is basically to keep the maintenance costs down over time even though the costs will increase slightly no matter what, that is just a fact.

![comparison](/wp-content/uploads/2015/11/comparison.jpg)

Modular Architecture ensures this goal by following some simple principles that preaches low coupling between modules in your code base. Basically a healthy dependency flow. The low coupling ensures that decayed code easily can be replaced without impacting the whole code base. _More on these principles in later posts._

The methodology that the architecture also preaches ensures consistency and easy maintainability. At Pentia we do not have dedicated supporters for specific solutions, _no lone heroes_, everyone can potentially work on any of the in-house solutions each and every support Friday. This flexibility has huge advantages for both us and our customers.

In some of our original training material we ended out with a food analogy. _Credits to Thomas Eldblom and/or Jens Mikkelsen._

If Sitecore was a dish consisting of the following ingredients:

![SitecoreIngredients](/wp-content/uploads/2015/11/SitecoreIngredients.jpg)

Where the rice are items, the shrimps are templates, the salmon are renderings, and the broccoli configuration and so on.

Then you could prepare a nice paella for your customer

![Paella](/wp-content/uploads/2015/11/Paella.jpg)

But what if the customer turned out to be allergic to mussels? Then the whole dish would be ruined. Any attempt of throwing out the mussels would still leave traces of taste and deteriorate the whole dish significantly.

By following the principles of Modular Architecture you can instead end up with a nice, healthy and tasty set of sushi

![sushi](/wp-content/uploads/2015/11/sushi.jpg)

I had to update the training material once and stumbled upon this image that I just had to add to the analogy.

![buginsushi](/wp-content/uploads/2015/11/buginsushi.jpg)

If a bug is found in one of the Sitecore Sushi pieces, then you can simply throw out that single piece. It will not change or deteriorate your salmon maki rolls or any of the other sushi. You can also leave it on the plate for some time whilst you eat the rest.

Being able to discard features in this manner is extremely beneficial and I as an architect on large Sitecore solutions cannot live without this option now when I know how easy it is to obtain. You just have to follow these simple principles introduced in Modular Architecture.

When a new developer joins my team I can in full confidence instruct him, or her haven't happened yet, to work on a feature of the solution with no concerns. I know that the rest of the solution is safe and rookie code will not break it or bring down the overall quality. Everyone has to start somewhere and by following the principles of Modular Architecture it is even easier to get new developers familiar with Sitecore quickly.

To be honest then I myself can also have a good time writing quick and dirty code just to get the job done in a pragmatic way. It can be necessary sometimes especially when some print-designers have gone rogue and tried to draw up a website.

I can write this dubious code that does the job with full tranquility as long as it is well confined within an easily detachable module that is isolated and can be replaced without impacting the rest of the solution.

All code does not have to win beauty contests. As long as nothing else depends on it and it is not part of the foundation of the solution then quick and pragmatic code will do the job better in the long run.

> Simplicity is key, not complexity. Problems can be complicated, solutions cannot.

There are many more reasons for following modular architecture depending on your role and your view on the world. I have yet to meet someone working on a Sitecore project on either side of the table that does not benefit in some way from Modular Architecture.

> I am excited about Sitecore now does an effort in teaching these principles to the Sitecore community and I will encourage all Sitecore developers to show their support and learn these principles and techniques.

This will help us all and hopefully in the long run free us from spaghetti monster solutions that get thrown around various partners and degrades Sitecores reputation.

## How do I get started on Modular Architecture?

Start out with watching these 3 videos by Thomas Eldblom from Sitecore

1. Architecture Introduction: [Video](https://youtu.be/2CELqflPhm0)
1. Introduction to Modules: [Video](https://youtu.be/DgPrikqFe4s)
1. Introduction to Layers: [Video](https://youtu.be/XKLpTMuQT4Y)

Then when you are done with these get the Habitat code from gitHub and follow the [Sitecore Habitat installation instructions](https://github.com/Sitecore/Habitat/wiki/01-Getting-Started).

That's it, then you will be ready to explore the architectural principles and apply them to your solutions. Getting the hang of identifying modules and creating the right structure takes some experience though.

I would highly recommend you to read chapter 28 in [Agile Principles, Patterns, and Practices in C#](http://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258) (plus the rest of the book) and [Clean Code: A Handbook of Agile Software Craftsmanship](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) both by Robert C. Martin aka Uncle Bob.

Another important reference is his slides on [Advanced Principles of Component Design](https://www.google.dk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwjOptihnLnJAhXFjg8KHaQsAmIQFggkMAE&url=http%3A%2F%2Fbutunclebob.com%2Ffiles%2FSDWest2006%2FAdvancedPrinciplesOfComponentDesign.ppt&usg=AFQjCNG4WR9u1-PzVMbdHj8X2fiYvNfOoQ&sig2=_3KgzWnGpNUiJcA01hiFWA&bvm=bv.108194040,d.ZWU).

## What about the Habitat code base itself?

The Habitat solution itself and the technology stack it depends on is state of the art at the moment. The solution relies on all the cool new stuff here-among the new Task Runner in Visual Studio 2015. I simply love that Gulp and Grunt has become first class citizens in Visual Studio. It is about time that we get a de-facto standard for task runners in .NET. Unicorn is used for serialization and thumbs up for that as well, I love [Unicorn 3](https://github.com/kamsar/Unicorn), it is simply ingenious.

Code will decay over time and as I wrote Habitat is based on what is state of the art today, what will be state of the art tomorrow no one knows. The technology stack behind Sitecore is also rather regularly updated so I will highly encourage Sitecore to release a new Habitat for all new Sitecore versions. If the Habitat is to be more than just a firefly then it needs to be fully supported and updated by both Sitecore and the community also tomorrow and the day after. Otherwise the Habitat solution itself will decay and built up technical debt until it will be more of a burden than a helping hand. Habitat should always work as a clean slate demo / reference for that specific Sitecore version. The architectural principles are fixed though and does not change but the way they are brought into live will also change slightly with time and technology.

## Whats next?

My next two posts will be about the [layering concept in Modular Architecture](/layers-in-sitecore-modular-architecture/), the thoughts behind them and also how the layering concept can be extended in different fashions to support large-scale solutions with several subsites and brands.

> Are you interested in Modular Architecture training sessions?

Then you are always welcome to [contact me](mailto:alc@pentia.dk). Pentia offers specialized training courses for Sitecore developers on request.
