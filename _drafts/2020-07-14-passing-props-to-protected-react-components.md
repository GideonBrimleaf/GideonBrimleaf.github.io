---
layout: post
title: Passing Props to Protected React Components
author: Gideon Brimleaf
postHero: /assets/images/kotlin-intellij.png
description: Using props with react components in react router that are protected
---

The React Router is a powerful tool for creating targeted React components for specific routes, mimicking traditional server-rendered views but giving you all the power of dynamic DOM manipulation. There's a lot of good content on how to add authentication out there.  We'll be focusing today on how to pass props to components that require authentication to be rendered.

## App Preparation

Building a React app with create-react-app, React Router and authentication is extensively covered and so, if you haven't already, I'd recommend checking out either X, Y or Z to get you up and running.  By that point your app should hopefully look something with a similar structure to the following:

<pre class="p-2 bg-primary text-light">
├── README.md
├── nodule_modules
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── components --New!
    │   ├── Auth.js
    │   ├── Home.js
    │   ├── Login.js
    │   ├── PrivateRoute.js
    │   └── SignUp.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    ├── serviceWorker.js
    └── setupTests.js
</pre>

The important components are:

* Auth.js - this component, takes advantage of ReactContext to share if the current user is authenticated or not to all components in the app



