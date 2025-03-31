---
author: alc
category:
  - sitecore
  - sitecore-7.1
  - speak
cover:
  alt: speaklist
  image: /wp-content/uploads/2014/01/speaklist.png
date: "2014-01-27T18:38:25+00:00"
guid: http://laubplusco.net/?p=658
title: Setting up a Sitecore SPEAK ListControl
url: /setting-sitecore-speak-listcontrol/

---
The ListControl in SPEAK is made as a generic control for showing lists. Data is fed into the list using a datasource component as shown in my previous post [here](/creating-simple-speak-json-datasource/ "Creating a simple SPEAK JSON datasource").

In this post I will show how to define the columns in a ListControl and setup the rendering on a SPEAK page. It is kind of similar to how it was done in the "old" version of SPEAK which has been used for ECM 2 (as one of the only concepts which has been kept in the new version).

## Defining the list

To define a list we start by creating a folder under PageSettings called Lists. In this folder we create a _**ListControl**_ item as shown below.

[![Messages List](/wp-content/uploads/2014/01/MessagesList.png)](/wp-content/uploads/2014/01/MessagesList.png)

On this item we can set some basic properties of the list we want to show.

Beneath the ListControl item we then create _**ColumnField**_ items for each column we want to show in the list.

[![Column field](/wp-content/uploads/2014/01/ColumnField.png)](/wp-content/uploads/2014/01/ColumnField.png)

On each column field you need to set the header text for the column and the name of the json property to show. This is actually just the field name if you use any of the out-of-the-box datasources which returns Sitecore items as json.

It is also possible to write another json property / field name for sorting the column and to provide a formatter for dates etc.

## Inserting the list on a page

Now open the presentation details for the page on which the list should be shown.

First add a DataSource rendering to the page if it is not there already and then add the ListControl rendering.

On the ListControl rendering set the datasource to the ListControl item which you created before and then set the Items parameter to bind with the collection on the datasource rendering.

[![List Control Rendering Parameters](/wp-content/uploads/2014/01/ListControlRenderingParameters1.png)](/wp-content/uploads/2014/01/ListControlRenderingParameters1.png) Notice that the datasource parameter is the datasource for the ListControl definition and not for the data shown in the list. The data shown in the list is set in the Items parameter, not the datasource. Obvious for Sitecore developers, but a bit confusing to write about.

And here is how the list looks which I took these screenshots from.

[![Finished List control](/wp-content/uploads/2014/01/ListControl.png)](/wp-content/uploads/2014/01/ListControl.png)

No coding needed but a nice looking list came out with sorting capabilities and what not.

## Retrospective

The ListControl in SPEAK is extremely easy to use with the built in datasources. You can have an advanced list up and running in less than five minutes if it is just showing data from items.

The ListControl in 7.1.130926 seems like an early version of what is to be in the future. It misses a lot of parameters for control how each column is rendered, for example width, line break behaviour etc.The formatter concept works fine for each column but could be easier if it was made as a dropdown of available formatters. The alignment dropdown is empty so not sure what it is meant to do.

I really hope that Sitecore will extend this control in the coming versions of SPEAK. Otherwise I will need to make my own version very soon, I need it for the Hackathon chat application asap.
