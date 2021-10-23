---
layout: post
title: "One Week Build: Paperworks.io"
tags: ["Nodejs", "JavaScript", "Startup", "Web Development", "DevOps"]
---

A one week build. Built with [node.js](https://nodejs.org/en/), deployed on [now](https://zeit.co/now).

![Paperworks](/assets/img/one-week-build-paperworks-io/1_FXbHMtUv7TePhM6tcrQXqQ.png)

All your receipts, invoices, and payments in one place. Sign up and connect one (or multiple) Gmail accounts. Paperworks automatically scans your inbox. Easily download attachments or save emails as PDFs. This story will highlight: why, how, and tools + services used for the build.


## Problem

I use many tools and services (both personal and for business). Every month I go through my bank statement and look for invoices/receipts that I need for bookkeeping.

1. Download bank statement for previous month

1. Go through 4 email accounts looking for paperwork

1. Download and organize attachments (when available)

1. Print > Save as PDF, when attachments aren’t available

1. Log into service and take screenshot, when no email receipt

I’ve been trying to figure out how I can optimize and automate this process. It’s an annoyance that I don’t want to deal with — and the problem grows with every new project that I start.


## Solution

Most of the paperwork I collect can be found directly in my inboxes. Let’s ignore the exceptions for now and start by only fetching from email. My goal was to build a service that provides:

* One place to collect receipts from all my inboxes

* Download attachments, with appropriate names

* Save emails to PDF in one click when no attachments

* Build and launch in one week


## Build

The primary goal was to build an MVP (minimum viable product) in one week. This goal helped me focus on the fundamental requirements and avoid spending too much time on specific features. The main build probably didn’t take more than about 2–3 days of development. The rest of the time was spent researching, testing, and deploying.

### Stack

* [Node.js](https://nodejs.org/en/) is one of my favorite platforms. It beats everything else when it comes to building web apps as quickly as possible — especially when considering the size of the community and packages available.

* [Express](http://expressjs.com/) is my go to framework for web apps. It’s what I have most experience with and it comes with Jade templates and Stylus stylesheets out of the box.

* [MongoDB](https://www.mongodb.com/) is my go to database for rapid prototyping, especially together with the [mongoose](https://www.npmjs.com/package/mongoose) object modelling library. There are a couple hosting options nowadays, including [compose.io](https://www.compose.com/) and [mLab](https://mlab.com/).

* [Docker](https://www.docker.com/) is a love-hate relationship for me. I struggle when dealing with larger deployments, persisted storage, etc. But for rapid prototyping it’s a great way to package your apps. Tip: [Dockering node.js apps](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

### Toolbox

Over the last year I have gathered a solid toolbox of things I use often. However, with every new project I find new ways of improving my workflow. Here are some noteworthy mentions:

* [Atom](https://atom.io/)— Cross-platform, clean, extensible code editor

* [nodemon](https://github.com/remy/nodemon) — Automatic process restarts during development

* [Tower](https://www.git-tower.com/) — Testing a new cross-platform Git Client

### Frontend

I’ve been doing a lot of work with react + redux lately. It’s great for building complex UI logic, but the boilerplate is quite intense. For this project I couldn’t justify going down that road. I went for a minimal server-side solution using Jade templates.

* [Jade](http://jadelang.net/) — Default view engine in Express (server-side)

* [Skeleton ](http://getskeleton.com/)— A very lightweight responsive CSS framework (with grids)

* [FontAwesome ](http://fontawesome.io/)— Pretty sure everyone already knows about this

### Authentication

User authentication is one of the things I dread the most when starting a new project. At first it seems pretty simple, but when you actually get into it you realize just how much there is to it (forms, secure passwords, reset logic with emails, etc).

The alternative approach I used for this project was a pure OAuth sign-in with Google solution. In this particular case it made a lot of sense, especially since the user has to connect their Gmail account(s) anyways to be able to use the service.

1. [Create a project in the Google API Console](https://console.developers.google.com)

1. [Use the built-in OAuth client in the googleapis package](https://www.npmjs.com/package/googleapis)


## Deployment + Hosting

I really just wanted something that **just works with minimal effort**. Sure, devops can be fun, but for a one week build I didn’t even want to think about servers, resources, orchestration, proxies, etc.

I recently discovered a really cool new service called [now](https://zeit.co/now), by a company called [Zeit](https://zeit.co/). You might recognize some of their other work (socket.io, mongoose, hyperterm, etc).

```
now allows you to take your JavaScript ([Node.js](https://nodejs.org/)) or Docker powered websites, applications and services to the cloud with ease, speed and reliability.
```


It’s simple to use, and by simple, I mean **really **simple:

```
// create an account directly from the terminal
$ now --login

// deploy (node app or dockerfile)
$ now

// associate a custom domain
$ now alias <id> paperworks.io
```


So how does it work? Deploying with **now** always creates a new deployment and returns a unique deployment id. Once you’ve deployed and made sure everything works, you can easily associate a custom-domain with the **alias** command.

They offer both a free and paid plan. The free plan makes your source code publicly available and doesn’t support custom domains. I subscribed to the paid plan and decided to give it a try.

```
1000 deploys per month
50GB monthly bandwidth
Dynamic realtime scaling
Custom domains
Private Source
SSL (Let's Encrypt)
$14.99 /month
```


One interesting thing with **now** is that you pay per deploy (push), not per deployment (instance). This means that you could (for the same price) deploy:

```
1000 different apps 1 time per month
1 app 1000 times per month
```


I’m not sure how suitable **now** is for production or business critical apps. I’ll be testing it for a month now and see how it goes. I read somewhere that they put inactive apps to sleep after a few hours. I haven’t really found any documentation regarding this though.

Here are some alternatives I’ve tested/worked with previously:

```
Heroku
- Price: $7/month per app
- No scaling
+ SSL
+ Custom Domains
+ Pretty easy to setup

Bluemix
- No SSL
- Clunky/slow UI
- Clunky/slow CLI (requires cf and bluemix)
+ Price: 512MB RAM free tier
+ Custom Domains

AWS
- Too much devops
- Price: Too complicated...
+ Custom Domains

DigitalOcean
- Too much devops
+ Price: $5/month
+ Custom Domains

GCS/GCP
- Too much devops
+ Price: Too complicated...
+ Custom Domains
```

**Conclusion: for a simple prototype most solutions are just way too complicated and/or expensive. I feel like [now](http://zeit.co/now) balances this perfectly.**

For database hosting I went for the 500 MB free tier @ [mLab.com](https://mlab.com/)


## Cost Breakdown

```
**Direct Costs - this project only
**Hover ........ $2.5 /month
mLab ......... $0 /month
CloudFlare ... $0 /month

Indirect Costs - reusable for other projects
GitHub ....... $7 /month
Now .......... $14.99 /month
```

## Time Breakdown

```
Day 1 - First Commit (1h)
Day 2 - Dev (12h)
Day 3 - Dev (12h)
Day 4 - Deploy Research(5h)
Day 5 - Deploy (2h)
Day 6 - Write-up (8h)
Day 7 - Publish
```

## Future Ideas

* “Download all” button
* Automatically synchronize to dropbox/gdrive
* Custom templates (recognized senders)
* Custom email filter
* Shareable link (for accountant access)
* Add support for IMAP accounts
* Extract details from emails (e.g. price/amount)

---

Read follow up story regarding the Product Hunt:
[https://medium.com/@pqvst/seriously-just-freaking-do-it-b934fa891e83](https://medium.com/@pqvst/seriously-just-freaking-do-it-b934fa891e83)