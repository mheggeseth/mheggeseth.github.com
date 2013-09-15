---
layout: post
title: "A Simple, Responsive Conference Session List (Part 1)"
date: 2013-09-12 22:55
comments: true
categories: 
published: false
---

August 2012 was the inaugural [That Conference](http://thatconference.com). It was an awesome conference (one that I will attend as long as it goes), but one thing bugged me: having just my iPhone, it was a challenge to peruse current and upcoming sessions using the conference's less-than-mobile-friendly website. So before I geared up to attend That Conference 2013, I decided to bring another tool: a responsive sessions list page.

### What I wanted

* View sessions as simply as possible: title, description, level, category.
* Hide irrelevant information. Specifically, hide sessions that have already happened.
* Mark sessions as favorites without needing an account.
* Tolerate inconsistencies in connectivity and bandwidth (it's a conference, after all).
* Provide a usable, consistent experience whether on mobile or not.

**To be clear,** the That Conference organizers did a great job creating mobile apps for 2013 (which I confess didn't know about when I wrote my page). I tried them and liked them, but there were a couple features my page had that were important to me so I ended up using my page for most of the conference. I'm willing to bet the mobile apps for 2014 will have useful features that I haven't even thought of.

### Part 1 of N...

To keep this from turning into *The Odyssey*, I'm breaking down the construction and evolution of the page into a series of posts. I'm not sure how many there will be yet. My intention is to go step-by-step through my "process" of building the page, paying special attention to the tools and techniques I used along the way as well as well as pointing out areas for improvement. Let's get started!

*Follow along! [Install Git](http://git-scm.com/downloads), clone the repo, then checkout the specified tag to get the full code state for that step.*
	git clone https://github.com/mheggeseth/ThatSessions.git

### Bootstrap boilerplate

	git checkout -f bootstrap-boilerplate

We'll start with the HTML boilerplate provided by [Bootstrap](http://getbootstrap.com/getting-started/). If you aren't famliar with Bootstrap, you're at least familiar with its style ([Twitter](http://twitter.com), [Angular](http://angularjs.org), and the web sites for many other open source frameworks). As a pretty, easy-to-use, responsive-by-default front-end library, it's an easy win for small projects.

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

I've already made one nice customization to the boilerplate, taking advantage of the [responsive utility classes](http://getbootstrap.com/css/#responsive-utilities) that come with Bootstrap. When the browser is narrow (i.e. mobile devices) shorten the header title to save space.

``` html
...
<a class="navbar-brand visible-xs" href="#">That / Sessions</a>
<a class="navbar-brand hidden-xs" href="#">That Conference / Sessions</a>
...
```

{% img /images/posts/bootstrap-boilerplate-screenshot-xs.png %}

### JSONP, OData, jQuery

