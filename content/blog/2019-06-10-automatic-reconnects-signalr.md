---
title: "Automatic reconnects with SignalR 3.0"
description:
  SignalR 3.0 adds the ability to automatically attempt to reconnect clients with the connection to the server is dropped
tags:
- signalr
- asp.net core
- javascript
---

In ASP.NET Core 3.0, Microsoft is updating the SignalR client and adding the ability to automatically reconnect when a connection is dropped. This blog post will show you how you can use this feature.

## Background

I am developing a Google Docs Add-On for [Cloudpress](https://www.usecloudpress.com/) that will allow users to publish content to their Content Management System from inside Google Docs. When a user publishes a document, the content is sent to Cloudpress where it will be processed by a background job (using [Hangfire](https://www.hangfire.io/)).

While the background job is being processed I want to give the user feedback on the progress, so I have implemented SignalR in the application for this purpose. I have implemented using the same principles as I described previously in my [Communicate the status of a background job with SignalR](https://www.jerriepelser.com/blog/communicate-status-background-job-signalr/) blog post.

This is what it looks like in action:

<video autoplay="" loop="" muted="">
<source src="/images/blog/2019-06-10-automatic-reconnects-signalr/screencast.mp4" type="video/mp4">
</video>