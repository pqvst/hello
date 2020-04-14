---
layout: post
title: "One Week Build: Hyperbnb.com"
tags: ["JavaScript", "Nodejs", "Airbnb", "Web Development"]
---

A one week build. Supercharge your Airbnb searches. More listings, more filters, and advanced sorting to help you find your perfect spot. Built with [node.js](https://nodejs.org/en/) and [vue.js](https://vuejs.org/), deployed on [zeit now](https://zeit.co/now).

![](/assets/img/one-week-build-hyperbnb-com/1_mPaNaAN0S-eJVjIEXdThtQ.png)

---

## Problem

Airbnb is amazing and I can’t imagine not having it when I travel. I use Airbnb a lot, and I spend a lot of time filtering, reading, and searching for the perfect spot. The effort seems to pay off since I’ve never had a bad experience. But, I find that their interface is too limiting.

![Airbnb interface](/assets/img/one-week-build-hyperbnb-com/1-0_P60mwRq1qkC0HRlHkNg.png)*Airbnb interface*

**#1 — I want to be able to sort listings.** I totally understand why Airbnb limits filtering/sorting capabilities. If Airbnb always sorted by most reviews then no one would ever see any new listings.

**#2 — I want to see more than 18 listings at once.** In small towns this isn’t an issue. However, in larger cities with lots of listings you have to either zoom in really close, or set really strict filters. This makes it really difficult to get an overview of what’s available or to compare lots of places in a larger area.

**#3 —Maximum number of beds.** I find it interesting how you can filter for the “minimum” number of beds, but you can’t filter for “maximum” number of beds. If I’m traveling by myself I really don’t want to stay in a place that has 3 double beds crammed into a single room. Feels pointless. I’d like to set a max of 1 or 2 beds (some hosts consider a sofa to be a bed).

![Not ideal for 1 person…](/assets/img/one-week-build-hyperbnb-com/1QndwkPzRAd4DxKbpLw1dZw.png)*Not ideal for 1 person…*

**#4 — Minimum number of reviews.** I’m pretty picky when it comes to choosing a spot. I would never stay at a new listing, so it’s just a pain that I can’t remove them or set a minimum number of reviews to help narrow down the choices.

---

## Solution

The solution I went for was to try to create a new UI layer on top of the Airbnb listings data. To my surprise Airbnb actually has a pretty nice undocumented API. More about this later. Here are things I wanted to solve:

* Load up to 100 listings at once.

* Show listings in a table and map layout.

* Allow more filters

* Allow sorting

---

## Build

This was a very simple build actually. It probably helps that I’ve learned from my previous one week builds ([Paperworks](https://medium.com/front-end-hacking/one-week-build-paperworks-io-4d048f2886f8#.y7l993njk) and [BrowserFrame](https://medium.com/@pqvst/one-week-build-browserframe-com-7762e1276ccd#.6yga973xf)) and I now have a pretty good starter template.

### Backend

The stack and backend is very straight forward. I tried to put a lot of the logic in the frontend and keep the backend as simple as possible.

* **node.js & express** — quick to set up and get started

* **no database** — nothing to keep track of…

### Frontend

I’ve become very fond of vue.js, which I used for this project as well. It’s lightweight and super easy to get started with.

* [vue.js](https://vuejs.org/) — super easy to get started with (compared to react/angular)

* [bootstrap-daterangepicker](http://www.daterangepicker.com/) — a great date ranger picker that, despite the name, does not actually require bootstrap! Check out [this example](https://github.com/pqvst/bootstrap-daterangepicker-without-boostrap).

* [js-rich-marker](https://github.com/googlemaps/js-rich-marker) — after tons of searching I finally found this library which lets you create HTML markers for the Google Maps API. Super useful!

---

## Airbnb API

My initial concern was how to actually get hold of the Airbnb data. Turns out, after some Googling, Airbnb has an undocumented API. More than that, someone has already tried to document it: [http://airbnbapi.org/](http://airbnbapi.org/)

The documentation for parameters is definitely lacking. For example, I wanted to search with a specific check-in and check-out date. I also wanted to search by map coordinates. Even though the endpoints are different, I was able to figure out those parameters by inspecting what requests the official Airbnb site makes.

![](/assets/img/one-week-build-hyperbnb-com/1cQR0nCI-My8Nc04WGIcQZw.png)

The main downside right now is that the endpoint disallows CORS (cross-origin resource sharing) which means that the client (browser) isn’t able to make AJAX requests directly to the Airbnb API endpoint (since it’s coming from a different domain).

![](/assets/img/one-week-build-hyperbnb-com/1HYPjw7_6b4zoFZNjh2wZqw.png)

Instead, the browser first has send an AJAX request to my backend. Then my backend fetches the data from the Airbnb API. This adds some extra latency and definitely makes the UI less snappy (and the Airbnb API is already quite slow to begin with).

![](/assets/img/one-week-build-hyperbnb-com/1NUcCRv65y-iOc_GrkL7XJA.png)

---

## Future Ideas

* Improve UI responsiveness

* Add histograms for pricing/ratings/reviews etc.

* Add sliders instead of min/max text boxes.

* Add filters for amenities and features.

---

## Give it a try :)

[https://hyperbnb.com](https://hyperbnb.com)  
[https://www.producthunt.com/posts/hyperbnb](https://www.producthunt.com/posts/hyperbnb)