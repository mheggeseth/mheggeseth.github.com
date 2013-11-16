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

Next, we'll make a few simple changes to our JavaScript. First, we define a `viewModel` containing a single [observableArray]() to be a container for our sessions. Then we set the `sessions` observable to the array of sessions returned by the AJAX request.

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

Notice that there is no longer any indication that we are dealing with the HTML or the DOM at this point.

HTML changes

Loading indicator.

Apply bindings

Haven't changed the behavior of our page, but the addition of Knockout to manage our data and DOM interaction should make adding future behavior much easier. 

Here's the complete diff for our latest set of changes.

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