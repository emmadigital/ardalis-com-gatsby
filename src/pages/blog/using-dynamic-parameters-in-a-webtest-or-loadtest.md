---
templateKey: blog-post
title: Using Dynamic Parameters in a WebTest or LoadTest
path: blog-post
date: 2010-09-17T12:30:00.000Z
description: >
  Using Visual Studio 2010 (and some earlier versions), it’s very easy to create
  a WebTest by recording one or more web requets to the system under test
  (SUT).  To get started, open Visual Studio 2010 and create a New
  Project.  Then select Test Project like so:
featuredpost: false
featuredimage: /img/snaghtml3e5239_1.png
tags:
  - load testing
  - performance
  - Scalability
  - testing
  - visual studio
category:
  - Software Development
comments: true
share: true
---
Using Visual Studio 2010 (and some earlier versions), it’s very easy to create a WebTest by recording one or more web requets to the system under test (SUT). To get started, open Visual Studio 2010 and create a New Project. Then select Test Project like so:

![](/img/snaghtml356cf0_1.png)

Next, add a Web Test to the project, and hit the URL you want to test (the one that accepts multiple different parameters that you want to make dynamic). Run the test and verify it passes with a single set of hardcoded parameters. You should see something like this:

![](/img/snaghtml393284_1.png)

Now it’s time to change the hardcoded parameters you’re using into dynamic parameters. The first thing you need to do is generate the code for your existing WebTest. To do this, click the Generate Code icon, shown here:

![](/img/web-test-1.png)

This will generate a new file called WebTest1Coded.cs (or whatever you named your test class plus “Coded”). Looking at that file, you can see that it was generated by a tool and if you want to keep your changes intact I recommend you rename the file so that any future clicks of Generate Code do not overwrite your modified file.

![](/img/snaghtml3e5239_1.png)

At this point, making the parameters dymamic is pretty straightforward. You can run your coded web test from Test Manager just like any other Web Test. You can easily add, modify, etc any of the requests using standard .NET code. The requests look something like this:

![](/img/web-request-1.png)

If you just need one or two parameters, then of course you can copy and paste (and rename each instance) these requests. In my case, though, I had quite a few different parameters, and each one had a fairly large number of possible values that I wanted to emulate, since these entries would largely correspond to separate ASP.NET cache entries, and I wanted to be able to [analyze my ASP.NET cache utilization](https://ardalis.com/real-world-monitoring-and-tuning-asp-net-caching).

To substitute a random, but valid, value for each querystring entry, I created a new class called RequestParameters. In its static constructor, I created an array for each querystring value and populated it with all of the possible options I wanted it to use. In many systems, there might be lookup tables that contain the possible values, and it would be simple to call out to a database to populate these arrays with real data (it would only be called once, not on every request in the load test, if done in the static constructor).

Finally, I created a simple method to return a random element of an array and called this from a method that I used in place of the value for the appropriate querystring value. Here’s the code for the random selection of the element and the code to return back a value for a querystring item:

```
public static string GetSku()
{
  return GetRandomElement(_validSkus);
}
private static T GetRandomElement<T>(T[] array)
{
  int index = _random.Next(array.Length);
  return array[index];
}
```

Once you have a GetWhatever() method, you can simply call it like this from the coded web test:

```
request1.QueryStringParameters.Add("sku", RequestParameters.GetSku(), false, false);
```

**Summary**

Using the testing features in Visual Studio, you can verify that web pages in your application load correctly, as well as simulate load tests that will let you determine the scalability of your application. In order to simulate real-world usage, though, it’s often necessary to simulate a wide variety of input parameters. Using coded tests, it’s quite straightforward to simulate real-world variation in the incoming web requests, allowing your load tests to simulate your production environment.