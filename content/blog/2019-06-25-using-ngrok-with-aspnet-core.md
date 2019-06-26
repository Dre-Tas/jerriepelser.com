---
title: "Access your local ASP.NET Core web application from the public web"
description:
  ngrok is tunnelling software that allows you to expose your local web server to the outside world. This is ideal for scenarios like testing webhooks.
tags:
- asp.net core
- webhooks
- ngrok
---

During the development of a web application, you may find yourself in a situation where you may need to access your local development web server from the public web. The most common scenario where this happens is when you need to test and debug webhooks since these need to be accessed from another system on the internet.

For situations like these, you can use [ngrok](https://ngrok.com/), which is tunnelling, reverse proxy software that establishes a tunnel from a public endpoint to your local web server. Let's look at how this works and how you can configure ngrok to work with an ASP.NET Core application.

## Installing ngrok

To install ngrok, head over to their [download page](https://ngrok.com/download) and download the correct version of ngrok for your operating system. Once you have downloaded the ZIP file, you can extract it to a directory on your computer. I took the extra step to add the directory where I extracted ngrok to my PATH so that I can run it from anywhere.

Before you can start using ngrok, you will need to connect your ngrok account. Log in to your ngrok account (or register for an account if you don't have one yet) and go the [Auth section of the dashboard](https://dashboard.ngrok.com/auth). Here you will see your tunnel authentication token listed:

![Find the tunnel authentication token in the dashboard](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/ngrok-auth-token.png)

Use that token to authenticate the ngrok CLI

```bash
ngrok authtoken 5cjX7h...
```

## Create the tunnel

Now that ngrok is installed and authenticated, you can create a tunnel. First, you need to start your ASP.NET Core application and take note of the port that it is using, specifically the HTTPS port, which is `5001` in the sample below.

![Run your ASP.NET Core application](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/run-application.png)

Start an HTTP tunnel by running the command `ngrok http` and passing the port, for example:

```bash
ngrok http 5001
```

![ngrok is running](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/ngrok-running.png)

In the status, ngrok will list the public URLs you can use to access your application. We're interested in the HTTPS URL, so in our case, this is `https://2dc5d7f4.ngrok.io`

## Bad gateway error

When I go to that URL, I am greeted with an error **502 Bad Gateway**.

![Bad gateway error when accessing the website](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/bad-gateway.png)

Searching the web, this seems to be [a common problem](https://stackoverflow.com/questions/40598428/ngrok-errors-502-bad-gateway) when using ngrok with ASP.NET Core. Thankfully, among the many suggestions listed in that StackOverflow issue, there is a correct solution.

Stop ngrok and run it again with the following parameters:

```bash
ngrok http https://localhost:5001 -host-header="localhost:5001"
```

ngrok will start again but take note that it gives you a different public URL each time you restart it:

![ngrok running with a different url](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/ngrok-running-take-2.png)

Head to this new URL, and this time you can see the application running correctly.

![Accessing the application via a public URL](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/accessing-website-via-public-url.png)

## Get a consistent URL

Having the public URL change every time you restart ngrok can be a bit of a nuisance. You can get custom, reserved subdomains by upgrading to their **Basic** plan. You can even use your own (white label) domain by upgrading to the **Pro** plan. This will save you the hassle of having the public URL change on you every time.

![ngrok paid plans](/images/blog/2019-06-25-using-ngrok-with-aspnet-core/ngrok-paid-plans.png)

## Conclusion

ngrok provides a convenient way of exposing your local development web server to other devices on the public internet. This can be useful in scenarios like testing and debugging webhooks.

BTW, **there is an alternative way to do this**, and that is with a Visual Studio extension called Conveyor. Be sure to check out my follow-up blog post describing [how to use Conveyor to access your IIS Express app over the internet](/blog/use-conveyor-access-iis-app-over-internet/).