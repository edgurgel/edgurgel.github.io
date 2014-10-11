---
layout: post
title: Authentication on an Websocket API
---

Some months ago I was looking for a way to authenticate a websocket endpoint on [Poxa](https://github.com/edgurgel/poxa)'s console. The answer in the end was easy. The same way that Pusher signs HTTP requests: a simple HMAC-SHA256 applied on the payload with a secret. The interesting part is that the payload is well defined. Here's an example from the Pusher [website](https://pusher.com/docs/rest_api):

It's always composed as:

* HTTP method;
* URI path;
* Query string composed of `auth_key`, `auth_timestamp`, `auth_version` and any other user parameter.

```
POST\n/apps/3/events\nauth_key=278d425bdf160c739803&auth_timestamp=1353088179&auth_version=1.0&body_md5=ec365a775a4cd0599faeb73354201b6f
```

This string is joined by `\n` and then signed using HMAC-SHA256 with the secret related to this key.

The last key value added to the query string is `auth_signature` which is the result of signing.

Which properties do you get with this?

* Timestamp can avoid replay attacks in a certain period of time;
* Auth version will let you handle changes on the authentication process and treat them differently;
* The method won't let it be reused on the same URI with a different method;
* The path won't let someone use the same method on a different URI;

In the end, any change on the request will mess up with the expected signature and invalidate it.

If you notice that a Websocket connection is started as a GET request, you can easily use this signing method to achieve the same properties.

On the client side, in this case the browser, I just used [crypto-js](https://code.google.com/p/crypto-js/) with jQuery `param`:

{% gist edgurgel/727c3849347dbd41d261 %}

On the backend side I used [signaturex](https://github.com/edgurgel/signaturex)(written in Elixir), but you could use the original ruby library [signature](https://github.com/mloughran/signature).

That's it! Don't forget to use SSL on top of it so you can encrypt everything around.
