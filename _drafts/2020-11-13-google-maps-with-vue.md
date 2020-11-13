---
layout: post
title: A super-light Google Maps Vue.js App
author: Gideon Brimleaf
postHero: /assets/images/vuejs-wide.png
description: Google Maps integration with Vue.js in the smallest way possible
---

Adding Google Maps to your Vue.js app doesn't necessarily require plugins or even the Vue CLI.  Today we'll look at how to create a super-lightweight Vue.js app which uses the Google Maps API.

As per [Markus Oberlehner's easy to follow guide](https://markus.oberlehner.net/blog/goodbye-webpack-building-vue-applications-without-webpack/) you can create a simple Vue.js app using just the [Vue CDN](https://vuejs.org/v2/guide/installation.html#CDN) and the BrowserSync npm package to serve up your app locally.  Stepping through the article should get you to something similar to this:

```
├── package.json
└── src
    ├── components
    │   └── App.js
    ├── index.html
    ├── main.css
    └── main.js
```

Without using Webpack or the Vue CLI we can get a small Vue app running with minimal fuss.  The [Google Maps API documentation also pitches a simple way to get started](https://developers.google.com/maps/documentation/javascript/overview#maps_map_simple-html) which involves adding a script tag to your index.html file which will call the Google Maps API and a javascript file which will contain the function which will get run to initialise the map once the API call has been successful.  Once you've gotten an API key from the [Google Maps platform](https://developers.google.com/maps/gmp-get-started) you can add modify your app like so:

```
├── package.json
└── src
    ├── components
    │   └── App.js
    ├── index.html
    ├── main.css
    ├── maps.js #new!
    └── main.js
```

<span class="font-weight-bold">*./src/maps.js*</span>

```
let map;

function initMap() {
  map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 51.513329, lng: -0.088950 },
    zoom: 8,
  });
}
```
<span class="font-weight-bold">*./src/index.html*</span>

```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.js"></script>
  <script type="module" src="./main.js"></script>
  <link rel="stylesheet" href="./main.css">
  <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap" defer></script>
  <title>My Amazing Selector</title>
</head>
<body>
  <section id="app"></section>
  <section id="map"></section>
</body>
</html>
```

Make sure that any HTML elements wrapping the map section have a set height - otherwise it might not render.  In this case, the section element with the map id, the body and the html tags all need some height:

<span class="font-weight-bold">*./src/main.css*</span>

```
html {
  height: 90%;
}

body {
  height: 90%;
  background-color: dodgerblue;
}

#map {
  height: 100%;
}
```

If all goes well you should see something like the following:

[Show the map screenshot]

That's all well and good but we really want to incorporate our newly minted Google Map into our Vue app - which hooks into the section element with the app id.  This can be so we can leverage all the juicy functionality that Vue can give us, such as a nice juicy data layer.

So we might think that we can move our initMap function to our `App.js` file.