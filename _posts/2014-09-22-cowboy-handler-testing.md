---
layout: post
title: Unit testing your Cowboy Handlers
tags: cowboy, erlang, elixir
---

[Cowboy](https://github.com/ninenines/cowboy) is a "Small, fast, modular HTTP server written in Erlang.". This server uses **handlers** that describe the way your endpoints will behave during the lifetime of the request. Here's an example in Elixir from [HTTParrot](https://github.com/edgurgel/httparrot) using [cowboy_rest](http://ninenines.eu/docs/en/cowboy/1.0/manual/cowboy_rest):

{% gist edgurgel/8a47f95ace1f137d44a6 %}

`UserAgentHandler` when initated upgrades the handler to a `cowboy_rest` (webmachine-like way of handling REST resources). After that it specifies that it will accept only GET requests and that replies only with JSON.

One interesting property that you get from functional programming is the ability of testing them without knowing any context. That's how I think you should test your cowboy handler (testing 2 callbacks only for the sake of simplicity):

{% gist edgurgel/9267d93c30d8d4b38936 %}

In this case we are using [meck](https://github.com/eproxus/meck) to mock modules and expect on function calls.

The first test will guarantee that the handler will accept only GET requests. In this case, `cowboy_rest` will reply with status code `405 Method Not Allowed`, but you don't need to test this behaviour as long as you respect the output expected for this [callback](http://ninenines.eu/docs/en/cowboy/1.0/manual/cowboy_rest/#rest_callbacks_description). In this case it's a tuple of 3 items:

* List of accepted methods as strings;
* Request structure;
* The state of the handler that can be used on next steps of the processing;

Notice that it's not important which values `req` and `state` have. The only important thing is that you are not changing them. The assertion expects the same value passed as `req` and `state`.

The second test is a little bit more complicated, but nothing special. First we expect that `:cowboy_req.header/3` will be called with these args `"user-agent", :req1, "null"` and we define the return as: `{:user_agent, :req2}`.

Notice that again the `req` value is not important, but it's important to return a different `req` so you can track on which order it was executed as this is important to cowboy.

The second expectation is on `JSEX.encode!/1` waiting for this argument `[{"user-agent", :user_agent}]` and return `:json`. The assertion will wait the tuple of 3 items:

* Response body;
* Request structure;
* The state of the handler;

That's it! It's important to observe that this may not be enough if you have a really complicated handler. In this case you can write unit tests for each callback as I described and write integration tests without mocking anything and doing real HTTP requests.

