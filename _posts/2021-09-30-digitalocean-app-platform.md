---
layout: post
title: "DigitalOcean App Platform + Auto-Scaling"
tags: [Tech, Cloud, DigitalOcean, Hosting]
---

I've been using DigitalOcean's new [App Platform](https://www.digitalocean.com/products/app-platform/) for several months now. As a long time user of DigitalOcean I was eager to try it out when it first launched as I'm always looking for ways to simplify/reduce dev-ops.

For me the main benefits with the App Platform are:
- Managed VPSs (no more droplet dev-ops)
- Auto-deployments out-of-the-box when pushing to GitHub
- Scaling capabilities
- Predictable and low cost (no hidden/variable fees)

## Overview

We migrated most of our project to App Platform over time, initially just some worker processes, but eventually our main web application as well. The project currently consists of 8 apps (split up into 7 workers and 1 web service) and has at least 15 instances running at once (sometimes more due to auto-scaling).

- 8 apps (7 workers, 1 web service)
- 15+ instances
- Auto-deploy from GitHub branch
- Auto-scaling using the DigitalOcean API

![Deployments](/assets/img/do-app-platform/apps.png)

Some pieces remain as regular droplets though. This includes our mongo database and our [self-hosted umami analytics](/2021/07/26/self-hosted-analytics-with-umami-and-traefik/). We've started looking into moving our MongoDB to a managed service (either DigitalOcean's new managed MongoDB offering or perhaps Mongo Atlas), but we're still evaluating our options.

Previously we also self-hosted our metrics/monitoring system, however, this has since moved to Grafana Cloud, which I covered in a [previous blog post](/2021/06/22/hosted-monitoring-evaluating-influx-and-grafana/).

In this post I'll highlight some of the main App Platform features, pros/cons, tips, and provide some of my feedback (wishes) to DigitalOcean.

### Sections:

