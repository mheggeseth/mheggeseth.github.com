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

[Last time](/blog/2013/10/13/conference-sessions-1/), we used jQuery, JSONP, and [OData](http://www.odata.org/) to load a list of conference sessions into some HTML [Bootstrap](http://getbootstrap.com/getting-started/) boilerplate. We got stuff to work, but it's not very pretty or useful yet so let's change that.

{% img /images/posts/load-session-list.jpg %}

## Enter KnockoutJS




``` diff
     <div class="container">
-        <ul id="sessionList" class="list-unstyled"></ul>
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