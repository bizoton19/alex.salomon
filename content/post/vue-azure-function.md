---
title: "Vue Azure Function"
date: 2018-07-22T16:56:44-04:00
categories: [Projects, Technology, Programming, Cloud]
tags: ["vue", "azure functions","javascript","async","lambda"]
draft: false
---

In this post, i will talk about how i built a pretty naive and primitive dashboard that displays the status of any URLs. The dashboard's interface is a `Vue.js` application with an `Azure function` backend. i will first give an overview of the app with it's architecture and the different technologies involved. I will focus more on the process and architecture rather than a step by step or code diving.

The prupose is to provide a simple but contextual level of monitoring to any website or sections of public websites or http APIs.
It uses http polling to ping the website and attempts to read the website or http service's content after getting a successfull http return code.
If an unsuccessful return code is read for one or more site, the application aggregates the number of failures and sends out an email alert.

![dashboard](/img/status-dashboard.png)

### Logical Architecture

The current architecure of the site status poller is depicted in the image below.
The app is hosted on [Netlify](https://www.netlify.com) which detects code changes on the master git branch and deploys it to...wherever. You don't have to use netlify of course, you can chose any hosting company. The web UI is a simple bootstrap/Vue app, created using the [Vue CLI 3.0](https://cli.vuejs.org/)  and Bootstrap 4.

![architecture](https://github.com/asalomon-cpsc/site-status-web/raw/master/azure_poller_architecture.png
"architecture")


### Infrastructure
The application and set of functions are built on [Microsoft Azure](https://docs.microsoft.com/en-us/azure/azure-functions/), a [serverless](https://azure.microsoft.com/overview/serverless-computing/) compute service that completely abstract the web server from the runtime and administration environment where developers can run code "as needed" or based on specific events.
The alerter module of the application is powered by [SendGrid](https://sendgrid.com/), an email delivery service that integrates well with azure functions, as a matter of fact, it has it's own function bindings in azure functions which allows for easy integration.

| Data Store    | Messaging     | Programming | Email Delivery Service |
| ------------- | ------------- |------------| -----------------------
| [Azure Storage Tables](https://azure.microsoft.com/en-us/services/storage/tables/)  | [Azure Queue Storage](https://azure.microsoft.com/en-us/services/storage/queues/)  | .NET/Javascript | [SendGrid](https://sendgrid.com/) |



### The Back end
I actually started with the back end first. I started with creating the azure functions. I had written a [polling program](https://github.com/bizoton19/site-status-notification-csharp) when i started familiarizing myself with `async await` in `.NET` where most of the code that i needed was already written but had to make some changes to it when porting to azure functions.

![architecture](https://github.com/asalomon-cpsc/azure-functions/raw/master/azure_functions_architecture.jpg "architecture")

As you can see in the folder structure below, the function names sort of matches the architecture layout image above. I realised that i had to decompose the modules in the original program into smaller functions in order to fully embrace the idea of lambda functions.

![azure function folders](/img/az-func-folders.png)

#### The Sequence Of Events

* The http poller function, *pollerTrigger* gets initiated by a pre defined poller trigger
                                          
* The poller function, [*statusPoller_http*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusPoller_http/run.csx) reads a list of urls by calling the [*statusUrlListReader*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusSiteStateReader/run.csx) function. The urls are stored in an azure storage table called **urlTable**.

* Then for each url retrieved from the **urlTable** , an http web request is triggered for the url/resource.

* The resultsets are aggregated and sent to two queues, **status-states-queue** & **status-history-queue**, for additional processing. All errors are sent to the *status-errors-queue*.

* Then the [*statusQueuePersister*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusQueuePersister/run.csx)  grabs the message in the **status-states-queue** to save it in the azure storage table named **statusesTable** . The [*statusHistoryQueuePersiter*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusHistoryQueuePersister/run.csx) also grabs the message from the **status-history-queue** to save them in the azure storage table named **statusHistoryTable**.

* If there are any errors, the email alerter function, [*emailAlerter_csharp*](https://github.com/asalomon-cpsc/azure-functions/blob/master/emailAlerter_csharp/run.csx), will retreive the errors from the **status-errors-queue** and send an email alert to pre configured receipients via the *sendGrid* service.

### Functions that support the web dashboard

* The [*urlPersister*](https://github.com/asalomon-cpsc/azure-functions/blob/master/urlPersister/run.csx) function receives a new URL from the dashboard and saves it to a queue, then the [*UrlQueuePersister*](https://github.com/asalomon-cpsc/azure-functions/blob/master/statusQueuePersister/run.csx) function grabs that URL to persist it in the **urlTable**

* The statusPoller_http is also triggered via the web dashboard in order to refresh the dashboard. The code snippet below shows the declaration of the azure function end points from the `vue.js` dashboard.

```javascript
import axios from "axios";
import moment from "moment";
import _ from "lodash";
import urlManager from "./UrlManager.vue"
import statusHistory from "./StatusHistory.vue"
export default {
  name: "poller",
  components: {
    axios,
    moment,
    urlManager,
    statusHistory
  },
  data: function() {
    return {
      response: {
        data: "",
        status: "",
        statusText: ""
      },
      fetching: "false",
      statuses: [],
      selectedStatus: {
        urlName: "",
        url: ""
      },
      statusListEndPoint:
        "https://functions-host.azurewebsites.net/api/statuses",
      statusRemoverEndPoint:
        "https://functions-host.azurewebsites.net/api/status",
      statusRefreshEndPoint: "https://functions-host.azurewebsites.net/api/poller"
    };
  },


```
code can be found [here](https://github.com/asalomon-cpsc/site-status-web/blob/master/src/components/Poller.vue)

### Complexity

The reason why the logical architecture seems complex is because thinking in terms of micro services, is a complex notion. The notion that failure in the operation pipeline is not an option, monolithic deployments and maintenance is no longer an option.
Now the reason why the architecture seems *overly* complex for an app as simple as a poller? Well the direction i **should** have taken was to use a service broker such as [azure service bus](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus) since we're already in the eco-system. The service bus would allow me to declare 

### The Pros
* Decoupled
* Fast
* Small Functions
* Cheap (besides the networking rate and the storage rate, the computing cost of azure functions is about .20 cents per 1000000 execution)

### The Cons

* Portability/Ownership
* Serverless eventually means relinquishing control over to the cloud vendor..Or you can be more strategic about your serverless architecture, smaller is more portable and less painful to rewrite!
* Rely on vendor for language feature and versions of language support

