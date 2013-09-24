---
layout: post
title: "AngularJS-ish filters in KnockoutJS"
date: 2013-09-22 15:00
comments: true
categories: 
published: false
---

As I learn more about [AnuglarJS](http://angularjs.org), I find [filters](http://docs.angularjs.org/guide/dev_guide.templates.filters) to be one of the most exciting features of the framework. The purpose of filters is to allow for applying common display logic to model (i.e. `$scope`) objects and expressions without having to include that logic in the model. For example, the AngularJS core framework comes with a handful of filters for doing things like formatting scalar values ([number](http://docs.angularjs.org/api/ng.filter:number), [currency](http://docs.angularjs.org/api/ng.filter:currency), [date](http://docs.angularjs.org/api/ng.filter:date)) and arrays ([filter](http://docs.angularjs.org/api/ng.filter:filter), [limit](http://docs.angularjs.org/api/ng.filter:limitTo), [order](http://docs.angularjs.org/api/ng.filter:orderBy)). And applying a filter to an expression in an Angular view is super easy: {% raw %}`{{ expression | filter }}`{% endraw %} and you're done.

In addition to "write once, use everywhere" convenience, filters are also chainable. This is particularly useful with Angular's array filters: {% raw %}`{{ customerList | filter:searchText | orderBy:'earnings' }}`{% endraw %} works just like you would expect it to (and of course it live-updates when you change the `searchText` property).

OK, so Angular is awesome. But what if I've already done a bunch of stuff using another library ([KnockoutJS](http://knockoutjs.com) in my case)? Can I do something like a filter in Knockout? **Hint: yes.**

Knockout has two ways that this could work: [extenders](http://knockoutjs.com/documentation/extenders.html) and ["fn" functions](http://knockoutjs.com/documentation/fn.html).

### How `fn` works

In Knockout, you can add functinality to any objects that are `ko.subscribable` (or any of its derived types) by extending the `fn` object those types with functions. And because of inheritance, functions that you add to a type will be usable by objects of any of its derived types. We'll see why this is great for filters in a second.

{% img /images/posts/type-hierarchy.png %}

For instance, if I wanted all observable arrays to have a `pluck` function that plucked a property from each element and returned a new array of those properties, I could do something like this:

``` javascript
ko.observableArray.fn.pluck = function(propertyName){
	return ko.computed(function() {
		var propName = ko.unwrap(propertyName); //property might be an observable
		//"this" is the instance of the observable array
		return ko.utils.arrayMap(this(), function(item) {
			return ko.unwrap(item[propName]); //item[propName] might be an observable!
		});
	}, this);
}
```

**Why are we retruning a `ko.computed`?** Let's say you wanted to use `pluck` in a data binding in your view like this:

``` html
<ul class="nameList" data-bind="foreach: customerList.pluck('name')">
	<li data-bind="text: $data"></li>
</ul>
```

If `pluck` just returned a simple JavaScript array (or something that's not `ko.subscribable`), Knockout doesn't know that we depend on the array and the `name` property of each element and wouldn't update this binding when either of those things changes. This is shameless foreshadowing of what our filters will do.

### Filters with `fn`

{% jsfiddle Mp5jw %}

TODO: EXPLANATION OF EXAMPLE

### How extenders work

TODO

### Filters with extenders

{% jsfiddle ubpS7 %}

TODO: EXPLANATION OF EXAMPLE

### Pros and Cons

TODO

### Conclusion

TODO



