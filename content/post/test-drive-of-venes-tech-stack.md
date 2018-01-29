---
title: "Test drive of VENES stack for searching open datasets -Part 1"
subtitle: "Hosted on a diverse cloud infrastructure "
date: 2017-11-15T00:58:09-05:00
categories: [Projects, Technology, Programming]
tags: ["vue", "elasticsearch","heroku","nodejs","aws","cloud","express"]
draft: false
---
### What is VENES?
**V**  [Vuejs](https://vuejs.org/)| 
**E**  [Express](https://expressjs.com/)| 
**N**  [Nodejs](https://nodejs.org/en/)|
**ES** [Elasticsearch](https://www.elastic.co/products/elasticsearch)
![stack](/img/venes.png)
### What we’re building
Given several open data sets or dataset apis, the idea is to extract the datasets, transform and load them into an [Elasticsearch](https://www.elastic.co/) cluster for fast searches via the Elasticsearch api. Users should be able to type in terms in a text box, google search style, and get instant relevant hits, then they should be able to narrow down their choices using some filtering mechanism. The ultimate vision is to have a more artifact focused search where users can search on a particular dataset, say recalls, and perform more focused searches.

### Why build it?
Building it out of curiosity really. I noticed a lot of text heavy datasets were being exposed as APIs by different organizations and agencies but it would be benificial to have a search repository where the open datasets/apis are the initial lnding service. There a few reasons why I wanted to build a full unified search application:
I needed to get familiar with the [Elasticsearch DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html), or at least enough of it where I can build these aggregations/facets and filters using a modern web UI framework. The other reason is a learning experience which involves new web tools, different cloud environments for development/hosting and get some real work done with powerful search platform.


### The Technology Stack & Architecture
The idea was to decouple the web front end with the server at development time so that it’s possible to fiddle with different Elasticsearch clients to see which was a better fit for the server side API. The technology behind the api that would proxy the Elasticsearch interfaces is written in NodeJS. The choice was between NodeJS and Golang Follow along and i’ll tell you why later.
The infrastructure is composed of Heroku for hosting the web app, mlab for possible user management and metrics storage (this could be Postgres, have not decided) and AWS elasticsearch OR elasticcloud, both are eventually on AWS but one is fully managed by the elastic team and one is fully managed by AWS elasticsearch service.

![infrastructure](/img/infra.png)



### Decision on programming languages and technology stacks

#### The front end
  Following the explosion of javascript serverside tooling and frameworks, i built some basic apps with  3 frameworks/libraries (`Angular 2`, `Aurelia`, `VueJs`) and realized that my decision would be based on the framework that gets me the less frustrated (learning curve), because they all had the same goal, simplify javascript development and management while increasing productivity and performance. All three of these frameworks are fantastic, the support, enthusiast and community behing them are excellent, i'm not an expert in any of these frameworks but here are is the  main reason i chose vue.

#### Angular 
Even tough i believe [Angular 2](https://angular.io/) is a nice upgrade from AngularJS with the introduction of Typescript and i understand the goal, the framework demanded me to commit and conform too much to a particular style of building apps. If i were building this app within a large team, specially within an organisation, Angular would be my go to language for because the default design of the framework's api mirror the n-tier layered architecure and design patterns that developers have been using in `Java` and `C#` for a long time.
Writing Angular, the right way, the Angular way can be very elegant and easy to maintain in the long run. Now look at [how to get started with Angular: ](https://angular.io/guide/quickstart#devenv):

##### Step 1. Set up Your Machine (must have NPM installed)
```
npm install -g @angular/cli
```
##### Step 2. Create new Project 
```
ng new ang-app
```
---
#### Aurelia 
[Aurelia](http://aurelia.io/) with typescript was very similar to Angular but it opted for present and future ECMAscript syntax. I thought it was great, if Aurelia had the support of Angular, i would chose Aurelia over Angular for building large apps with large teams within an organisation. The support is fanstastic but depending on the organisation, they tend to go with the products that are backed by the biggest companies. [The getting started](http://aurelia.io/docs/tutorials/creating-a-todo-app):

##### Step 1. Set up Your Machine (must have NPM installed)
```
npm install -aurelia-cli -g
```
##### Step 2. Create new project
```
au new
```
---
#### Vue
 [Vue](https://vuejs.org/) is all about keeping the cognitive mental model of an isolated component in «now memory». The Vue components are self contained, they accept data/commands as inputs, animate and/or produce events and come alive. I hit the ground running pretty fast with Vue and never looked back. the documenation pages have live js fiddle code that you can get quick hands on practice with after reading the concepts. The getting started is as simple as that. Of course to build a full blown [single page app](https://blog.angular-university.io/why-a-single-page-application-what-are-the-benefits-what-is-a-spa/), your getting started will look more like the two other previously mentioned framework but the simple version focuses on just the view layer, no routing , no need to have npm or node installed as a pre-requisite. You can also slowly upgrade a legacy application, which are usually written in a composition of JSP/Javascript/Flash or ASP.net/Javascript/Flash etc.
 To get started:

##### Step 1. Drop this reference in a new or existig web project or in js fiddle

```
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

##### Step 2. Happy Coding!

Declare the main div tag where the vue instance will interact with the DOM (Document Object Model)
```html
<div id="app">
  {{ message }}
</div>
```
Declare a vue instance in your javascript file and reference the main div tag created earlier

```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
Serving your html file should now show you

```
Hello Vue!
```


Alternatively, use the `vue-cli` for users familiar with nodejs and it's build tools. This will allow you to build a full single page app just like Angular and Aurelia's cli tools.

##### Step 1. Set up Your Machine (must have NPM installed)
```
npm install --global vue-cli
```
##### Step 2. Create new project
```
vue init webpack vue-app
```

Here is a teaser of what the current Vue js based search page looks like. My elasticsearch instance is still local and the api proxy to elasticsearch is not yet stable:

![stack](/img/ui-teaser.gif)

#### The Elasticsearch proxy APIs
I wanted a dynamic/semi dynamic or lightweight language which means I had to learn something else.
I chose Nodejs because well, Javascript… I originally started with Golang but the offical Go elasticsearch client is not yet mature. There is another open source [Go client](http://olivere.gitub.io/elastic/) although it’s highly performant and is constantly improving, my instincts told me to go with [elaticsearch.js](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/index.html). It seems a natural fit given the Restful nature of the elasticsearch API. Here is an example of what an elasticsearch query looks like from the elasticsearch Restful API :

{{< highlight json "linenos=inline">}}

GET /*/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "_all": "poison"
          }
        }
      ],
          "filter": {
        "bool": {
          "must": [
            
            {
              "terms": {
                "type": [
                  "recall"
                ]
                
              }
            },
            {
              "terms": {
                "category.keyword": [
                  "Consumer Products","Foods, Medicines, Cosmetics"
                ]
                
              }
            },
            
            {
              "terms": {
                "artifactSource": [
                  "cpsc","fda"
                ]
                
              }
            },
            
             
            {
              "range": {
                "artifactDate": {
                  "gte": "1970-09-20",
                  "lte": "2019-09-26"
                }
              }
            }
          ]
        }
      }
    
    }
    
  
  },
  "size":"10",
  
  "aggregations": {
    "artifact_type": {
      "terms": {
        "field": "type.keyword"
        
      }
    },
    "artifact_category": {
      "terms": {
        "field": "category.keyword"
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

This elasticsearch query performs a search where the word *fire* is present, the `"match": {
                    "_all": "poison"
                }`
                  predicate tells elasticsearch to look for the word *poison* in every field in the document that has text data and then perform a filter on the result where the only data that gets return are of type **fdarecall** AND are between 1970 and 2009. The data is then sorted by type and artifact date. Then an aggregation object is also returned which retrieves the number of indexed documents that meet all the conditions of the aggregation
In `SQL` syntax the equivalent would roughly be :
```sql
SELECT [fields] FROM index_name 
WHERE fulltext_field LIKE '%poison%'
AND _type = 'fdarecall'
AND artifactDate BETWEEN '1970-09-20' AND '2009-09-26'
GROUP BY artifact_Source, artifact_type
ORDER BY _type, artifactDate DESC
```

The only difference is that, this query would **NOT** run in the SQL engine because you have to specify the fields that are going to be grouped and you can't really mix aggregation queries with resultset in one simple SQL command. You'de have to write a much more complicated and lenghty query.

We can even reduce the verbosity of the Json queries with the [bodybuilder](http://bodybuilder.js.org/) javascript package  by writing something like this (not tested yet, i want to make sure i can execute with the raw DSL first) :
```javascript
bodybuilder()
        .query('match', '_all', 'poison')
        .Filter('terms', '_type', 'fdarecall')
        .andFilter('range', 'artifactDate', [{ to: '2009-09-26' }, { from: '1970-09-20' }])
        .sort('_type','desc')
        .sort('artifactDate','desc')
        .aggregation('terms', 'type.keyword')
        .aggregation('terms','artifactSource.keyword')
        .build()
```

### Mapping the datasets for indexing
The mapping, transformation etc. of dataset fields to indexes is a stand alone ETL program, once that's finalized, i will also write a post about it. The basic structure of the initial search index will follow this data contract in order to be included in the home page's search results:
```csharp
 public interface IArtifact:IDataCategoryType
    {
        String UUID { get;  }
        String Title { get; set; }
        DateTime ArtifactDate { get;  }
        string ArtifactSource { get; }
        string Description { get;  }
        string Category { get; }
        string FullTextSearch { get;set;}
    }
```


