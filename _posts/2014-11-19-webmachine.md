---
layout: post
title: Webmachine - REST done right
---

Webmachine describes resources on top of HTTP to provide RESTful APIs. This is pretty much what every modern web framework states, right?

After writing some good amount of controllers on MVC frameworks you realise that it's complicated to keep a clean controller without using callbacks/middlewares to handle things like:

* Authorisation;
* Authentication;
* Conditional redirection;
* Caching;
* Content representation.

On Rails code, for example, It's common to find **before_filter**s (**before_action** on Rails 4) describing behaviour that should happen before handling the resource itself. Here's an example of discourse's [ApplicationController](https://github.com/discourse/discourse/blob/ec76be964ecfc2f1a750a8ca1bbd900953445608/app/controllers/application_controller.rb#L32-L41):

{% gist edgurgel/1952048af02e83dd69e5 %}

These callbacks will run doing assorted things on pretty much every controller on the app. Just by looking on the name of the callbacks you can notice some redirection, authorisation, authentication and content representation logic. One important thing is that the order here matters and you will probably hit lots of issues if you change the order of the **before_filter**s. There's also the problem that unit tests are complex as you need to test each callback indirectly.

Could it be less complicated? Of course!

## HTTP Protocol

Webmachine describes the request handling as a [state machine](https://raw.githubusercontent.com/wiki/basho/webmachine/images/http-headers-status-v3.png) where HTTP status codes are end states. It implements the HTTP protocol as a collection of functions that will describe the flow into this state machine. Below you can see the initial state and some transitions:

![Webmachine](http://i.imgur.com/sWjlYru.png)

Here's an [example](https://github.com/seancribbs/webmachine-ruby/blob/master/documentation/examples.md#get) from `webmachine-ruby`:

{% gist edgurgel/b8b4b85b7aea6598ba57 %}

This simple resource you will get automatically the following status:

* **405 Method Not Allowed** if it's not a GET;
* **406 Not Acceptable** if the request does not specify an `Accept` header that tolerates `application/json`;
* **200 OK** if everything is fine with and the correct content-type header will be provided.

Now let's handle when the order does not exist adding the `resource_exists?` method to the resource:

{% gist edgurgel/d52321df0e5e07c4eb7a %}

Now **404 Not Found** will be given if `resource_exists?` returns falsy value.

Let's now add authorisation and authentication:

{% gist edgurgel/f524b5be7c94a75be0cf %}

And again easily we will get:

* **401 Unauthorized** if `is_authorized?` returns falsy value;
* **403 Forbidden** if `forbidden?` returns falsy value

This is, in my opinion, a scalable way of describing your resources as the code will be as complex as the resources are. The framework already takes care of the nuances of the HTTP protocol keeping the code clean and extremely simple to unit test.

## More

The original implementation of Webmachine is in Erlang but you can find webmachine in multiple programming languages: [Erlang](https://github.com/basho/webmachine), [Ruby](https://github.com/seancribbs/webmachine-ruby), [Clojure](https://github.com/cmiles74/bishop), [Javascript](https://github.com/tautologistics/nodemachine), [etc](https://github.com/search?o=desc&q=webmachine&s=stars&type=Repositories&utf8=%E2%9C%93) ...

There's also a neat presentation by [Sean Cribbs](https://twitter.com/seancribbs): [Resources, For Real This Time (with Webmachine)](https://www.youtube.com/watch?v=odRrLK87s_Y)
