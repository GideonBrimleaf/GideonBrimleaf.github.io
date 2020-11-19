---
layout: post
title: A super-light Google Maps Vue.js App
author: Gideon Brimleaf
postHero: /assets/images/Google-Maps-Logo.png
description: Google Maps integration with Vue.js in the smallest way possible
---

Adding Google Maps to your Vue.js app doesn't necessarily require plugins or even the Vue CLI.  Today we'll look at how to create a super-lightweight Vue.js app which uses the Google Maps API.

As per [Markus Oberlehner's easy to follow guide](https://markus.oberlehner.net/blog/goodbye-webpack-building-vue-applications-without-webpack/) you can create a simple Vue.js app using just the [Vue CDN](https://vuejs.org/v2/guide/installation.html#CDN) and the [BrowserSync npm package](https://www.npmjs.com/package/browser-sync) to serve up your app locally.  Stepping through the article should get you to something similar to this:

<pre class="p-2 bg-primary text-light">
├── package.json
└── src
    ├── components
    │   └── App.js
    ├── index.html
    ├── main.css
    └── main.js
</pre>

Without using Webpack or the Vue CLI we can get a small Vue app running with minimal fuss.  The [Google Maps API documentation also pitches a simple way to get started](https://developers.google.com/maps/documentation/javascript/overview#maps_map_simple-html) which involves adding a script tag to your <span class="code-snippet">index.html</span> file which will call the Google Maps API and a javascript file which will contain the function which will get run to initialise the map once the API call has been successful.  Once you've gotten an API key from the [Google Maps platform](https://developers.google.com/maps/gmp-get-started) you can add modify your app like so:

<pre class="p-2 bg-primary text-light">
├── package.json
└── src
    ├── components
    │   └── App.js
    ├── index.html
    ├── main.css
    ├── maps.js #new!
    └── main.js
</pre>

<span class="font-weight-bold">*./src/maps.js*</span>

<pre class="p-2 bg-primary text-light">
let map;

function initMap() {
  map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 51.513329, lng: -0.088950 },
    zoom: 8,
  });
}
</pre>

<span class="font-weight-bold">*./src/index.html*</span>

<pre class="p-2 bg-primary text-light">
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
  &lt;script src=&quot;https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.js&quot;&gt;&lt;/script&gt;
  &lt;script type=&quot;module&quot; src=&quot;./main.js&quot;&gt;&lt;/script&gt;
  &lt;link rel=&quot;stylesheet&quot; href=&quot;./main.css&quot;&gt;
  &lt;script src=&quot;./map.js&quot;&gt;&lt;/script&gt; &lt;!-- New --&gt;
  &lt;script src=&quot;https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;callback=initMap&quot; defer&gt;&lt;/script&gt; &lt;!-- New --&gt;
  &lt;title&gt;My Amazing Map&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;section id=&quot;app&quot;&gt;&lt;/section&gt;
  &lt;section id=&quot;map&quot;&gt;&lt;/section&gt; &lt;!-- New --&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre>

Make sure that any HTML elements wrapping the map section have a set height - otherwise it might not render.  In this case, the section element with the map id, the body and the html tags all need some height:

<span class="font-weight-bold">*./src/main.css*</span>

<pre class="p-2 bg-primary text-light">
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
</pre>

If all goes well you should see something like the following:

<pre class="shadowy">
<img src="/assets/images/working-vue-gmaps.png" class="img-fluid" alt="simple html dropdown">
</pre>

That's all well and good but we really want to incorporate our newly minted Google Map into our Vue app - which hooks into the section element with the app id.  This can be so we can leverage all the functionality that Vue can give us, such as a nice juicy data layer.

The first thing to do is to remove the map section element from our <span class="code-snippet">index.html</span> file as we want to render the map inside our Vue component instead. 

We might then think that we can delete the <span class="code-snippet">maps.js</span> file and move our <span class="code-snippet">initMap</span> function to our <span class="code-snippet">App.js</span> file and trigger it when the component loads like so:

<span class="font-weight-bold">*./src/App.js*</span>

<pre class="p-2 bg-primary text-light">
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
    &lt;section id="map"&gt;
    &lt;/section&gt;
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
</pre>

This would be great if it wasn't for the following problem:

<pre class="shadowy">
<img src="/assets/images/vue-gmaps-error.png" class="img-fluid" alt="simple html dropdown">
</pre>

Looks like our Vue component isn't talking to our Google Maps API script properly!  We need our API call to happen in the same context where we define our map creation (i.e. inside the Vue component).  The complicating factor here is that the API call runs in a script tag in our HTML which references a globally accessible function (<span class="code-snippet">initMap</span>) to create our function.

This means we need a couple of things to happen when the Vue component loads:

1. Register our <span class="code-snippet">initMap</span> function to the global scope - so that a Google Maps API url can access it
2. Inject the script tag which calls the API

<span class="font-weight-bold">*./src/App.js*</span>

<pre class="p-2 bg-primary text-light">
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
      const apiKey = 'YOUR API KEY HERE';
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
    &lt;section id="map"&gt;
    &lt;/section&gt;
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
</pre>

So now the initMap function is added to the global window scope once the Vue component loads, we then inject the script tag into our <span class="code-snippet">index.html</span> which can then refer to it as well as your API Key. Once that is in you should see a nice map being rendered!

<pre class="shadowy">
<img src="/assets/images/working-vue-gmaps.png" class="img-fluid" alt="simple html dropdown">
</pre>

You can find a sample codebase for this app [here](https://github.com/GideonBrimleaf/vue-one-pager/tree/super-simple-maps-app)