---
title: Cloning Disqus - Introducing CommentR
layout: post
---

This blog is [open source](https://github.com/jdaigle/jdaigle.github.io) and is built using [Jekyll](http://jekyllrb.com/), a [static website generator](https://staticsitegenerators.net/). One of the interesting side effects of hosting a completely *static* website is that I don't have a *dynamic* backend that I can use for comments. Comments, at a minimum, require a) a database b) a form to post comments and c) a dynamically generated view.

In the world of static websites, a new category of SaaS as sprung up around "blog hosting services". By far the most popular service is [Disqus](https://disqus.com/), but there are others such as [Discourse](https://www.discourse.org/).

Rather than simply pull in one of these services, I decided to try building my own! I decided to base my design on Disqus. Except my service won't serve ads, does not require creating an account, and won't track you.

The bare minimum requirements:

* It must be easy to add to a page: i.e., include a single JavaScript reference and a placeholder element.
* It must allow posting of comments without creating an account: just an author and body input.

Eventually I might add these features:

* Moderation: I should be able to delete comments, or approve comments before they're visible.
* Moderator comments: My comments should be highlighted in some way.
* Self-Service: Someone who has just posted a comment should be able to edit or delete that comment.
* SPAM Detection: I don't want a CAPTCHA, so I might build some trivial SPAM detector.

I decided to name my project **CommentR**. It's open source and available on [GitHub](https://github.com/jdaigle/CommentR).

## Building CommentR

Architecturally I knew that CommentR would be a simple API service. So I started with two simple API calls:

    curl -X GET /Comments?permalink=XXX
    
    curl -X POST --data "permalink=XXX&author=YYY&body=ZZZ" /Comment

The `GET` call returns the visible comments for a page (keyed using a `permalink` URL) either as JSON or an HTML fragment depending the HTTP Accept header. The `POST` call submits a new comment for a page (again keyed using a `permalink` URL) and returns the same result as the `GET` call.

And for the simplest level of "security" the API checks the HTTP Refer header from a known whitelist and rejects HTTP requests that are invalid.

To integrate this into a web page I built a really simple JavaScript file that's hosted alongside the API which simply loads the comments and appends them to the page along with a form. The form, when submitted, submits a new comment and reloads the comments.

This approached worked fine on my computer, but I soon as I deployed it to an Azure site and tried it I ran into the dreaded [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) issue. Rather than open up that can of worms, I decided to change gears and implement CommentR using an IFrame.

## The Final Design: IFrames

As it turns out, this is the approach that Disqus uses! It's actually very simple:

* Include a reference to some JavaScript resource. This code is responsible for setting up the IFrame and setting the correct src.
* The IFrame src is a page that displays the comments specifically for the parent page (keyed using the same `permalink` URL as before).
* All AJAX calls occur from within the IFrame, so the same-origin policy applies.

One clever trick I implemented was to have the embedded page "publish" a message whenever the content is loaded with the total scrollHeight of the content. The original JavaScript resource in the parent window responds by dynamically re-sizing the IFrame. So for the user, the comments window is completely seamless.

You should be able to see the final working version below. Leave a comment!
