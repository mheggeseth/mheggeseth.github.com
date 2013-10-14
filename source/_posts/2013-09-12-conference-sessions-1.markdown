---
layout: post
title: "A Simple, Responsive Conference Session List: Part 1"
date: 2013-10-13 22:55
comments: true
categories: responsive bootstrap jquery jsonp
---

August 2012 was the inaugural [That Conference](http://thatconference.com). It was an awesome conference (one that I will attend as long as it keeps going), but having just my iPhone with me, it was a challenge to peruse upcoming sessions using the conference's less-than-mobile-friendly website. So before I geared up to attend That Conference 2013, I built a mobile-friendly page for browsing sessions.

### What I wanted

* View sessions as simply as possible. Minimize cruft.
* Hide or diminish irrelevant information. Specifically, hide sessions that have already happened.
* Track favorite sessions without accounts.
* Tolerate inconsistent connectivity and bandwidth (it's a conference, after all).
* Provide a usable, consistent experience at any screen width.

**To be clear,** the That Conference organizers did a great job creating a mobile app for 2013 (which I didn't know about when I wrote my page). I tried it and liked it, but there were a couple features my page had that were important to me so I ended up using my page for most of the conference. I'm looking forward to seeing how the mobile app improves for 2014. It will probably have useful features that I haven't even thought of.

### Part 1 of N...

To keep this post from becoming *The Odyssey*, I'm breaking down the construction and evolution of the page into a series of posts. I'm not sure how many there will be yet. My intention is to go step-by-step through my "process" of building the page, paying special attention to the tools and techniques I used along the way as well as well as pointing out difficulties and areas for improvement. Let's get started!

*Want to follow along? [Install Git](http://git-scm.com/downloads), clone the repo below, then checkout the specified tag to get the full code state for that step.*
	git clone https://github.com/mheggeseth/ThatSessions.git

## Bootstrap boilerplate

	git checkout -f bootstrap-boilerplate

We'll start with the HTML boilerplate provided by [Bootstrap](http://getbootstrap.com/getting-started/). If you aren't famliar with Bootstrap, you're at least familiar with its style ([Twitter](http://twitter.com), [Angular](http://angularjs.org), and the web sites for many other open source frameworks). As a nice-looking, easy-to-use, responsive-by-default front-end library, it's an easy win for small projects.

``` html
<!DOCTYPE html>
<html>
  <head>
    <title>That Conference / Sessions</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet" media="screen">
  </head>
  <body>
    <nav class="navbar navbar-inverse" role="navigation">
        <div class="navbar-header">
            <a class="navbar-brand visible-xs" href="#">That / Sessions</a>
            <a class="navbar-brand hidden-xs" href="#">That Conference / Sessions</a>
        </div>
    </nav>

    <div class="container">
        <div class="jumbotron">
            <div class="container">
                <h1>Hello, world!</h1>
                <p>Do you think we should write some code to load the sessions? Yes.</p>
            </div>
        </div>
    </div>

    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="//code.jquery.com/jquery.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="js/bootstrap.min.js"></script>
  </body>
</html>
```

{% img /images/posts/bootstrap-boilerplate-screenshot.png %}

I've already made one nice customization to the boilerplate, taking advantage of the [responsive utility classes](http://getbootstrap.com/css/#responsive-utilities) that come with Bootstrap. When the browser width is narrow (i.e. mobile devices) shorten the header title to save space. The `visible-xs` class ensures an element will only be visible when the browser width is "extra small". `hidden-xs` ensures an element is always hidden when the browser width is narrow.

``` html
...
<a class="navbar-brand visible-xs" href="#">That / Sessions</a>
<a class="navbar-brand hidden-xs" href="#">That Conference / Sessions</a>
...
```

{% img /images/posts/bootstrap-boilerplate-screenshot-xs.png %}

This will come in handy later in case we want to add other controls to the page header and keep them tidy on mobile devices.

### Tangent: Ad-Hoc HTTP Server with Node and `http-server`

If you have a favorite way of spinning up ad-hoc local HTTP servers, vaya con dios. For me, I turn to the [http-server](https://github.com/nodeapps/http-server) package for [Node](http://nodejs.org). Install it globally and you can fire up an HTTP server from any location with `http-server`. It even gives you a unique port if you spin up multiple servers. Not too bad.

## Get me some sessions (with help from OData, jQuery, & JSONP)

    git checkout -f load-sessions

So we have a fairly nice-looking page that does exactly nothing. Let's get some sessions. 

First I have to figure out how to do that. I'm somewhat familiar with [OData](http://www.odata.org/) so that's what I'll sniff out first. If you aren't familiar, OData is short-hand for the Open Data Protocol: an effort to document and encourage a standard for data APIs on the web.

And I think I might be in luck. Taking a look at the kind of AJAX-y stuff the That Conference home page is accessing using the Chrome Developer Tools, it looks like the site exposes an OData API.

{% img /images/posts/odata-discovery.jpg %}

Let's see what we get at `thatconference.com/odata/api.svc/?$format=json`.

{% img /images/posts/odata-discovery-1.jpg %}

Mmmm, `Sessions`. That sounds good. I'll have that. Let's see what's behind `thatconference.com/odata/api.svc/Sessions?$format=json`.

{% img /images/posts/odata-discovery-sessions.jpg %}

Data. Just as I thought. Let's get this sweet, sweet data in our page using our old friend jQuery.

We replace our "hello world" hero box with

``` html
<ul id="sessionList" class="list-unstyled"></ul>
```

and get the sessions using a jQuery `$.ajax` call with JSONP:

``` html
<script type="text/javascript">
    $.ajax({
        url: "http://www.thatconference.com/odata/api.svc/Sessions?$format=json&$callback=?",
        dataType: "jsonp",
        success: function (data) {
            var sessions = data.d;
            var $sessionList = $("#sessionList");
            $.each(sessions, function(index, session) {
                $("<li></li>").text(session.Title).appendTo($sessionList);
            });
        }
    });
</script>
```

### Tangent: JSONP

For security reasons, browsers don't allow AJAX requests to different domains than the current page. This is a good thing, but what if you want to do something legit like show recent tweets or load some conference sessions. JSONP is a clever trick for doing that. While you can't do AJAX requests to any old domain, you can load JavaScript from any old domain.

JSONP says, "If I add a `<script>` tag to this page with the URL of a remote API I want to call and I also provide it the name of a callback function, you (the remote API) promise to return valid JavaScript invoking that function with the data I've requested."

### JSONP/OData Caveat

JSONP is not part of the OData spec, however JSONP support can easily be added to a WCF Data Service which natively supports OData. It just so happens that `thatconference.com/odata/api.svc` is one of these.

## Result

{% img /images/posts/load-session-list.jpg %}

It's a pretty bland and useless list of sessions. But we'll change that.

In the next post, we'll add a proper view model with [KnockoutJS](http://knockoutjs.com) and see what we can do with Bootstrap to improve our style.



