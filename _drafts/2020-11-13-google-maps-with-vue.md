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

That's all well and good but we really want to incorporate our newly minted Google Map into our Vue app - which hooks into the section element with the app id.  This can be so we can leverage all the functionality that Vue can give us, such as a nice juicy data layer.

The first thing to do is to remove the map section element from our index.html file as we want to render the map inside our Vue component instead. 

We might then think that we can delete the `maps.js` file and move our initMap function to our `App.js` file and trigger it when the component loads like so:

<span class="font-weight-bold">*./src/App.js*</span>

```
const App = {
  name: 'App',
  data() {
    return {
      map: null
    }
  },
  mounted() {
      this.initMap()
  },
  methods: {
    initMap: function() {
    
      this.map = new google.maps.Map( document.getElementById("map"), {
        center: {
          lat: 51.513329,
          lng: -0.088950
        },
        zoom: 14
      });
    }
  },
  template: `
    <section id="map">
    </section>
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
```

This would be great if it wasn't for the following problem:

[Show screenshot of the undefined error]

Looks like our Vue component isn't talking to our Google Maps API script properly!  We need our API call to happen in the same context where we define our map creation (i.e. inside the Vue component).  The complicating factor here is that the API call happens as a script tag in our HTML which references a globally accessible function (initMap) to create our function.

This means we need a couple of things to happen when the Vue component loads:

1. Alter our initMap function so that the map is created inside the Vue component defined in the template.
2. Register our initMap function to the global scope - so that a Google Maps API url can access it
3. Inject the script tag which calls the API

<span class="font-weight-bold">*./src/App.js*</span>

```
const App = {
  name: 'App',
  data() {
    return {
      map: null
    }
  },
  mounted() {
      window.gmapsCallback = () => this.initMap() // NEW!
      this.gmapsInit() // NEW!
  },
  methods: {
    // NEW!
    gmapsInit: function() {
      const apiKey = '<YOUR API KEY HERE>';
      const callbackName = 'gmapsCallback';

      const script = document.createElement('script');
      script.async = true;
      script.defer = true;
      script.src = `https://maps.googleapis.com/maps/api/js?key=${apiKey}&callback=${callbackName}`;
      document.querySelector('head').appendChild(script);
    },
    initMap: function() {
    
      this.map = new google.maps.Map( this.$el, { // NEW!
        center: {
          lat: 51.513329,
          lng: -0.088950
        },
        zoom: 14
      });
    }
  },
  template: `
    <section id="map">
    </section>
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
```

So now the initMap function is added to the global window scope once the Vue component loads, we then inject the script tag into our `index.html` which can then refer to it as well as your API Key. Once that is in you should see a nice map being rendered!

[Show map]