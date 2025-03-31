---
author: alc
category:
  - sitecore
date: "2014-08-21T20:45:50+00:00"
draft: "true"
guid: http://laubplusco.net/?p=973
title: How to use PowerShell to automate your Sitecore development environment
url: /

---
During my years as a Sitecore developer I have seen way more than a dozen different ways of setting up a development environment. Each agency have their own approach and most agencies even use different approaches on different projects.

The best development environment I've ever encountered is the one at Pentia which cover almost all steps in any Sitecore development process. The setup is pragmatic, extremely easy to use whilst still being easy to understand.  Even though it is years old and uses somewhat old technology it is really superior to any of the new modern setups I've seen and probably will be for years to come. The reason for this is because it puts process over technology. It is the well-defined steps in the development process that decides how things are done and not the off-the-shelf tool that is used.

Some times our customers require their own development environment and we helped quite a few tailoring their own either on premises or in the cloud where we implement the steps they need in their process inspired by our own approach and tools.

I've used a wide selection of technologies  for automating various tasks such as creating deploy packages, merging configuration files, including modules etc. The steps needed in the development process varies from customer to customer and differences in their server setup also sets forth some limitations.

Some years ago I would advice to use batch files and build events to keep the setup simple whereas now the "hip" technologies count among other Powershell, NuGet, Octopus Deploy, SlowCheetah, TDS and so on which are all really great and makes our live as developers a joy.

There is a problem though. When using a wide set of different technologies it requires developers who work on the solution to know these to some extend or at least be aware of how they work. This creates an overhead and the knowledge tends to heap up at one or two members on the team. Explaining the development environment becomes too complicated, the work gets tedious and boring and time is wasted adjusting to the tools instead of coding and having fun.

At some point such development environments traded simplicity in for having the new cool technologies.

## I like simple things

At Pentia we can use our build tool to run a full build which copies over Sitecore, Sitecore modules, third party libraries and everything needed to get the solution up and running. It is possible to setup a new working Sitecore installation with a range of modules installed within less than 10 minutes. Checking out an existing solution for the first time on a developer machine takes 5 minutes or less.

All of this is done by clicking a button.

There is a lot of different steps executed to achieve this. All of the steps are really simple by themselves but together they solve a complex problem and form an extremely well-functioning development environment.

One of the key ingredients is a shared build library which contains all Sitecore versions from the initial 5.3 and upwards to the latest one publicly available. The library also contains all Sitecore modules you can think of and a ton of different third party libraries. Pentia also have a range of our own modules which can be installed from the library by simply referencing them in the solution build file.

I really miss the build library when I am out setting up a custom environment for a customer. We've tried replacing the functionality with NuGet but it has turned out that this adds a layer of complexity to the setup and suddenly it is the tool which decides the process and not the process itself.

## PowerShell to the rescue

Imagine that by simply pressing build in Visual Studio you would automatically get all the required external dependencies to make your code run.
