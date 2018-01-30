---
title: "We Are the Watchers on the Vue"
date: 2017-12-10T22:14:49-05:00
categories: [Projects, Technology, Programming]
tags: ["vue", "watchers","javascript","async"]
draft: false
---

### What are watchers in Vue?
The Vue documentation describes watchers as a generic way to react to data changes through the watch option.

In other words when you want to perform an action based on a data change, using watchers are more appropriate. This would certainly be true per example if you are passing data from a parent to a child component where the data being passed is going to be part of a `submit()` operation that calls an api to do a `POST`.

### Under the hood
"When you have some data that needs to change based on some other data" usually means you need a watcher or in server side programming this can also be refered to as an observer, the only thing is that in vue, the watcher is a property called `watch`, it doesn't maintain a list of subject or property names to observe and react or notify another component when they change. Instead the watcher only watches a single property. The general rules are:

1. The watcher has to have the same name as the data property being watched.
2. the watcher has to perform an action such as calling a method that's part of the component or an external operation.

### Example
In this example, there is a watcher setup which essentially observes the `pageSelected` property of the component. The `pageselected` property gets updated by the event  on that search-pagination child component. When a new page number is a selected in the child component, the page number gets propagated to the search parent component then, the watcher detect the changes before calling the search function with the new page number.
```javascript
<template>

   
  <div id="search">
    
    <search-pagination 
     v-if="hasResults" 
    :totalPages="stats.totalPages"
    :currPage="pageselected"
     @pageSelected="pageselected=$event">
    </search-pagination>
    
   
  </div>
  
</template>
<script>
import searchPagination from "./SearchPagination.vue";

export default {
  name: "search",
  components: {
    searchPagination,
  },
data: function() {
    return {
     stats: {
        took: Number,
        total: Number,
        totalPages: Number,
        maxScore: Number,
        resultCount: Number
      },
      pageselected:1
      
    };
  },
watch: {
    
    pageselected: function(page){
      var that = this;
      console.log("watching page just turned to ", page)
      console.log("watching  ", this.searchinput)
      that.search(that.searchinput)
    }
  }
}
</script>
```
This component was simplified, the full code and the `searchPagination` code can be found [here](https://github.com/bizoton19/open-data-search-vuejs/blob/master/src/components/SearchPagination.vue)
