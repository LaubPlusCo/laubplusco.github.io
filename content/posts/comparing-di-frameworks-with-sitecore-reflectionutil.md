---
author: alc
category:
  - best-practices
  - sitecore
date: "2015-04-21T12:46:51+00:00"
guid: http://laubplusco.net/?p=1478
title: Comparing DI frameworks with Sitecore ReflectionUtil
url: /comparing-di-containers-and-sitecore-reflectionutil/

---
As a follow up on my last post about [IoC / DI containers and how to use the Sitecore API to perform inversion of control using ReflectionUtil](/simple-ioc-container-using-only-the-sitecore-api/ "Simple IoC container using only the Sitecore API"), I've made a quick speed comparison of some common DI frameworks, Sitecores ReflectionUtil and the new keyword as the best case for how quick it is to instantiate an object.

## The results

The table below shows how long time it took to resolve and instantiate 1000 instances of a type without using constructor or parameter injection and without any lifetime management.
Ninject27.360421582498Unity2.73812495641971ReflectionUtil0.19632199878594Simple Injector0.0757655566157367new Keyword0.0386374153259534
I ran the test 10 times and the results that I show here are the median, ranking from worst in top to best in bottom.

## The loser is..

Ninject with no comparison. Wow that was so slow that I almost couldn't believe it. Cool name though.

## And the winner is..

Clearly the new keyword as expected, with Simple Injector as a runner up.

## The morale

> Inversion of Control using Dependency Injection is cool and it has many usages.
>
> But only use it when you actually have a good reason to do so.
>
> Please think before introducing a dependency on a third party framework.
>
> _Is the framework really needed to solve any real world issues or are you just adding complexity for the sake of technology?_

If you need simple inversion of control for testing interfaces or allowing people to change your precompiled implementation etc. then you can use what is already present in the Sitecore API, good old ReflectionUtil. There is no need to introduce a new dependency on a third party framework. Life time management can be achieved in few lines of code as shown in my previous post.

Simple Injector really impressed me in regards to performance but I did not find it easy to use. I tried to pass in an object to the constructor which probably is a really simple thing to do when you first know how to do it, but the framework still lost a few points in my book due to the initial learning curve.

That said then Simple Injector is probably the IoC / DI framework that I will recommend when there is a good reason to use a DI framework at all.
