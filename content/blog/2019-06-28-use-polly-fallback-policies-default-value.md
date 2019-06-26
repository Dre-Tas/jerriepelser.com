---
title: "Use Polly fallback policies for default return values"
description:
  Fallback policies in Polly allow you to provide a substitute value in the event of a failure. This is useful in situations you don't want failure to throw an exception, but instead return a default value.
tags:
- dotnet core
- polly
- httpclient
---

## Introduction

There are certain situations where you are accessing an external service where an error result is a legitimate response. In situations like this, you do not want your application to throw an exception, but rather return a "default" value instead. In this blog post, I will demonstrate how you can use [Polly fallback policies](https://github.com/App-vNext/Polly/wiki/Fallback) for these scenarios.

## Background

I have blogged about [Polly](https://github.com/App-vNext/Polly) on a couple of previous occasions. The first time was about using retry policies to [automatically retry failed requests](https://www.jerriepelser.com/blog/retry-network-requests-with-polly). The second time was also about retry policies, but this time, I used it to [refresh a Google access token](https://www.jerriepelser.com/blog/refresh-google-access-token-with-polly/).

Well, I ran into another scenario where Polly bailed me out with an elegant solution.

In this scenario, I am interfacing with the [Contentful Content Management API](https://www.contentful.com/developers/docs/references/content-management-api/) via their [.NET SDK](https://github.com/contentful/contentful.net). At one point in my application, I attempt to retrieve a previously created entry in Contentful, based on a reference to the entry which I have stored.

Since the management of the entry on Contentful is outside of the control of my application, it may happen that the user has since deleted the entry. So, when I try and retrieve the entry, the Contentful SDK will throw an exception indicating that an underlying error occurred with HTTP Status 404.

Instead of throwing an error in this case, the logic of my application allows me to carry on gracefull by returning a `null` value, rather than throwing an error.

Of course, you can handle this in a try-catch block, but I saw another opportunity to make use of Polly. 

Enter the **Fallback Policy**.

## Configuring and using a fallback policy

According to the [Polly documentation](https://github.com/contentful/contentful.net), the purpose of the fallback policy is _To provide a substitute value (or substitute action to be actioned) in the event of failure._ This is exactly what I want to do in my scenario described above.

To retrieve an entry with the `ContentfulManagementClient`, you can call the `GetEntry()` method. This will return an instance of `Entry<dynamic>`. So, not considering any exceptions, the code in my application would look as follows:

```cs
var entry = await client.GetEntry("document-key")
```

Changing this to use Polly, I first create a policy. The policy handles any exception of type `ContentfulException`, where the `StatusCode` is `404`. I specify the fallback value to return a null value of type `Entry<dynamic>`:

```cs
var policy = Policy<Entry<dynamic>>
    .Handle<ContentfulException>(exception => exception.StatusCode == 404)
    .FallbackAsync((Entry<dynamic>)null);
```

Then, I can wrap the previous inside this policy as follows:

```cs
var  entry = await policy.ExecuteAsync(() => client.GetEntry("document-key"));
```

This allows my application to perform a check to see whether `entry` is null and if so, handle the situation more gracefully. This is what the complete code looks like:

```cs
var policy = Policy<Entry<dynamic>>
    .Handle<ContentfulException>(exception => exception.StatusCode == 404)
    .FallbackAsync((Entry<dynamic>)null);

var  entry = await policy.ExecuteAsync(() => client.GetEntry("document-key"));

if (entry != null) 
{
    // Handle the normal situation
}
else
{
    // Handle the situation where the entry does not exist anymore more gracefully
}
```

## Conclusion

Polly fallback policies allow you to handle failures gracefully. It is entirely possible to do this with a regular try-catch block, but I think the use of the Polly fallback policy makes the code flow a bit more logically.