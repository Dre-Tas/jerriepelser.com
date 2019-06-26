---
title: "Use Conveyor to access your IIS Express app over the internet"
description:
  Conveyor is an extension for Visual Studio that allows you to access your local IIS Express app from the public web. This can be useful in situations where you want to test webhooks, for example.
tags:
- asp.net core
- webhooks
---

## Introduction

In [yesterday's blog post](/blog/using-ngrok-with-aspnet-core/), I looked at how you can use [ngrok](https://ngrok.com/) to access an ASP.NET Core web application running on your computer from the public internet. This is very useful in situations where you want to test and debug webhooks, for example.

One of the issues I mentioned on that blog post was that ngrok changes the public URL for your application every time you restart it. This can be a nuisance, and though you can prevent that by buying a paid subscription, there is another, free option available to you as well. 

It is a Visual Studio extension called [Conveyor](https://marketplace.visualstudio.com/items?itemName=vs-publisher-1448185.ConveyorbyKeyoti). In this blog post, I'll show you how to install and use Conveyor.

## Installing Conveyor

You can install Conveyor from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vs-publisher-1448185.ConveyorbyKeyoti). In Visual Studio (_I am using VS 2019_), go to _Extensions > Manage Extensions_.

![Open the Manage Extensions menu](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/manage-extensions-menu.png)

Ensure you have selected the _Online_ section, then search for _conveyor_. You will see it listed as _Conveyor by Keyoti_. Click on the **Download** button to install the extension.

![Search for Conveyor in the extensions marketplace](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/manage-extensions.png)

You will need to close and restart Visual Studio as part of the installation process.

## Using Conveyor

After the installation is complete, open and run the application that you want to expose on the internet. Be sure to run your application with **IIS Express**.

![Run the application with IIS Express](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/run-with-iis-express.png)

Once the application has started up, you can access Conveyor by going to _Tools > Conveyor_.

![Access Conveyor from the Tools menu](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/conveyor-menu-item.png)

A Visual Studio tool window will open and you will see your application listed, alongside a _Remote URL_. You can access the application using that remote URL. 

![Conveyor running with the Remote URL](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/conveyor-running-with-remote-url.png)


As you can see, that URL is just an IP address with a port, but you may need to access your application using a proper domain name without a port. To do that, click on the **Access over Internet** button.

![Click the access over internet button](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/access-conveyor-over-internet.png)

At this point, you will be asked to log in to the Conveyor website or sign up for a new account.

![Sign up or log in to Conveyor ](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/log-in-to-conveyor.png)

After you have logged in, you will see an _Internet URL_ in the Conveyor tool window.

![Conveyor running with the internet URL](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/conveyor-running-with-internet-url.png)

You can now access your application from any internet device with this URL:

![Access your application over the internet](/images/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/access-application-with-internet-url.png)

## Conclusion

In this blog post, I demonstrated how you can use the Conveyor extension for Visual Studio to make your application running on IIS Express available over the internet. This is useful for scenarios like testing and debugging webhooks, or anywhere else an external system needs to access your application.