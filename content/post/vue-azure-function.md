---
title: "Vue Azure Function"
date: 2018-07-22T16:56:44-04:00
categories: [Projects, Technology, Programming, Cloud]
tags: ["vue", "azure functions","javascript","async","lambda"]
draft: true
---

In this post, i would like to demonstrate how i built a pretty naive and primitive dashboard that displays the status of any URLs. The dashboard's interface is a `Vue.js` application with an `Azure function` backend. i will first give an overview of the app with it's architecture and the different technologies involved. Then i will  do a part 2 with a step by step detailed tutorial with an explanation of why there is a more efficient approach when using decoupled azure functions instead of explicitly using several queues as i did here.

The prupose is to provide a simple but contextual level of monitoring to any website or sections of public websites or http APIs.
It uses http polling to ping the website and attempts to read the website or http service's content after getting a successfull http return code.
If an unsuccessful return code is read for 1 or more site, the application aggregates the number of failures and sends out an email alert.

![dashboard](/img/status-dashboard.png)

## Logical Architecture

The current architecure of the site status poller is depicted in the image below.
The app is hosted on [Netlify](https://www.netlify.com) which detects code changes on the master git branch and deploys it to...wherever. You don't have to use netlify of course, you can chose any hosting company but i'll stil show how to do this with netlify. The web UI is a simple bootstrap/Vue app, created using the [Vue CLI](https://cli.vuejs.org/)


![architecture](https://github.com/asalomon-cpsc/site-status-web/raw/master/azure_poller_architecture.png
"architecture")


## Infrastructure
The application and set of functions are built on [Microsoft Azure](https://docs.microsoft.com/en-us/azure/azure-functions/), a [serverless](https://azure.microsoft.com/overview/serverless-computing/) compute service that completely abstract the web server from the runtime and administration environment where developers can run code "as needed" or based on specific events.
The alerter module of the application is powered by [SendGrid](https://sendgrid.com/), an email delivery service that integrates well with azure functions, as a matter of fact, it has it's own function bindings in azure functions which allows for easy integration.

| Data Store    | Messaging     | Programming | Email Delivery Service |
| ------------- | ------------- |------------| -----------------------
| [Azure Storage Tables](https://azure.microsoft.com/en-us/services/storage/tables/)  | [Azure Queue Storage](https://azure.microsoft.com/en-us/services/storage/queues/)  | .NET/Javascript | [SendGrid](https://sendgrid.com/) |

![architecture](https://github.com/asalomon-cpsc/azure-functions/raw/master/azure_functions_architecture.jpg "architecture")

## The Back end
I actually started with the back end first. I started with creating the azure functions. I had written a [polling program](https://github.com/bizoton19/site-status-notification-csharp) when i started familiarizing myself with `async await` in `.NET` where most of the code that i needed was already written but had to make some changes to it when porting to azure functions.

As you can see in the folder structure below, the function names sort of matches the architecture layout image above. I realised that i had to decompose the modules in the original program into smaller functions in order to fully embrace the idea of lambda functions.

![azure function folders](/img/az-func-folders.png)

## The Sequence Of Events

1. The http poller function, *pollerTrigger* gets initiated by a pre defined poller trigger
2. The poller function, *statusPoller_http* reads a list of urls by calling the [*statusUrlListReader*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusSiteStateReader/run.csx) function. The urls are stored in an azure storage table called ***urlTable***.
3. Then for each url retrieved from the ***urlTable*** , an http web request is triggered for the url/resource.
4. The resultsets are aggregated and sent to two queues, *status-states-queue* & *status-history-queue*, for additional processing. All errors are sent to the *status-errors-queue*.
5. Then the *statusQueuePersister* grabs the message in the ***status-states-queue*** to save it in the azure storage table named ***statusesTable*** . The *statusHistoryQueuePersiter* also grabs the message from the *status-history-queue* to save them in the azure storage table named ***statusHistoryTable*** .

## Functions that support the web dashboard

* *UrlPersister* function receives a new URL from the dashboard and saves it to a queue, then the *UrlQueuePersister* grabs that URL to persist it in the **urlTable**

* The statusPoller_http is also triggered via the web dashboard in order to refresh the dashboard.

