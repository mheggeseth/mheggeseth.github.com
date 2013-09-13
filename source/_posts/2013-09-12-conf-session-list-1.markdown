---
layout: post
title: "Revenge of the Simple, Responsive Conference Session List: Part 1"
date: 2013-09-12 22:55
comments: true
categories: 
published: false
---

August 2012 was the inaugural [That Conference](http://thatconference.com). It was an awesome conference (one that I will attend as long as it goes), but one thing bugged me: having just my iPhone in my arsenal, it was a challenge to peruse current and upcoming sessions using the conference's less-than-mobile-friendly website. So before I geared up to attend That Conference 2013, I decided to arm myself with another tool: a responsive sessions list page.

To be clear, the That Conference organizers did a lot of great work creating mobile apps for 2013 (which I confess didn't know about when I wrote my page). I tried them and liked them, but there were a couple features that my page had that were important to me so I ended up using my page for most of the conference. I'm willing to bet the mobile apps for 2014 will be a lot better and have useful features that I haven't even thought of.

### What I wanted

* View sessions as simply as possible: title, description, level, category.
* Don't show information that isn't relevant. Specifically, hide sessions that have already happened.
* Allow data persistence without account management
* Have a high tolerance for connectivity and bandwidth inconsistency (it's a development conference, after all)

### Part 1 of N...

I'm not sure how many there will be since I'm kind of making this up as I go. My intention is to go step-by-step through my "process" of building the page, paying special attention to the tools and techniques I used along the way. So let's get started.

### Bootstrap boilerplate

### JSONP, OData, jQuery

