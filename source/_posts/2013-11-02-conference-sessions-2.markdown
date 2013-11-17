---
layout: post
title: "ThatConference Session List 2: Knockout My Bootstrap!"
date: 2013-11-02 13:42
comments: true
categories: knockoutjs bootstrap
published: false
---

``` bash
# REMEMBER: follow along with these examples by cloning the
#   repository and checking out the tag for each step
$ git clone https://github.com/mheggeseth/ThatSessions.git
$ git checkout -f [tag-name-for-step]
```

[Last time](/blog/2013/10/13/conference-sessions-1/), we used [jQuery](http://jquery.com/), JSONP, and [OData](http://www.odata.org/) to load a list of conference sessions into some HTML [Bootstrap](http://getbootstrap.com/getting-started/) boilerplate. We got loading to work, but our page isn't very pretty or useful yet so we're going to fix that (at least part way).

{% img /images/posts/load-session-list.jpg %}

## Enter KnockoutJS

    git checkout -f knockoutjs

Obviously, we'll add the [KnockoutJS](http://knockoutjs.com) library.

``` html
<script type="text/javascript" src="js/knockout-2.3.0.js"></script>
```

Next, we'll make a few simple changes to our JavaScript.

``` javascript
var viewModel = {
    sessions: ko.observableArray()
};

$.ajax({
    url: "http://www.thatconference.com/odata/api.svc/Sessions?$format=json&$callback=?",
    dataType: "jsonp",
    success: function (data) {
        viewModel.sessions(data.d);
    }
});
```

 First, we defined a `viewModel` object containing a single [observableArray](http://knockoutjs.com/documentation/observableArrays.html) to be a container for our sessions. Then we set the `sessions` observable to the array of sessions returned by the AJAX request.

Notice that our JavaScript has now lost any notion of dealing with HTML or the DOM. This is a good thing. A nice benefit of using Knockout's MVVM pattern is separating your data (the view model) from how the data is presented in HTML markup (the view). This allows the HTML to accurately describe, to a great extent, the behavior of the page. This is a distinct advantage over jQuery, whose behavior is generally defined in JavaScript and you are left to guess what role markup elements play based on context clues in their IDs or CSS class names. 

Let's update our HTML using Knockout's `data-bind` attribute to bind our session list to the page.

``` html
<div class="container">
    <span data-bind="visible: !sessions().length">Loading sessions...</span>
    <ul class="list-unstyled" data-bind="foreach: sessions">
        <li data-bind="text: Title"></li>
    </ul>
</div>
```

The `foreach: sessions` binding tells Knockout to repeat the element's inner HTML for each object in the `sessions` array. The binding context of the inner HTML of `<ul data-bind="foreach: sessions">` then becomes the current element of `sessions`. So `<li data-bind="text: Title"></li>` will add a list item for each session whose value is the `Title` property of the session.

Notice that we added another element in there: `<span data-bind="visible: !sessions().length">Loading sessions...</span>`. Knockout's `visible` binding shows the current element if its value is truthy and hides it when its value is falsey. This allows us to easily provide a friendly loading message before sessions are loaded and then take it away once we've loaded at least one session.

We forgot to do one thing, probably the most important thing. We need to tell Knockout to apply the bindings in the HTML to a view model. We add a call to `ko.applyBindings` to the end of our JavaScript after we've defined out view model. Without this, the `data-bind` attributes we added to the markup are about as useless as a white crayon.

``` javascript
//find KO binding declarations and associate them with target viewModel members
ko.applyBindings(viewModel);
```

So this is all fine and good, but you've probably noticed that with the exception of the loading indicator, we haven't changed the look and feel of our page at all. However, the addition of Knockout to manage data and its application to the DOM should pay big dividends in the future. For one, because `sessions` is not just any array but an observable array, if we added or removed a session, Knockout would automatically update the view, a task that would have required significantly more code with our previous jQuery setup.

For reference, here's the diff for our latest set of changes to `index.html`.

``` diff
     <div class="container">
-        <ul id="sessionList" class="list-unstyled"></ul>
+        <span data-bind="visible: !sessions().length">Loading sessions...</span>
+        <ul class="list-unstyled" data-bind="foreach: sessions">
+            <li data-bind="text: Title"></li>
+        </ul>
     </div>
 
     <script type="text/javascript" src="js/jquery-1.10.2.min.js"></script>
+    <script type="text/javascript" src="js/knockout-2.3.0.js"></script>
     <script type="text/javascript" src="js/bootstrap.min.js"></script>
     <script type="text/javascript">
+        var viewModel = {
+            sessions: ko.observableArray()
+        };
+
         $.ajax({
             url: "http://www.thatconference.com/odata/api.svc/Sessions?$format=json&$callback=?",
             dataType: "jsonp",
             success: function (data) {
-                var sessions = data.d;
-                var $sessionList = $("#sessionList");
-                $.each(sessions, function(index, session) {
-                    $("<li></li>").text(session.Title).appendTo($sessionList);
-                });
+                viewModel.sessions(data.d);
             }
         });
+
+        //find KO binding declarations and associate them with target viewModel members
+        ko.applyBindings(viewModel);
     </script>
   </body>
```

## Bootstrap Window Dressing

    git checkout -f bootstrap-style

So now that we have the beginnings of an application, let's make it look a little bit nicer.

nav bar

list group style

more data

``` diff
@@ -4,13 +4,15 @@
     <title>That Conference / Sessions</title>
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <!-- Bootstrap -->
-    <!-- <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css"> -->
     <link rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
-    <!-- <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap-theme.min.css"> -->
     <link rel="stylesheet" type="text/css" href="css/bootstrap-theme.min.css">
+    <style type="text/css">
+        /* Necessary for fixed header http://getbootstrap.com/components/#navbar-fixed-top */
+        body { padding-top: 70px; }
+    </style>
   </head>
   <body>
-    <nav class="navbar navbar-inverse" role="navigation">
+    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
         <div class="navbar-header">
             <a class="navbar-brand visible-xs" href="#">That / Sessions</a>
             <a class="navbar-brand hidden-xs" href="#">That Conference / Sessions</a>
@@ -18,17 +20,20 @@
     </nav>
 
     <div class="container">
-        <span data-bind="visible: !sessions().length">Loading sessions...</span>
-        <ul class="list-unstyled" data-bind="foreach: sessions">
-            <li data-bind="text: Title"></li>
+        <div class="alert alert-info" data-bind="visible: !sessions().length">Loading sessions...</div>
+        <ul class="list-group" data-bind="foreach: sessions">
+            <li class="list-group-item">
+                <span class="label label-primary" data-bind="text: new moment(ScheduledDateTime).format('ddd M/D/YY h:mm a') + ' | ' + ScheduledRoom"></span>
+                <span class="label label-default" data-bind="text: Category + ' | ' + Level"></span>
+                <h4 class="list-group-item-heading" style="margin-top:5px" data-bind="text: Title"></h4>
+            </li>
         </ul>
     </div>
 
-    <!--<script src="//code.jquery.com/jquery.js"></script>-->
     <script type="text/javascript" src="js/jquery-1.10.2.min.js"></script>
     <script type="text/javascript" src="js/knockout-2.3.0.js"></script>
-    <!--<script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0/js/bootstrap.min.js"></script>-->
     <script type="text/javascript" src="js/bootstrap.min.js"></script>
+    <script type="text/javascript" src="js/moment.min.js"></script>
     <script type="text/javascript">
         var viewModel = {
             sessions: ko.observableArray()
```

## Next Time: Functionality!

