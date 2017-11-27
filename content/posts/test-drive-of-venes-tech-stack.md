---
title: "Test drive of VENES stack for searching open datasets -Part 1"
subtitle: "Hosted on a diverse cloud infrastructure "
date: 2017-11-15T00:58:09-05:00
categories: [Projects]
tags: ["vue", "elasticsearch","heroku","nodejs","aws","cloud","express"]
draft: false
---
### What is VENES?
**V**::Vuejs| 
**E**::Express| 
**N**::Nodejs|
**ES**::Elasticsearch
![stack](/img/venes.png)
### What we’re building
Given several open data sets or dataset apis, the idea is to extract the datasets, transform and load them into an [Elasticsearch](https://www.elastic.co/) cluster for fast searches via the Elasticsearch api. Users should be able to type in terms in a text box, google search style, and get instant relevant hits, then they should be able to narrow down their choices using some filtering mechanism. The ultimate vision is to have a more artifact focused search where users can search on a particular dataset, say recalls, and perform custom searches on it. With that said, the infrastructure choices needed to be carefully considered based on ease of use and cost.

### Why build it?
The open data search was built out of curiosity really. I noticed a lot of text heavy datasets were being exposed as APIs by different organizations and agencies but there wasn’t a way to search across all the different apis. There a few reasons why I wanted to build a full unified search application:
The real interest was when I started evaluating [Solr vs Elasticsearch](https://logz.io/blog/solr-vs-elasticsearch/) in 2016 and was really impressed by both tools, but what the elastic team has built I thought was more than a search engine, it was a powerful and flexible platform. I’m usually skeptical of any tool that tries to do and be everything under the sun but the different elastic tools that complement elasticsearch, do their job very well. I needed to get familiar with the [Elasticsearch DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html), or at least enough of it where I can build these aggregations/facets and filters using a modern web UI framework.
I have been a dotnet developer, (VB.NET), was the first programming language i built tools with, I was never a classic computer science student, although I retro actively sought the knowledge, dotnet had all a developer could ever want, besides the fact that it would only run naturally on Windows, and when i decided to switch to C# a couple years later, i was relieved to have left the verbose VB behind and was even more convinced and a happier dotbet developper when i got my hands on LINQ and the other powerful functional programming features that language like Java only had available within the last two years. Having said that, i knew that the middle tier of this particular app could not be in dotnet for a couple of reasons:
I wanted a dynamic/semi dynamic or lightweight language which means I had to learn something else and that was exciting to me.
The prospect of creating an all javascript css html app complete with full on two way binding,state management, command line scaffolding…yeah that was a challenge i was looking forward to.
To experiment in the cloud's free tiers, without switching to dotnet core (which a cloud platform such as [heroku](https://www.heroku.com/) currently doesn't support) would have been an unecessary impedement

### The Technology Stack & Architecture
 The idea was to decouple the web front end with the server at development time so that it’s possible to fiddle with different Elasticsearch clients to see which was a better fit for the server side API. The technology behind the api that would proxy the Elasticsearch interfaces is written in NodeJS. The choice was between NodeJS and Golang Follow along and i’ll tell you why later.
The infrastructure is composed of Heroku for hosting the web app, mlab for possible user management and metrics storage (this could be Postgres, have not decided) and AWS elasticsearch OR elasticcloud, both are eventually on AWS but one is fully managed by the elastic team and one is fully managed by AWS elasticsearch service.

![infrastructure](/img/infra.png)



### Decision on programming languages and technology stacks
#### Javascript & it’s framework nations
I think i am definitely an average javascript developer, i have no really developed full blown apps all in vanila javascript, i value my time and sanity but i know the gotchas and the deceptive nature of the language, also it's powerful features. I previously had some experience with AngularJS and was excited by it’s promise but the JavaScript world never stoped evolving, never stopped moving, I carefully followed the explosion of tools and framework battles, built some basic apps with at least 4 frameworks/libraries (Angular 2, Aurelia, VueJs, Knockout) and realized that my decision would be based on the framework that gets me the less frustrated, in comes VueJs.

#### Angular2
After checking out Angular 2, i had my mind made up because i thought that Angular 2 was an upgrade from AngularJS with the introduction of Typescript so i built an small service with Spring boot and Angular 2, i understand the goal but the framework was a bit too heavy and opinionated for a beginner like me.
#### Aurelia 
Aurelia with typescript was very similar to Angular but it opted for present and future ECMAscript syntax instead of inventing new ones. I thought it was great but again both Angular and Aurelia tried to force me to structure the  app as if i was writing a server side 3 tier app, something didn’t feel right, in came VueJS.
#### Vue
With VueJS at first i saw the .vue files generated by the vue-cli , all the properties, css , javascript and html in the same file, this made me cringe, i closed the project. I thought ok, why am i in such a hurry, take some time and revisit the frameworks. I started reading the wonderful documentation on the VueJS site, it all made sense, i never looked back, VueJS is all about keeping the cognitive mental model of an isolated component in «now memory». The VueJS components are self contained, they accept inputs and come alive. 
#### Middle Tier
For the middle tier, i chose Nodejs because well, Javascript…I wanted to write vanilla javascript and learn more about the NodeJs ecosystem. I really started with Golang but the Go elasticsearch client is not yet mature, although it’s highly performant and fantastic so far. In Node, the elasticsearch javascript client seems a natural fit given the Restful nature of the elasticsearch API. Yes i probably could have used .NET but i would not be able to use Heroku andeven though the elastic client for .NET is really robust, this is a subset of what it looks like when mapping fields to elastic search index: 

{{< highlight csharp "linenos=inline">}}
.Nested<Recall.Manufacturer>(rm => rm
                    .Name(mu => mu.Manufacturers)
                    .AutoMap()
                    .Properties(manu => manu
                        .String(id => id
                            .Name(n => n.CompanyID)
                            .NullValue("0")
                        )
                        .String(name => name
                            .Name(n => n.Name)
                            .Fields(f => f
                                .String(sf => sf
                                    .Name("raw")
                                    .NotAnalyzed()
                                 )
                            )
                        )
                   )
                )
                .Nested<Recall.Retailer>(rr => rr
                    .Name(ret => ret.Retailers)
                    .AutoMap()
                    .Properties(manu => manu
                        .String(id => id
                            .Name(n => n.CompanyID)
                            .NullValue("0")
                        )
                        .String(name => name
                            .Name(n => n.Name)
                            .Fields(f => f
                                .String(sf => sf
                                    .Name("raw")
                                    .NotAnalyzed()
                                 )
                            )
                        )
                   )
                )
                .Nested<Recall.ManufacturerCountry>(mc => mc

                    .Name(rmc => rmc.ManufacturerCountries)
                    .AutoMap()
                    .Properties(manu => manu
                        .String(id => id
                            .Name(n => n.Country)
                            .NullValue("none")
                        )
                        .String(name => name
                            .Name(n => n.Country)
                            .Fields(f => f
                                .String(sf => sf
                                    .Name("raw")
                                    .NotAnalyzed()
                                 )
                            )
                        )
                   )
                )
      
{{</ highlight >}}


C# code tragically trying to mimic a JSON document. I love C# and i did put it to good use as the major tool doing the ETL between the Source APIs & datasets to ElasticSearch. All explained in Part 2 of this series since it deserves it’s own post. Here is an example of an generic search query in JSON:
{{< highlight json "linenos=inline">}}
{
    "query": {
        "bool": {
            "must": [

                {
                    "match": {
                        "_all": "fire"
                    }
                }
            ],

            "filter": {
                "bool": {
                    "must": [{
                            "terms": {
                                "_type": ["neissreport"]
                            }

                        },

                        {
                            "range": {
                                "artifactDate": {
                                    "gte": "1970-09-20",
                                    "lte": "2009-09-26"
                                }
                            }
                        }
                    ]

                }
            }

        }

    },
    "sort": [
        { "_type": { "order": "desc" } },
        { "artifactDate": { "order": "desc" } }

    ],
    "aggregations": {
        "artifact_type": {
            "terms": {
                "field": "type.keyword"
            }
        },
        "artifact_source": {
            "terms": {
                "field": "artifactSource.keyword"
            }
        }
    }


}
{{</ highlight >}}