- [Auto-deploy from GitHub](#auto-deploy-from-github)
- [App Spec Configuration](#app-spec-configuration)
- [Environment Variables and Secrets](#environment-variables-and-secrets)
- [Runtime Logs](#runtime-logs)
- [App Insights](#app-insights)
- [Console Access](#console-access)
- [Scaling](#scaling)
  - [Building an Auto-Scaler](#building-an-auto-scaler)
- [Access Control](#access-control)
- [Conclusion](#conclusion)

---

## Auto-deploy from GitHub

This is probably my favorite feature, as it makes our deployment workflow extremely streamlined. Our project uses a monorepo structure (one GitHub repository containing multiple sub-projects). Therefore I've setup one "deploy" branch per sub-project. I can deploy changes to one sub-project without triggering a re-deploy of all sub-projects.

![Deployments](/assets/img/do-app-platform/branch.png)

For the most part, auto-deploys have been working great. We did however run into some issues once where our app refused to deploy, but this seemed to be caused by an outage at DigitalOcean.

Initially we put all of our workers in the same app, as separate components. This was not ideal though. Accessing the logs was troublesome and scaling caused re-builds every time. Also, App Platform performed a clean build for every component even though they all shared the same code.

Tip: We eventually split up our different workers into separate apps with one component each, and things became a lot better.

### Feedback

One feature that would be nice to have would be to easily be able to revert to a previous deployment from the web UI. However, I guess this may cause problems if the deployment is out of sync with the GitHub branch.

---

## App Spec Configuration

Due to our monorepo structure, we ended up having multiple docker files, one for each sub-project, located in the root of the repository.

```
foo.Dockerfile
bar.Dockerfile
```

Unfortunately the App Platform doesn't understand this and instead thinks the project is just a plain node.js project. The setup wizard doesn't offer a way to explicitly choose a project type (i.e. docker instead of node.js), nor is there any way to choose a non-default dockerfile, which is kind of annoying.

Luckily there is a workaround though. Simply create the app using the default settings, go to the app settings, scroll down to the App Spec, and click Download.

![App Spec settings screenshot](/assets/img/do-app-platform/appspec.png)

You can then easily change the app to be a docker app and you can specify a different docker file using the `dockerfile_path` key. Then, simply upload the modified App Spec to apply the changes.

```yml
name: foo
region: fra
workers:
- dockerfile_path: foo.Dockerfile
  envs:
    ...
  github:
    branch: deploy-foo
    deploy_on_push: true
    repo: ...
  instance_count: 1
  instance_size_slug: basic-xs
  name: foo
  source_dir: /
```

Tip: You can also rename the component (if you don't like the default name that was generated).

### Feedback

It would be nice if the setup wizard offered a way customize this (or even to be able to create an app directly from an App Spec file). It would also be helpful if there was a UI setting for changing the `dockerfile_path`.

---

## Environment Variables and Secrets

As expected, App Platform supports environment variables and secrets. They can be defined both on an app-level (available to all components) and on an individual component level. However, there is unfortunately no way to define them on a project level.

![Environment Variables screenshot](/assets/img/do-app-platform/env.png)

It is possible to encrypt the variables and can thus not be read in plain text in the settings. However, I guess this is really only useful for preventing someone snooping on your screen. Anyone who has access to the project can easily dump all the variables by typing `printenv` in the Console (more about the Console soon)...

### Feedback

There doesn't seem to be any way to define project level variables. That would be nice to have, since most of my apps for a project share a lot of common variables.

---

## Runtime Logs

A useful feature is the built-in log viewer. It's basic, but very nice to have. It is limited to the last 3000 lines, and if you use scaling, you can only see logs from one instance. There is no way to see logs from other instances. There's definitely plenty of room for improvement here.

![Runtime logs screenshot](/assets/img/do-app-platform/logs.png)

For better logging you'd probably want to use a third party service. Since we already use [Grafana Cloud](/2021/06/22/hosted-monitoring-evaluating-influx-and-grafana/) for metrics, I recently started exploring their logging service, which offers 50GB/month free.

### Feedback

It would be great if there was a way to view logs for all instances, and to be able to filter/choose logs for a specific instance.

Also, it would be nice if there was a way to filter the log, rather than just search. For filtering I'd like only see lines that match my filter (and hide all the other lines). This makes it much easier to, for example, find and inspect all occurrences of an error.

---

## App Insights

Another useful feature is the insights section. The graphs are similar to the ones used for regular droplets, but with some differences. If you're using scaling, then you can see metrics for every instance, which is nice.

![Insights screenshot](/assets/img/do-app-platform/insights.png)

There's also a graph showing restarts, which I actually never noticed before (maybe it's new or I just never noticed it earlier). I guess it's useful way of keeping track of unexpected crashes (re-deploying does not count as a restart).

There are also several graphs related to CDN bandwidth, which I haven't really explored in detail yet.

![CDN insights screenshot](/assets/img/do-app-platform/cdn.png)

---

## Console Access

Another nice feature is console access. While I haven't really had a need for it yet, I imagine it might come in handy to be able to quickly access check on some data files, logs, caches, or maybe inspect error dumps. For docker apps, you can of course install any tools you'd like to have access to in the shell.

The only major downside is that, again, if you're using scaling, you only have access to one instance.

![Console screenshot](/assets/img/do-app-platform/console.png)

### Feedback

Offer a way to access the console of a specific instance (rather than just a random instance).

---

## Scaling

One of the main reasons for migrating to App Platform was for scaling, so we were really eager test what capabilities were available.

First of all it's good to know that apps are available in two different tiers: Basic and Pro. Basic apps start at $5 (512 MB, 1 vCPU) and Pro apps start at $12 (1GB, 1 vCPU).

Only Pro apps can be scaled horizontally. With a Pro app you can easily adjust the number of instances you'd like to deploy, which makes it super easy to scale up/down.

![Scaling settings](/assets/img/do-app-platform/scaling.png)

We did run into one issue initially though. Every time we adjusted the number instances our app would go through the entire re-building process. This meant that scaling our app took a good 5-10 minutes since it had to wait for the build process.

We did eventually workaround this issue when we decided to split up all of our different workers into separate apps (rather than all in one app as separate components). It seems that the app platform isn't really able to figure out how to re-use existing builds when you have multiple components.

### Building an Auto-Scaler

At the time of writing, DO does not offer any sort of auto-scaling feature. It does seem like this is one the roadmap though. My guess is that the first iteration of this would probably be based on resource utilization (e.g. CPU or memory).

In our particular case, each worker instance takes care of processing work. So, for us being able to perform auto-scaling based on a custom metric (processing backlog) is a must.

We were able to build a basic auto-scaler using the DigitalOcean API. It's actually very simple: fetch the current configuration, adjust the scaling factor, and update the app. We run the auto-scaler as a separate job in our "tasks" app that contains a collection of scheduled jobs.

```
1) Fetch current app sepc:

   GET https://api.digitalocean.com/v2/apps/${appId} --> App Spec

2) Get current instance count:

   currentInstanceCount = resp.app.spec.workers[0].instance_count

3) Calculate new instance count:

   resp.app.spec.workers[0].instance_count = computeNewInstanceCount(currentInstanceCount, backlog)

4) Apply new app spec

   App Spec --> PUT https://api.digitalocean.com/v2/apps/${appId}
```

Another feature we added was logging the instance count to Grafana, that way we can easily keep an eye on how our auto-scaler is performing.

![Auto-scaler metrics screenshot](/assets/img/do-app-platform/autoscaler.png)

We're still working on fine-tuning our scaling algorithm, so perhaps I'll write a more detailed post about that in the future.

### Feedback

It would be fantastic if App Platform would support auto-scaling with custom metrics. I guess a simple implementation of this for web services could be to query an endpoint defined by the user, that can respond with the desired number of instances, given the current number of instances:

```
/auto-scale?currentInstances=4 --> 5
```

If it even supported some options like being able to specify minimum time before scaling up/down, that would be fantastic!

---

## Access Control

At the moment there doesn't seem to be any support for using Firewalls with the App Platform. I.e. there is no way to setup IP whitelisting or other access control rules. It would also be nice if there was a way to restrict traffic to an internal VPC for apps and droplets.

---

## Conclusion

Overall, I'm very happy with the App Platform so far. There have been some small issues that we've had to deal with, but for the most part, everything has worked great. We'll definitely keep using it and hope to see more improvements in the near future!

Related posts:

- [Self-Hosted Analytics with Umami + Traefik](/2021/07/26/self-hosted-analytics-with-umami-and-traefik/)
- [Hosted Monitoring: Evaluating InfluxDB Cloud and Grafana Cloud](/2021/06/22/hosted-monitoring-evaluating-influx-and-grafana/)
