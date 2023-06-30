---
layout: post
title: Going Multi-Region with Zero Devops
---

I've been a huge fan of DigitalOcean for many years, and one of my SaaS businesses, [AwardFares](https://awardfares.com), has lived there for the past 5 years. We initially ran everything on a couple of Droplets (Ubuntu VPS that we managed ourselves). When DigitalOcean announced their [app platform](/2021/09/30/digitalocean-app-platform/) we immediately migrated our app to that, reducing devops to nearly zero. Our database, however, remained as a single mongodb instance running on a self-managed VPS.

## What's the problem?

1. We started to hit the limits of our single VPS mongodb instance, and for a long time we had been looking to move to a managed mongodb solution to improve updates, maintenance, security and scalability.

2. Our app was hosted in a single region on the DigitalOcean app platform. The majority of our users (Europe) experience a snappy low latency service. However, accessing our service from other regions, like North America and Asia is significantly slower. As a larger proportion of new users are coming from outside of Europe, we want to make sure these users also have a good experience.

## What are our requirements?

We evaluated many different options for how to deal with these issues, but let's start by going through what are main requirements are:

1. Reducing devops workload is my personal #1 priority.
2. Our app should be multi-region. Initially Europe and North America, but with the possibility to add other regions (like Asia) in the future.
3. Our database *reads* should be multi-region, meaning our primary node would be located in Europe, with additional nodes or read replicas available in other regions.
4. Our app and database should be located in the same data centers.

## Goodbye DigitalOcean, Hello Render & MongoDB Atlas

Based on these requirements we concluded that it no longer made sense for us to stay at DigitalOcean. The primary reason for this is that while DigitalOcean does offer a managed mongodb service, it does not support multi-region nodes/replicas. So this meant that we would be moving our database to a different provider, in our case we chose MongoDB Atlas.

Since DigitalOcean use their own data centers and do not offer hosting in other cloud providers (like GCP or AWS), this meant that we would also need to move our app hosting, in order to keep our app and database in the same data center or cloud platform. For this we could have chosen to host our app directly on GCP, AWS, Azure, but this would have been a nightmare of devops.

We also considered Heroku, however the UI just wasn't very appealing (felt very clunky) and adding static outbound IP addresses was surprisingly expensive (something we wanted to have for whitelisting access to our database).

