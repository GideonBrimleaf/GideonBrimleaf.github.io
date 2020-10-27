---
layout: post
title: Default Dropdown Values using the Vue.js Data Layer
author: Gideon Brimleaf
postHero: /assets/images/vuejs-wide.png
description: How to define a default value for a dropdown using the data layer in Vue.js
---

Vue.js is a powerful framework for giving users a dynamic, responsive experience in your web app.  The edge it has over React is its ability to neatly slot into server-side rendered templates in a lean fashion as well as offer the [VueCLI](https://cli.vuejs.org/) for ambitious single-page apps. However, the way it interacts with core HTML elements does have its quirks to be mindful of.  When rendering a dropdown in HTML you can have a simple set of options with a default prompt to select one of them like so:

<pre class="p-2 bg-primary text-light">
&lt;section&gt;
  &lt;h1&gt;My Amazing Selector!&lt;/h1&gt;
  &lt;select&gt;
    &lt;option value=&quot;&quot; selected=&quot;selected&quot; disabled&gt;Select a Thing:&lt;/option&gt;
    &lt;option value=&quot;banana&quot;&gt;banana&lt;/option&gt;
    &lt;option value=&quot;orange&quot;&gt;orange&lt;/option&gt;
    &lt;option value=&quot;apple&quot;&gt;apple&lt;/option&gt;
  &lt;/select&gt;
&lt;/section&gt;
</pre>

The default option as the first item would be rendered first but you wouldn't be able to select it to continue:

<pre class="shadowy">
<img src="/assets/images/original-html-selector.png" class="img-fluid" alt="simple html dropdown">
</pre>

Let's say we wanted to dynamically generate the dropdown using Vue - the options would be held as an array in the data layer and then we loop over them rendering each option.  We want to hold what got selected as a new variable in our data layer to so we're going to <span class="code-snippet">v-model</span> the select to generate the two-way binding to achieve this.

<pre class="p-2 bg-primary text-light">
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: null
    }
  },
  template: `
    &lt;section&gt;
      &lt;h1&gt;My Amazing Selector!&lt;/h1&gt;
      &lt;select v-model=&quot;selectedStuff&quot;&gt;
        &lt;option value=&quot;&quot; selected=&quot;selected&quot; disabled&gt;Select a Thing:&lt;/option&gt;
        &lt;option v-for=&quot;thing in this.stuff&quot;&gt;{{thing}}&lt;/option&gt;
      &lt;/select&gt;
    &lt;/section&gt;
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
</pre>

This will hook into our index.html where we have an element with the <span class="code-snippet">id="app"</span> property to replace the hard-coded html selector from before and render out like so:

<pre class="shadowy">
<img src="/assets/images/broken-html-dropdown-with-vue.png" class="img-fluid" alt="html dropdown not showing default value">
</pre>

Uh oh.  Looks like the default value is no longer showing - even though it shows when you click on the dropdown. This is because of the way our HTML is now interacting with Vue - because we have a <span class="code-snippet">v-model</span> property on the select, HTML will be rendered for values that have an appropriately matching property in the data layer.  At the moment, our disabled option has an empty string as its value.  As this does not match the <span class="code-snippet">selectedStuff</span> data that we are modelling (which is <span class="code-snippet">null</span>) it essentially breaks when it tries to render.  We need to get the disabled option to have a value that matches and there are two ways to achieve this:

<strong>Option 1: Change the data point that is linked in the <span class="code-snippet">v-model</span> to an empty string:</strong>

<pre class="p-2 bg-primary text-light">
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: '' //Modified
    }
  },
  template: `
    &lt;section&gt;
      &lt;h1&gt;My Amazing Selector!&lt;/h1&gt;
      &lt;select v-model=&quot;selectedStuff&quot;&gt;
        &lt;option value=&quot;&quot; selected=&quot;selected&quot; disabled&gt;Select a Thing:&lt;/option&gt;
        &lt;option v-for=&quot;thing in this.stuff&quot;&gt;{{thing}}&lt;/option&gt;
      &lt;/select&gt;
    &lt;/section&gt;
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
</pre>

This is the simplest fix as we already have our disabled default option set up with an empty string value.  Setting an empty string may not be as semantic as keeping it as <span class="code-snippet">null</span> though in the data layer and may cause interesting bugs down the road.

<strong> Option 2: Use <span class="code-snippet">v-bind</span> to give the default option a <span class="code-snippet">null</span> value.</strong>

<pre class="p-2 bg-primary text-light">
const App = {
  name: 'App',
  data() {
    return {
      stuff: ["banana", "orange", "apple"],
      selectedStuff: null
    }
  },
  template: `
    &lt;section&gt;
      &lt;h1&gt;My Amazing Selector!&lt;/h1&gt;
      &lt;select v-model=&quot;selectedStuff&quot;&gt; 
        &lt;option v-bind:value=&quot;null&quot; disabled&gt;Select a Thing:&lt;/option&gt; &lt;!-- Modified --&gt;
        &lt;option v-for=&quot;thing in this.stuff&quot;&gt;{{thing}}&lt;/option&gt;
      &lt;/select&gt;
    &lt;/section&gt;
  `,
};

new Vue({
  render: h => h(App),
}).$mount(`#app`);
</pre>

We can't just set the value in HTML to <span class="code-snippet">null</span> as this would be a string value rather than the primitive value we're aiming for.  To get the latter we need to use <span class="code-snippet">v-bind</span> to tell our HTML to use a Javascript value.  This means we can keep to a more semantic default value (i.e. nothing has been assigned yet to our <span class="code-snippet">selectedStuff</span> data) but it might be a bit more tricky to wrap your head around what's going on.

With either solution you should see your nicely displayed dropdown like so:

<pre class="shadowy">
<img src="/assets/images/fixed-html-dropdown-with-vue.png" class="img-fluid" alt="html dropdown showing default value">
</pre>