---
layout: post
title: "AngularJS-ish filters in KnockoutJS"
date: 2013-09-22 15:00
comments: true
categories: 
published: false
---

As I learn more about [AnuglarJS](http://angularjs.org), I find [filters](http://docs.angularjs.org/guide/dev_guide.templates.filters) to be one of the most exciting features of the framework. The purpose of filters is to allow for applying common display logic to model (i.e. `$scope`) objects and expressions without having to include that logic in the model. For example, the AngularJS core framework comes with a handful of filters for doing things like formatting scalar values ([number](http://docs.angularjs.org/api/ng.filter:number), [currency](http://docs.angularjs.org/api/ng.filter:currency), [date](http://docs.angularjs.org/api/ng.filter:date)) and arrays ([filter](http://docs.angularjs.org/api/ng.filter:filter), [limit](http://docs.angularjs.org/api/ng.filter:limitTo), [order](http://docs.angularjs.org/api/ng.filter:orderBy)). And applying a filter to an expression in an Angular view is super easy: {% raw %}`{{ expression | filter }}`{% endraw %} and you're done.

In addition to the "write once, use everywhere" conveinence of a filter, filters are also chainable. This is particularly useful with Angular's array filters: {% raw %}`{{ customerList | filter:searchText | orderBy:'earnings' }}`{% endraw %} works just like you would expect it to (and of course it live-updates when you change the `searchText` property).

OK, so Angular is awesome. But what if I've already done a bunch of stuff using another library ([KnockoutJS](http://knockoutjs.com) in my case)? Can I do something like a filter in Knockout? **Hint: yes.**

Knockout has two ways that this could work: [extenders](http://knockoutjs.com/documentation/extenders.html) and [`fn` functions](http://knockoutjs.com/documentation/fn.html). Let's take a look at 

{% jsfiddle Mp5jw %}