In the end we decided to try out [Render](https://render.com/). It fulfilled all of our requirements, and we've been very happy with our choice so far. Furthermore, since our apps are just a couple of Dockerfiles, moving to a different PaaS was surprisingly easy.

## Zero Devops Multi-Region

So how did we manage to go multi-region with practically zero devops? There are really only three components we needed:

1. App Hosting: Render (AWS)
2. Database Hosting: MongoDB Atlas (AWS)
3. Load Balancer with Geo-routing: CloudFlare

![](/assets/img/zero-devops-multi-region/flowchart.png)

Let's take a look at each component in more detail.

### 1. App Hosting on Render

First of all, the Render dashboard was really a breath of fresh air. DigitalOcean has been great, and easy to use, but it does feel sluggish at times. The Render dashboard is simple, clean, and very snappy.

![](/assets/img/zero-devops-multi-region/render.png)

It also comes with all of the essential features that we've come to enjoy at DigitalOcean:

- Auto-deploy from GitHub
- Built-in log streaming
- Environment variables and secrets (Env Groups are great!)
- Fast builds

Render doesn't have built-in support for multi-region app hosting (although there is a 3 year old feature request "under review"). However we can solve this by hosting multiple instances of our app, each in a different region. We then just need an extra loud balancer on top to direct users to the correct instance (more on this below).

Some minor oddities we've noticed with Render. Their zero downtime mechanism means that when you deploy a new version, the old version and new version will both be available for a brief period of time. This is fine for web apps, but the same applies to background workers as well (which makes no sense to me). There is a nice workaround for it though, attaching a 1GB disk ($0.25/month) prevents zero downtime.

### 2. Migrating to MongoDB Atlas

MongoDB Atlas was the easy choice in terms of managed mongo hosting. We didn't really explore other options, and Atlas already comes with everything we need. The most important feature for us was multi-region read replicas. Our primary database would still be located in Europe, however, by adding read replicas in other regions we will hopefully be able to make our search queries as responsive as possible for users in other regions. We haven't done much performance testing on this yet, but hopefully something we'll have a chance to dig into more later.

![](/assets/img/zero-devops-multi-region/mongo.png)

Some added bonuses with moving to Atlas is the extensive built-in monitoring, auto-scaling, and backups, which means we can remove most of our existing "devops" chores. The "Performance Advisor" is also something we will spend some time exploring, to see if we can figure out how to make some improvements to our indexes.

### 3. CloudFlare DNS Load Balancing

Given that we will have a separate deployment of our app in each region that we want to support, we need some way of routing users in different regions to the correct origin. Since we already use CloudFlare for our DNS, we figured it would make sense to try CloudFlare's Load Balancer, which does have support for "geo routing" making it possible to direct users to different origin servers based on their geographic location.

![](/assets/img/zero-devops-multi-region/cloudflare2.png)

The process was fairly straightforward, but there was a lot of new terminology that we weren't familiar with. Also, CloudFlare's pricing information for Load Balancers was quite confusing, and we weren't always sure what "add-ons" we actually needed.

In the end we figured out that we needed the "Geo-Routing" add-on which enabled us to use "Proximity steering". This works by providing (approximate) GPS coordinates for each origin server, letting CloudFlare choose the best origin for each request based on the user's IP (location).

- Basic Load Balancing (2 origins) - $5/mo
- Load Balancing Geo-Routing - $10/mo
- Additional origins cost $5/mo. $0.5 per 500k DNS requests

Overall the pricing feels very reasonable.

#### The importance of Host Headers

Our CloudFlare load balancer uses our primary domain (`awardfares.com`), and we added each origin server using the `onrender.com` subdomains that Render assigns to our app instances. However, this didn't work at first, since the incoming host header (`awardfares.com`) when someone visits our site didn't match what Render was expecting.

Our first attempt to solve this was to setup our primary domain as a custom domain on Render, until we eventually realized this wouldn't work either, since the same custom domain can't be used on several app instances at the same time (duh).

Eventually we found the "Host Header" setting in CloudFlare, allowing us to specify the `onrender.com` host header that should be used when CloudFlare forwards requests to our origin, solving that issue.

![](/assets/img/zero-devops-multi-region/host-header.png)

#### How not to lose your resource caching rules

Another aspect to take into consideration is caching. Render uses CloudFlare by default, and there is no way to turn this off. Furthermore, one quirk with Render is there is no support for CDN, meaning none of our static assets are served by CloudFlare's CDN.

Luckily this is fairly easy to workaround if you have Render behind your own CDN. Since we already use CloudFlare for DNS and load balancing, we were able to create our own Cache Rules in our CloudFlare account, forcing our static assets to be served by the CDN

![](/assets/img/zero-devops-multi-region/cache-rules.png)

## Conclusion

So that was basically it. Three fully managed components, dealing with our app, database, and load balancer. This was my first time digging into multi-region, so it was a fun and useful learning experience.

### Review of performance metrics

I was planning to include some performance analysis in this post as well, but I think I'll save that for another time. Needless to say, we did immediately see improved latency, loading times, and general app responsiveness in all regions.

### What about costs?

I think my main conclusion regarding costs is just how much time you can save by choosing the right tools for the right job, and how inexpensive most of those tools are! We are by no means a massive operation, but also not insignificant (serving tens of thousands of users).

Let's take a look at some numbers.

- App: Our app hosting costs went from $10/month on DigitalOcean for a single instance to $14/month for 2 multi-region instances on Render.

- Database: Our (oversized) mongo droplet on DigitalOcean went from $150/month to roughly $300/month on MongoDB Atlas. A pretty big cost jump, but going from a completely self-managed VPS to a fully managed service is well worth the additional cost.

- Load Balancer: Adding a CloudFlare load balancer increased our hosting costs from $0 to $15/month.

Each of our app instances on Render run on the Starter instance type ($7/month). There are some additional costs that we may need to take into consideration for Render, namely built minutes and bandwidth - but neither of those should be particularly big.
