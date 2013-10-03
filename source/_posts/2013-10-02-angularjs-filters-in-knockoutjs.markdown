---
layout: post
title: "AngularJS-ish filters in KnockoutJS"
date: 2013-10-02 22:00
comments: true
categories: 
published: false
---

As I learn more about [AnuglarJS](http://angularjs.org), I find [filters](http://docs.angularjs.org/guide/dev_guide.templates.filters) to be one of the most exciting features of the framework. Filters allow common, mostly-simple display-level modifications to expressions in a view without having to include that logic in the model (i.e. `$scope`).

Applying a filter to an expression in an Angular view is super easy: {% raw %}`{{ expression | filter }}`{% endraw %} and you're done. Angular comes with a handful of filters for scalar values ([number](http://docs.angularjs.org/api/ng.filter:number), [currency](http://docs.angularjs.org/api/ng.filter:currency), [date](http://docs.angularjs.org/api/ng.filter:date)) and arrays ([filter](http://docs.angularjs.org/api/ng.filter:filter), [limit](http://docs.angularjs.org/api/ng.filter:limitTo), [order](http://docs.angularjs.org/api/ng.filter:orderBy)) but it's also pretty easy to write your own.

In addition to "write once, use everywhere" convenience, filters are also chainable. This is particularly useful with Angular's array filters: {% raw %}`{{ customerList | filter:searchText | orderBy:'earnings' }}`{% endraw %} works just like it reads (only show customers with a property value containing the value of the model property `searchText` and then order those customers by the `earnings` property). And of course it live-updates when you change `searchText`.

OK, so Angular is awesome. But what if I've already done a bunch of stuff using another library ([KnockoutJS](http://knockoutjs.com) in my case)? Can I do something like a filter in Knockout? **Hint: yes.**

Knockout has two ways that this could work: ["fn" functions](http://knockoutjs.com/documentation/fn.html) and [extenders](http://knockoutjs.com/documentation/extenders.html).

## How `fn` works

Each of Knockout's subscribable types exposes a public `fn` object to which you can add new functions to extend that type with cool new stuff. From then on, that stuff will be available on any new instance of that type. You can add new functions to any of the following:

* `ko.subscribable.fn`
* `ko.observable.fn`
* `ko.observableArray.fn`
* `ko.computed.fn`

And because of inheritance, functions that you add to a type will be usable by objects of any of its derived types. We'll see why this is great for filters in a second.

{% img /images/posts/type-hierarchy.png %}

## Filters with `fn`

So how do we turn this into something like filters? Take a look at this fiddle.

{% jsfiddle Mp5jw js,html,resources,result %}

You can see that I've added 4 functions to `ko.subscribable.fn`:

* `filter` to filter an array by a string value
* `orderBy` to order an array by an element property value
* `currency` to format a number as currency
* `date` to format a date

### Why `ko.subscribable.fn`?

Why not put `filter` and `orderBy` on `ko.observableArray.fn` and put `currency` and `date` on `ko.observable.fn`? Simple: adding these functions to `ko.subscribable` maximizes chainability.

Let me explain. First, you want to return a `ko.computed` from filter functions. This ensures automatic UI updates because Knockout tracks subscribables accessed in the function body of a `ko.computed` and automatically re-evaluates the function when any of those values changes. But a `ko.computed` is not an observable or an observableArray. It is a `ko.subscribable`, though. So adding these functions to `ko.subscribable.fn` allows the inclusion of clear, chainable filters in data bindings like this one: `data-bind="foreach: dudes.filter(search, 'name').orderBy(orderBy)"`.

It even makes sense to put the scalar (`currency` and `date`) filters on `ko.subscribable.fn` in case you ever add a filter (or filters) that whittles an array to a single value (i.e. `first`, `last`, `findOne`, `imFeelingLucky`, etc.).

By putting filter functions on `ko.subscribable.fn`, they are available to **all** subscribable types, giving you lots and lots of flexibility.

## How extenders work

Extenders and `fn` are really similar. To create and use an extender, throw in some code like this:

``` javascript
ko.extenders.awesomeSauce = function (target, options) {
	
	//target is the subscribable object you are extending
	//	add stuff to it and you can use it in your viewModel and bindings

	//options is the object provided to the initialization of the extender
	//	use it to control what to add or how those additions behave
	
	return target; //if you don't return target, you might lose track of it
};

foo = ko.observable().extend({ awesomeSauce: { thing1: true, thing2: false }});
```

Ultimately, there's little difference between `fn` and extenders as far as the end result is concerned. They both allow you to extend a subscribable. These are really the only differences:

* You have to (or "can be") be more explicit when you want to add extenders to a subscribable by calling `.extend()`. No automatic inclusion as with `fn`.
* You can (or "have to") provide options when you add an extender. This allows you to be more precise about what exactly your subscribable is extended with or how that stuff behaves.

## Filters with extenders

{% jsfiddle ubpS7 js,html,resources,result %}

In the fiddle above, instead of putting all the filter functions I might want to use into the same bag, they are separated by name.

`arrayExtensions` is used to extend an `observableArray` with the filter functions `filter` and `orderBy`. These functions work exactly the same as in the `fn` example. However, in order to make the `arrayExtensions` methods chainable, I have to return a `subscribable` and extend it with `arrayExtensions`. This could be easy to forget to do when writing your own filter function. Also, this approach to chainability is not a frugal use of memory since two *new* functions are added to the target each time you call `extend({ arrayExtensions })`.

`formatting` is used to extend an `observable` with filter functions that are appropriate to its data type (provided in the extender options). Due to their simplicity, I did not make these filter functions chainable. I suppose they could be, but I couldn't think of a good reason to do that here.

To reiterate, in order to take advantage of these filter functions, I have to call `.extend()` for my `amount`, `date`, and `dudes` observables.

## Discussion and Gotchas

You can see that there is no difference between the `fn` and the extender approach in how the `data-bind` attributes in the HTML consume our filter functions. That's pretty cool.

``` html
<input type="text" data-bind="value: search, valueUpdate: 'afterkeydown'" />
<select data-bind="value: orderBy">...</select>

<table class="table">
    ...
    <tbody data-bind="foreach: dudes.filter(search, 'name').orderBy(orderBy)">
        <tr>
            <td data-bind="text: name"></td>
            <td data-bind="text: amount.currency()"></td>
            <td data-bind="text: date.date('dddd MMMM Do YYYY')"></td>
        </tr>
    </tbody>
</table>
```

While it's convenient for all functions you add to `fn` to be available to all subsequent instances of that type, *ALL* functions you add will be available to instances of that type. In the examples above, I added all my functions to `ko.subscribable.fn` to maximize chainability, but that small set of functions is already mutually-exclusive. `filter` and `orderBy` only make sense for observable arrays, `currency` only makes sense for numbers, and `date` only makes sense for dates. They are all there whether they make sense or not. In general, this is not a concern for performance or scalability because each `fn` function object will be shared by every `subscribable`.

Also, with the `fn` approach, putting everything in the same bucket is nice for availability, but if you start defining lots of `fn` functions in multiple files and possibly loading different subsets of those files on each page, you need to be pretty good at keeping those things straight, using unique function names, and keeping function names meaningful.

Chainability is powerful, but it can lead to long filter chains. In life, as in Knockout, you don't get money for nothing and chicks for free. Because we have used `ko.computed` at each step of the way, a change at a point in the filter chain will cause re-evaluation of the remainder of the chain. If you have 10 filters chained together and you're wondering why your page is sluggish, look there first.

I mentioned this above, but I wanted to reiterate it. Using the extender approach can be costly for memory consumption. Each time you `.extend()` a `subscribable`, new functions are added to the target. If you end up extending hundreds or thousands of objects on your page, you might start feeling the pain.

## Conclusion

I hope this was a useful demonstration of the power of `fn` and extender functions in KnockoutJS. In summary, if you are looking for simplicity and ease of use with respect to filtering, I'd say go with `fn` functions. If you need more precision, try extenders.