---
title: "Determine available runtimes on Azure App Service"
description:
  Demonstrates how to determine the available .NET Core and Node runtimes on Azure App Service
tags:
- azure
- app service
- .net code
- nodejs
---

## Introduction

When deploying your .NET Core or Node application to Azure App Services, you may require a specific version of the .NET Core or Node runtimes being available. Here is how you can determine whether the required runtime version is available.

## Determine available runtimes

In the Azure Portal, navigate you your Azure App Service. Search for and select "Advanced Tools" in the navigation sidebar. then click on the "Go" link.

![Select Advanced Tools](/images/blog/2019-04-12-azure-app-service-available-runtimes/app-service-sidebar.png)

This will open the Kudu Service page. Click on the link labelled "Runtime versions".

![Open Kudu Services](/images/blog/2019-04-12-azure-app-service-available-runtimes/kudu-services.png)

This will open a JSON file listing all of the available runtimes. This file will look similar to the one below:

```json
{
    "nodejs": [
        {
            "version": "0.10.40",
            "npm": "1.4.28"
        },
        ...
        {
            "version": "8.5.0",
            "npm": "5.3.0"
        },
        {
            "version": "8.9.4",
            "npm": "5.6.0"
        }
    ],
    "dotnetcore32": {
        "shared": {
            "microsoft.netcore.app": [
                "1.0.12",
                ...
                "2.2.3"
            ],
            "microsoft.aspnetcore.app": [
                "2.1.7",
                ...
                "2.2.3"
            ],
            "microsoft.aspnetcore.all": [
                "2.1.7",
                "2.1.8",
                "2.1.9",
                "2.2.1",
                "2.2.2",
                "2.2.3"
            ]
        },
        "sdk": [
            "1.1.10",
            ...
            "2.2.105"
        ]
    },
    "dotnetcore64": {
        ...
    },
    "aspnetcoremodule": {
        "aspnetcoremodule": "12.2.18296.0",
        "aspnetcoremodulev2": "12.2.18296.0"
    },
    "system": {
        "os_name": "Windows Server 2016",
        "os_build_lab_ex": "14393.2791.amd64fre.rs1_release(bryant).181230-1202",
        "cores": 4
    }
}
```

## Setting Node runtime

If you want to specify a specific Node version for your website, you can specify it using the `WEBSITE_NODE_DEFAULT_VERSION` Application Setting. In the screenshot below, you can see that the Node version for this App Service has been configured to version `8.9.4`.

![Configure the Node version in the Application Settings](/images/blog/2019-04-12-azure-app-service-available-runtimes/app-service-app-settings-node-version.png)

This can be confirmed by opening the Azure Console and listing the Node version:

![Confirming the Node version in the Console](/images/blog/2019-04-12-azure-app-service-available-runtimes/app-service-console-node-version.png)
