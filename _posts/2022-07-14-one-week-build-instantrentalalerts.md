---
layout: post
title: "Building a SaaS product in 1 week"
tags: ["One Week Build", "SaaS", "Node.js", "Web App"]
image: https://instantrentalalerts.nl/img/share.png
---

A one week build: [InstantRentalAlerts.nl](https://instantrentalalerts.nl/): Instant email alerts for new rentals in the Netherlands. Built with [node.js](https://nodejs.org/en/) and deployed on [DigitalOcean App Platform](https://m.do.co/c/0ffbbf933d7c).

## Problem

After deciding to move to the Netherlands, one of the first things I heard was "there is a housing crisis here". Rentals are very competitive and viewing time slots fill up quickly.

My rental hunt started by manually checking various listing sites, but this quickly became quite tedious. Most of the main rental websites offer email notifications, however, they're limited to 1 notification per day.

I wanted something to notify me as soon as a new apartment became available. The faster I discovered it, the quicker I could contact the agent and arrange a viewing. 

## Solution

A simple web service that monitors rental websites for new listings and sends email alerts as soon as something new is detected.

Users are able to create an account and subscribe to new rentals based on some basic search preferences like city, size, price, etc.

The service operates like a web crawler, generating a small amount of traffic on the rental sites. It respects robots.txt and only extracts Open Graph and JSON-LD metadata, just as any well behaving web crawler would.

The service redirects users to the original rental website as to not attempt to clone/sell their data, or "steal" any traffic they would otherwise have received.

## Build

The build itself is actually quite boring. I didn't use any fancy modern frameworks or shiny new web technologies. Most of it is simply just server-side rendered HTML. For an overview of the tech and services I used for the build, head over to the [Tech Stack](https://instantrentalalerts.nl/stack).

[![](/assets/img/instantrentalalerts/stack.png)](https://instantrentalalerts.nl/stack)

### Backend

- **[Node.js](https://nodejs.org/en/)** - I'm still a huge fan of node.js, I love working with it and it gets the job done fast. I'm sure I'll eventually branch out and try something else, but not right now.

- **[Express.js](https://expressjs.com/)** - Still my go-to web framework. One must-use middleware is [`express-async-errors`](https://www.npmjs.com/package/express-async-errors), which automatically handles error routes for async handlers.

- **[pug](https://pugjs.org/api/getting-started.html)** - HTML templates, quick and easy to use, with useful features like layouts, mixins, and custom helper functions. Still one of my favorites.

- **[Stylus](https://stylus-lang.com/)** - CSS generator, again very quick and easy to use, supporting nesting and user-defined constants.

### Frontend

Just plain old vanilla HTML and a little bit of JS. No fancy frontend framework required here. There are just a couple of lines of JavaScript to reload the Live feed and Histogram.

The live feed and histogram are just plain old HTML rendered server-side. They are updated using the browser `fetch` API, and then replacing the `innerHTML` of the target containers. Super simple and works great.

```js
function watch(target, url, interval) {
  const el = document.querySelector(target);
  function update() {
    fetch(url).then(resp => {
      resp.text().then(text => {
        el.innerHTML = text;
      });
    });
  }
  setInterval(update, interval * 1000);
  update();
}

// update live feed every 10 seconds
watch('#preview', '/_preview', 10);

// update historgram every 30 seconds
watch('#histogram', '/_histogram', 30);
```

### Emails

**[Postmark](https://postmarkapp.com/)** is my go-to provider for transactional emails. I've used them for many years now and I have *never* had any deliverability issues. I've tried many other providers, but I've always had issues with deliverability to hotmail, yahoo, and outlook addresses.

I like Postmark's simple yet functional UI, and their pricing is fair. New accounts start with 100 free emails (great for prototyping). They also maintain a suppression list for "unsubscribed" email addresses, so you don't have to deal with that yourself.

### Authentication

User account management is always the thing I dread the most for new projects. I hate re-building the same sign up, login, password resets, account activation over and over again. I'm sure there are libraries, frameworks, or services I could use for this, but I'd rather not have to pull in a massive dependency or a 3rd party service.

Instead, this time I decided to go for **passwordless magic links**. This really simplifies things, since the sign up, login, and account activation steps can all be combined into a single flow. And in this case, password reset aren't needed at all.

1. Login with email
2. User doesn't exist yet? -> Create user
3. Generate login token and save to database
4. Send email with login link
5. User clicks link
6. User not activated yet? -> Activate user
7. Done

![](/assets/img/instantrentalalerts/login.png)

For pre-validating email addresses I use [`isemail`](https://www.npmjs.com/package/isemail), which helps to filter out junk input. Also don't forget to normalize your email address input using `toLowerCase()` to avoid duplicate accounts with different casing (you'd be surprised how often this happens due to auto-completion).

### Database

- **Postgres** - Since I knew I'd be hosting this on DigitalOcean's App Platform I figured using a Postgres database would be most convenient. App Platform provides an addon "dev" database. Totally worth it for zero hassle.

- **[sequelize](https://www.npmjs.com/package/sequelize)** - I've been using Sequelize as my node.js postgres ORM. While it definitely isn't perfect, it does do a fantastic job for simple applications. It automatically syncs my table structure, and it's nice not having to write any SQL. Also, [`connect-session-sequelize`](https://www.npmjs.com/package/connect-session-sequelize) works great with [`express-session`](https://www.npmjs.com/package/express-session) for session storage.

- **[TablePlus](https://tableplus.com/)** - This is my go-to desktop database UI on macOS. I'm still on the free version, which works great. But I should probably upgrade at some point, mainly to support the product.

### Billing

This used to be a pain point and lot of work. However, with all of Stripe's latest "low-code" and "no-code" additions this has become so much easier.

![](/assets/img/instantrentalalerts/billing.png)

- **Stripe Checkout for new customers** - I create a new Stripe customer for the user (if they don't already have one) and then create a checkout session using the Stripe API.

- **Stripe Hosted Customer Portal for existing customers** - Existing customers who want to cancel or update their payment method get redirected to Stripe's hosted customer portal page. This saves a lot of work, and has some nice features, like offering to pause/resume subscriptions rather than canceling. You can also ask customers why they decided to cancel their subscription, another feature I now don't have to build myself.

- **Stripe Tax for EU VAT collection** - Since my company is registered within the EU I have to collect VAT on all purchases made within the EU. Stripe can take care of this as well, automatically applying the correct EU VAT rate depending on the customer's billing country.

- **Webhooks for tracking subscription status** - Rather than polling Stripe's API (which is a little slow) to check that a user still has an active subscription, I instead use webhooks to get notified any time a user's subscription status changes. Then I just maintain a flag in my user database to keep track of which users are actively paying for the service.

  - `customer.subscription.created` -> `active = status === 'active'`
  - `customer.subscription.updated` -> `active = status === 'active'`
  - `customer.subscription.deleted` -> `active = false`

### Deployment + Hosting

- **[DigitalOcean App Platform](https://m.do.co/c/0ffbbf933d7c)** - I've been using DigitalOcean's App Platform for quite some time now and it has really sped up my development flow. It's cheap, easy to use, and works great. Check out my other [blog post](https://pqvst.com/2021/09/30/digitalocean-app-platform/) for more details.

- **[Hover](https://hover.com/DhR6YHsN)** - Hover is still my go to service for purchasing domains. Not much to say here except I've been a happy customer for many years.

- **[Cloudflare](https://www.cloudflare.com/)** - For DNS I use Cloudflare. One of the benefits with using Cloudflare is that it makes it very easy to force HTTPS, redirect HTTP to HTTPS, and setup a Page Rule for www redirects.

### Analytics

I use self-hosted **[Umami](https://github.com/umami-software/umami)** for web analytics. Check out my other [blog post](https://pqvst.com/2021/07/26/self-hosted-analytics-with-umami-and-traefik/) for more about that!

### Pricing

For pricing I decided on €10 per month. This should be enough to cover any costs with sending email alerts (even for someone who decides to get alerts for every single new listing).

The price is slightly higher than what I initially thought, since I'm assuming most people will probably only subscribe for 1 or maybe 2 months, during the time they're hunting for a new place to live (based on what I've heard from others and my own experience).

An alternative idea I had was to use dynamic pricing. For example, with few users the price goes down. As there are more users the usefulness of the service will go down (since the competition increases), at which point the price can increase. In a way, balancing supply and demand. It's a fun concept, but perhaps tricky to execute.

## Profit/Cost Analysis

Here's a quick breakdown of all my costs for running the service, and the fees that Stripe charges me for billing.

Once you start using all of Stripe's "low-code" and "no-code" features, the fees start to add up. But, I am **more than happy** to pay 5% considering how much development time it saves me. Also, I can of course always get rid of the Customer portal and Stripe Tax fees in the future by building those features myself, if I wanted to.

```
Server Costs
------------
  App Platform
    App ............... $5/month
    Database .......... $12/month
  Domain(s) ........... $10/year

Billing Costs (per transaction)
-------------------------------
  Stripe
    Fixed fee ......... €0.25
    Variable fee ...... €0.14 (1.4%)
    Customer portal ... €0.05 (0.5%)
    Stripe Tax ........ €0.05 (0.5%)
                      = €0.41

Profit (per transaction)
------------------------
  Revenue ............. €10.00
  Billing Fees ...... - € 0.41 ( 4.9%)
  Net Profit ........ = € 9.51 (95.1%)
```

## Future Ideas

- Add other sign in methods, like Sign in with Google. I'm also interested in trying to add "Sign in with Apple" as another login option. I started looking into this but it seems you need an Apple developer account in order to do that, which costs $100/year.

- Offer Other notification channels or RSS

- Option to reduce sending frequency (i.e. only send 1 email per hour)

- Add other countries/cities

## My other "One week builds"

- [One Week Build: Browserframe 2.0](https://pqvst.com/2019/06/14/one-week-build-browserframe-2-0/)
- [One Week Build: Hyperbnb.com](https://pqvst.com/2017/01/23/one-week-build-hyperbnb-com/)
- [One Week Build: BrowserFrame.com](https://pqvst.com/2016/11/17/one-week-build-browserframe-com/)
- [One Week Build: Paperworks.io](https://pqvst.com/2016/09/28/one-week-build-paperworks-io/)