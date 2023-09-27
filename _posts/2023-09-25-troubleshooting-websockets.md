---
layout: post
title: Troubleshooting Express.js WebSockets
---

One day while checking our Stripe API usage we discovered that we had some very large traffic spikes, much higher than what we were expecting, peaking at nearly 20k requests per day.

<img src="/assets/img/fun-with-websockets/stripe-graph-before.png" style="max-width: 450px" />

While cross referencing with our server logs, we couldn't see any traffic spikes that correlated to the Stripe API spikes. This was very weird since we knew that the Stripe API calls were coming from our code and our servers.

After several days and a lot of debugging we eventually figured out that our WebSocket server was the culprit. This led us to discovering two issues in our code:

1. WebSocket connections bypass the default logging library
2. Session changes made by middleware in the WebSocket chain were not saved

---

We use WebSockets to send real-time updates from our backend to the frontend. Our [`express`](https://expressjs.com/) web server uses the [`express-ws`](https://www.npmjs.com/package/express-ws) library to easily add a WebSocket endpoint, which works well with regular Express middleware.

```js
app.ws('/websocket-endpoint', () => {
    
});
```

## Properly logging WebSocket connections

We use [`morgan`](https://www.npmjs.com/package/morgan) to log our web server activity, which can be easily added as a simple middleware:

```js
app.use(morgan('dev'));
```

However, it turns out that the default configuration for `morgan` does not catch and log any WebSocket connections, which is why were unable to correlate our Stripe API usage with our web server traffic.

In order to make morgan catch WebSocket connections we had to configure it to run in immediate mode (which logs requests as soon as they come in, rather than waiting for a response).

We created a helper function to initialize the morgan middleware (our real function is more complicated than this though, since we have a custom morgan format).

```js
function loggingMiddleware(websocket) {
  // To catch websocket connections, set immediate mode to true
  const immediate = !!websocket;
  return morgan('dev', { immediate });
}
```

Then we simply have to add the logging middleware twice, one time for "normal" requests without immediate mode, and another time only on the WebSocket endpoint with immediate mode enabled.

```js
// Normal logging
app.use(loggingMiddleware(false));

// WebSocket + WebSocket logging
app.use('/websocket-endpoint', loggingMiddleware(true));
app.ws('/websocket-endpoint', function () {
  
});
```

This worked, and we were now able to clearly see that the spikes in traffic were indeed coming from WebSocket connections, which correlated with the increase in Stripe API calls.

## Sessions and WebSockets not cooperating

So what was causing all of the Stripe API calls from our WebSocket connections? Well, it turns out that session changes made by middleware running earlier in the request chain were not getting saved.

Previously our middleware chain looked like this. We were looking up the billing status for the logged in user before hitting the WebSocket endpoint:

```js
app.use(billingStatusMiddleware);
app.use(initWebsocketRoutes());
```

The billing middleware calls the Stripe API to check the user's subscription, and then caches this result in the user's session. However, for some reason, this data was not getting saved to the session for the WebSocket route. So, the billing status was never cached, and every single connection attempt was triggering a Stripe API call.

The fix for this was quite simple, since we don't even need the billing status in the WebSocket route we just changed the order of the middleware, to make sure the WebSocket endpoint is initialized further up in the chain.

```js
app.use(initWebsocketRoutes());
app.use(billingStatusMiddleware);
```

Perhaps sessions just don't work with WebSockets. I'm not sure, but in any case, this solved our problem at least.

<img src="/assets/img/fun-with-websockets/stripe-graph-after.png" style="max-width: 450px" />