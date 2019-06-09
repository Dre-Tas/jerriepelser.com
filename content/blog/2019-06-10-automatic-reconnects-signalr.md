---
title: "Automatic reconnects with SignalR 3.0"
description:
  SignalR 3.0 adds the ability to automatically attempt to reconnect clients with the connection to the server is dropped
tags:
- signalr
- asp.net core
- javascript
---

In ASP.NET Core 3.0, Microsoft is updating the SignalR client by adding the ability to automatically reconnect when a connection is dropped. This blog post will show you how you can use this feature.

## Background

I am developing a Google Docs Add-On for [Cloudpress](https://www.usecloudpress.com/) that will allow users to publish content to their Content Management System from inside Google Docs. When a user publishes a document, the content is sent to Cloudpress where it will be processed by a background job (using [Hangfire](https://www.hangfire.io/)).

While the background job is being processed I want to give the user feedback on the progress, so I have implemented SignalR in the application for this purpose. I have implemented this using the same principles as I described previously in my [Communicate the status of a background job with SignalR](https://www.jerriepelser.com/blog/communicate-status-background-job-signalr/) blog post.

This is what it looks like in action:

<video autoplay="" loop="" muted="" style="width:100%" class="mb-3">
<source src="/images/blog/2019-06-10-automatic-reconnects-signalr/screencast.mp4" type="video/mp4">
</video>

This works great, but every so often I would run into an issue with the SignalR connection dropping. When this happens, the user experience is really bad, since the status of the job will not update, and the user does not know what is happening. You can manually reconnect, but you need to be careful about it and take things like exponential back-off into account.

I am aware that automatic reconnect is being added to SignalR in ASP.NET Core 3.0 and that the [PR for it was already merged](https://github.com/aspnet/AspNetCore/pull/8566) a few months ago. So, I decided rather than creating my own retry mechanism, I would try and use the 3.0 preview JavaScript client with my ASP.NET 2.2 application.

It turns out that for my limited use case, it works great. Let's see how we can configure it.

## Configuring automatic reconnects

Configuring automatic reconnects only requires a call to `withAutomaticReconnect` on the `HubConnectionBuilder`. Here is what my JavaScript code looks like for configuring my connection:

```js
connection = new signalR.HubConnectionBuilder()
    .withUrl("/publish-document-job-progress")
    .withAutomaticReconnect()
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

To try it out I ran the application and killed IIS after the SignalR connection was established. As you can see from the browser's console output, the connection is lost and SignalR tries to reconnect. Once I started IIS again, the connection was reestablished.

![SignalR tries to reconnect using exponential back-off](/images/blog/2019-06-10-automatic-reconnects-signalr/signalr-reconnect.png)

You will notice in the screenshot above, that SignalR tried to reconnect as soon as the connection was lost. It could not reconnect at that time, so it tried again 2 seconds later at which time IIS was back up and the connection was reestablished.

By default, SignalR will try and reconnect immediately, then 2 seconds later, then again after 10 seconds and then after 30 seconds. At that time, if the connection could still not be reestablished, it will stop trying to reconnect:

![SignalR disconnects permanently after 5 tries](/images/blog/2019-06-10-automatic-reconnects-signalr/signalr-permanent-disconnect.png)

You can configure the backoff period by passing an array of retry delays to the call to `withAutomaticReconnect()`. The default for this is `[0, 2000, 10000, 30000, null]`. The `null` value tells SignalR to stop trying. So, for example, if I wanted it to retry at 0, 1 second and 5 seconds, I can configure my `HubConnectionBuilder` as follows:

```js
connection = new signalR.HubConnectionBuilder()
    .withUrl("/publish-document-job-progress")
    .withAutomaticReconnect([0, 1000, 5000, null])
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

In my tests, it appears that it will stop retrying either when it runs out of retry delays in the array, or when it encounters a `null` value. So calling `withAutomaticReconnect([0, 1000, 5000, null])` or `withAutomaticReconnect([0, 1000, 5000,])` appears to have pretty much the same result.

Looking at the TypeScript source code for `HubConnectionBuilder`, it seems [you can also pass a custom retry policy](https://github.com/aspnet/AspNetCore/blob/4683b077da3ae8ddfbe238b4bc23e186147d8f99/src/SignalR/clients/ts/signalr/src/HubConnectionBuilder.ts#L169). The signature for that method is 

```ts
public withAutomaticReconnect(reconnectPolicy: IRetryPolicy): HubConnectionBuilder;
```

Where `IRetryPolicy` is defined as 

```ts
/** An abstraction that controls when the client attempts to reconnect and how many times it does so. */
export interface IRetryPolicy {
    /** Called after the transport loses the connection.
     *
     * @param {RetryContext} retryContext Details related to the retry event to help determine how long to wait for the next retry.
     *
     * @returns {number | null} The amount of time in milliseconds to wait before the next retry. `null` tells the client to stop retrying.
     */
    nextRetryDelayInMilliseconds(retryContext: RetryContext): number | null;
}

export interface RetryContext {
    /**
     * The number of consecutive failed tries so far.
     */
    readonly previousRetryCount: number;

    /**
     * The amount of time in milliseconds spent retrying so far.
     */
    readonly elapsedMilliseconds: number;

    /**
     * The error that forced the upcoming retry.
     */
    readonly retryReason: Error;
}
```

So, if you want to write retry logic with more complicated logic, you have that option at your disposal as well.