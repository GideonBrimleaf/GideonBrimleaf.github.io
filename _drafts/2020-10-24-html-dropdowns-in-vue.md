---
layout: post
title: Default Dropdown Values using the Vue.js Data Layer
author: Gideon Brimleaf
postHero: /assets/images/MySQL-Logo.jpg
description: How to define a default value for a dropdown using the data layer in Vue.js
---

Vue is a powerful framework for giving users a dynamic, responsive experience in your web app.  The edge it has over React is its ability to neatly slot into server-side rendered templates in a lean fashion as well as offer the VueCLI for ambitious single-page apps. However, the way it interacts with core HTML elements does have its quirks to be mindful of.  When rendering a dropdown in HTML you can have a simple set of options with a default prompt to select one of them like so:

```html
<section>
  <h1>My Amazing Selector!</h1>
  <select>
    <option value="" selected="selected" disabled>Select a Thing:</option>
    <option value="Banana">Banana</option>
    <option value="Orange">Orange</option>
    <option value="Apple">Apple</option>
  </select>
</section>
```

The default option as the first item would be rendered first but you wouldn't be able to select it to continue:

[Show index.html view]

Let's say we wanted to dynamically generate the dropdown using Vue - the options would be held as an array in the data layer and then we loop over them rendering each option.  We want to hold what got selected as a new variable in our data layer to so we're going to ```v-model``` the select to generate the two-way binding to achieve this.

```js
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: null
    }
  },
  template: `
    <section>
      <h1>My Amazing Selector!</h1>
      <select v-model="selectedStuff">
        <option value="" selected="selected" disabled>Select a Thing:</option>
        <option v-for="thing in this.stuff">{{thing}}</option>
      </select>
    </section>
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
```

This will hook into our index.html where we have an element with the ```id="app"``` property to replace the hard-coded html selector from before and render out like so:

[Show index.html view]

Uh oh.  Looks like the default value is no longer showing - even though it shows when you click on the dropdown. This is because of the way our HTML is now interacting with Vue - because we have a ```v-model``` property on the select, HTML will be rendered for values that have an appropriately matching property in the data layer.  At the moment, our disabled option has an empty string as its value.  As this does not match the ```selectedStuff``` data that we are modelling (which is ```null```) it essentially breaks when it tries to render.  We need to get the disabled option to have a value that matches and there are two ways to achieve this:

1. Change the data point that is linked in the ```v-model``` to an empty string:

```js
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: '' //Modified
    }
  },
  template: `
    <section>
      <h1>My Amazing Selector!</h1>
      <select v-model="selectedStuff">
        <option value="" selected="selected" disabled>Select a Thing:</option>
        <option v-for="thing in this.stuff">{{thing}}</option>
      </select>
    </section>
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
```

This is the simplest fix as we already have our disabled default option set up with an empty string value.  Setting an empty string may not be as semantic as keeping it as ```null``` though in the data layer and may cause interesting bugs down the road.

2. Use v-bind to give the default option a ```null``` value.

```js
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: null
    }
  },
  template: `
    <section>
      <h1>My Amazing Selector!</h1>
      <select v-model="selectedStuff"> 
        <option v-bind:value="null" disabled>Select a Thing:</option> <!-- Modified -->
        <option v-for="thing in this.stuff">{{thing}}</option>
      </select>
    </section>
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
```

We can't just set the value in HTML to ```null``` as this would be a string value rather than the primitive value we're aiming for.  To get the latter we need to use ```v-bind``` to tell our HTML to use a Javascript value.  This means we can keep to a more semantic default value (i.e. nothing has been assigned yet to our ```selectedStuff``` data) but it might be a bit more tricky to wrap your head around what's going on.

With either solution you should see your nicely displayed dropdown like so:

[Show index.html view]