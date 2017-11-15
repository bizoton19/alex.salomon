---
title: "Test drive of “VENES” tech stack for searching open datasets powered by a diverse cloud infrastructure -Part 1"
date: 2017-11-15T00:58:09-05:00
draft: true
---
## Test drive of “VENES” stack for open data searches powered by a diverse cloud infrastructure -Part 1

### What we’re building

Given several open data sets or dataset apis, the idea is to extract the datasets, transform and load them into an Elasticsearch cluster for fast searches via the Elasticsearch api. Users should be able to type in terms in a text box, google search style, and get instant relevant hits, then they should be able to narrow down their choices using some filtering mechanism. The ultimate vision is to have a more artifact focused search where users can search on a particular dataset, say recalls, and perform custom searches on it, possibly save their searches…Well not going to get ahead of ourselves here as there is much to learn and lots of work to do. With that said, the infrastructure choices needed to be carefully considered based on ease of use and cost.

### Why we’re building it.
The open data search was built out of curiosity really. I noticed a lot of text heavy datasets were being exposed as APIs by different organizations and agencies but there wasn’t a way to search across all the different APIs. There a few reasons why I wanted to build a full unified search application:
The real interest was when I started evaluating Solr vs Elasticsearch in 2016 and was really impressed by both tools, but what the elastic team has built I thought was more than a tool, it was a powerful and flexible platform. I’m usually skeptical of any tool that tries to do and be everything under the sun (TFS, Sharepoint etc) but the scope of the different elastic tools that complement elasticsearch, did their job very well. I wanted to learn the Elasticsearch DSL, or at least enough of it where I can build these aggregations/facets and filters using a modern web UI framework.
I have been a dotnet developer (VB.NET) was my first programming language, I was never a classic computer science student, although I retro actively sought the knowledge, .NET had all a developer could ever want, and when i decided to switch to C# a couple years later, i was relieved to have left the verbose VB behind and was  even more convinced and a happier .NET developper when i got my hands on LINQ. Having said that, i knew that the middle tier of this particular app could not be in .NET, I wanted a dynamic, semi dynamic or light weight language which means I had to learn something else and that was exciting to me.
The prospect of creating an all javascript css html app complete with full on two way binding,state management, command line scaffolding…yeah that was a challenge i was looking forward to.

### The Technology Stack & Architecture
VENES(Vuejs, Express, Node, ElasticSearch), was the choice at the end, altough the idea was to decouple the web front end with the server at development time so that it’s possible to fiddle with different elastic search clients to see which was a better fit for the server side API. The candidate programming languages for this task wasn’t a large group, the choice was strictly between NodeJs and Go. Follow along and i’ll tell you why later.
The infrastructure is composed of Heroku for hosting the web app, mlab for possible user management and metrics storage (this could be Postgres, have not decided) and AWS elasticsearch OR elasticcloud, both are eventually on AWS but one is fully managed by the elastic team and one is fully managed by AWS elasticsearch service.


### Decision on programming languages and technology stacks
#### Javascript & it’s framework nations
All i know about JavaScript, is that it’s single threaded, functions are first class citizens, and it’s super dynamic nature can get you in trouble if not careful. After that, it all seems pretty familiar to me. I previously had some experience with AngularJS and was excited by it’s promise but the JavaScript world never stoped evolving, never stopped moving, I carefully followed the explosion of tools and framework battles, built some basic apps with at least 4 frameworks/libraries (Angular 2, Aurelia, VueJs, Knockout) and realized that my decision would be based on the framework that gets me the less frustrated, in comes VueJs.

#### Angular2
After checking out Angular 2, i had my mind made up because i thought that Angular 2 was an upgrade from AngularJS with the introduction of Typescript so i built an small service with Spring boot and Angular 2, i understand the goal but the framework was a bit too heavy and opinionated for a beginner like me.
#### Aurelia 
Aurelia with typescript was very similar to Angular but it opted for present and future ECMAscript syntax instead of inventing new ones. I thought it was great but again both Angular and Aurelia tried to force me to structure the  app as if i was writing a server side 3 tier app, something didn’t feel right, in came VueJS.
#### Vue
With VueJS at first i saw the .vue files generated by the vue-cli , all the properties, css , javascript and html in the same file, this made me cringe, i closed the project. I thought ok, why am i in such a hurry, take some time and revisit the frameworks. I started reading the wonderful documentation on the VueJS site, it all made sense, i never looked back, VueJS is all about keeping the cognitive mental model of an isolated component in «now memory». The VueJS components are self contained, they accept inputs and come alive. 
#### Middle Tier
For the middle tier, i chose Nodejs because well, Javascript…I wanted to write vanilla javascript and learn more about the NodeJs ecosystem. I really started with Golang but the Go elasticsearch client is not yet mature, although it’s highly performant and fantastic so far. In Node, the elasticsearch javascript client seems a natural fit given the Restful nature of the elasticsearch API. Yes i probably could have used .NET but i would not be able to use Heroku and my elastic search queries would look like this: {image of c# code} 



C# code tragically trying to mimic a JSON document. I love C# and i did put it to good use as the major tool doing the ETL between the Source APIs & datasets to ElasticSearch. All explained in Part 2 of this series since it deserves it’s own post.
