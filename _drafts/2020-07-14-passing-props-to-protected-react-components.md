---
layout: post
title: Passing Props to Protected React Components
author: Gideon Brimleaf
postHero: /assets/images/react-router.png
description: Using props with react components in react router that are protected
---

The React Router is a powerful tool for creating targeted React components for specific routes, mimicking traditional server-rendered views but giving you all the power of dynamic DOM manipulation. We'll be focusing today on how to pass props to components that require authentication to be rendered.

## App Preparation

Building a React app with <span class="code-snippet">[create-react-app](https://create-react-app.dev/)</span>, [React Router](https://reactrouter.com/web/guides/quick-start) and authentication is extensively covered and so, if you haven't already, I'd recommend checking tutorials such as those by [Maxim Ivanov](https://maksimivanov.com/posts/firebase-react-tutorial/), or [Binh Tran](https://medium.com/@thanhbinh.tran93/private-route-public-route-and-restricted-route-with-react-router-d50b27c15f5e#:~:text=full%20source%20code-,Private%20Route,function%20separately%20in%20utils%20folder.) to get you up and running.  [Google Firebase](https://firebase.google.com/) is a great platform for building database-backed React apps quickly and gives you authentication out of the box.  By that point your app should hopefully look something with a similar structure to the following (you can find the actual code [here](https://github.com/GideonBrimleaf/react-basic-auth-template)):

<pre class="p-2 bg-primary text-light">
├── README.md
├── .env --NEW!
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
    ├── components --NEW!
    |   ├── Auth.js
    │   ├── Firebase.js
    │   ├── Home.js
    │   ├── Login.js
    │   ├── PrivateRoute.js
    |   ├── SignUp.js
    │   └── UserProfile.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    ├── serviceWorker.js
    └── setupTests.js
</pre>

The important components are:

* <span class="code-snippet">Auth.js</span> - this component, takes advantage of ReactContext to share if the current user is authenticated or not to all components in the app
* <span class="code-snippet">Firebase.js</span> - we will be using Google Firebase to give us some Authentication out of the box
* <span class="code-snippet">Home.js</span> - the home component that will require a user to sign up and/or login to view
* <span class="code-snippet">Login.js</span> - this will allow the user to login
* <span class="code-snippet">PrivateRoute.js</span> - the wrapping component that will render components only when a check has been made to ensure the user has logged in, otherwise it will redirect the user to login
* <span class="code-snippet">SignUp.js</span> - this will allow the user to sign up to view private content
* <span class="code-snippet">UserProfile.js</span> - another component which only a logged-in user can see

Once you have these components in place you can modify <span class="code-snippet">App.js</span> to use React Router to have all our components designated to specific URLs. This would look something like the following:

<span class="font-weight-bold">*src/App.Js*</span>

<pre class="p-2 bg-primary text-light">
import React from 'react';
import './App.css';
import {BrowserRouter as Router, Route, Switch} from 'react-router-dom'
import { AuthProvider } from './components/Auth'
import Login from './components/Login'
import Home from './components/Home'
import SignUp from './components/SignUp'
import UserProfile from './components/UserProfile'

function App() {
  return (
    &lt;div className=&quot;App&quot;&gt;
      &lt;AuthProvider&gt;
          &lt;Router&gt;
            &lt;Switch&gt;
            &lt;Route exact path=&quot;/&quot; render={() =&gt; &lt;Home message=&quot;Hello from the home page!&quot; /&gt;} /&gt;
            &lt;Route path=&quot;/user/:userId&quot; render={(matchProps) =&gt; &lt;UserProfile {...matchProps} profileText=&quot;Welcome to your profile&quot; /&gt;} /&gt;
            &lt;Route exact path=&quot;/login&quot; component={Login} /&gt; 
            &lt;Route exact path=&quot;/signup&quot; component={SignUp} /&gt; 
            &lt;/Switch&gt; 
          &lt;/Router&gt;
        &lt;/AuthProvider&gt;
    &lt;/div&gt;
  );
}

export default App;
</pre>

There's a couple of things going on here:

1. All of our components are on public routes at the moment - everyone can see the Home and UserProfile components
2. Our <span class="code-snippet">Home</span> and <span class="code-snippet">UserProfile</span> components have props that need to be passed out to them.  In order to get a component with props working with React Router, you need to user the render argument, passing in a callback with the component and its props to be returned. 

## Setting up Authentication

We want these components behind our authentication wall, which is being provided by Firebase and being shared across all our components by the AuthProvider component.  If you've been following other tutorials you'll probably start by just substituting in the <span class="code-snippet">PrivateRoute</span> component for the Route component like so:

<span class="font-weight-bold">*src/App.Js*</span>

<pre class="p-2 bg-primary text-light">
import ... [As before]
import PrivateRoute from './components/PrivateRoute' 

function App() {
  return (
    &lt;div className=&quot;App&quot;&gt;
      &lt;AuthProvider&gt;
          &lt;Router&gt;
            &lt;Switch&gt;
            &lt;PrivateRoute exact path=&quot;/&quot; render={() =&gt; &lt;Home message=&quot;Hello from the home page!&quot; /&gt;} /&gt; { /*NEW!*/ }
            &lt;PrivateRoute path=&quot;/user/:userId&quot; render={(matchProps) =&gt; &lt;UserProfile {...matchProps} profileText=&quot;Welcome to your profile&quot; /&gt;} /&gt; { /*NEW!*/ }
            &lt;Route exact path=&quot;/login&quot; component={Login} /&gt; 
            &lt;Route exact path=&quot;/signup&quot; component={SignUp} /&gt; 
            &lt;/Switch&gt; 
          &lt;/Router&gt;
        &lt;/AuthProvider&gt;
    &lt;/div&gt;
  );
}

export default App;
</pre>

However this isn't quite enough, you'll get some nasty errors as React doesn't know how to handle the <span class="code-snippet">render</span> argument (where it was fine with it in the <span class="code-snippet">Route</span> component).  So we need to make some changes.  Let's start with our <span class="code-snippet">PrivateRoute</span> component:

<span class="font-weight-bold">*src/components/PrivateRoute.js*</span>

<pre class="p-2 bg-primary text-light">
import React, { useContext } from "react";
import { Route, Redirect } from "react-router-dom";
import { AuthContext } from "./Auth";

const PrivateRoute = ({ component: ComponentToRender, ...rest }) => {
  const {currentUser} = useContext(AuthContext);
  return (
    &lt;Route
      {...rest}
      render={routeProps =&gt;
        !!currentUser ? (
          &lt;ComponentToRender {...routeProps} /&gt;
        ) : (
          &lt;Redirect to={&quot;/login&quot;} /&gt;
        )
      }
    /&gt;
  );
};


export default PrivateRoute
</pre>

So this is a function, which takes in our component that we want to render, checks the context we made to see if the user is authenticated and returns a React Router <span class="code-snippet">Route</span> component with our component we want to render in.  What we need to do is add an additional argument to this function which represents the props we need to pass to our component, then we can make sure our component we want to render can access those props:

<span class="font-weight-bold">*src/components/PrivateRoute.js*</span>

<pre class="p-2 bg-primary text-light">
import React, { useContext } from "react";
import { Route, Redirect } from "react-router-dom";
import { AuthContext } from "./Auth";

const PrivateRoute = ({ component: ComponentToRender, componentProps, ...rest }) => {
  const {currentUser} = useContext(AuthContext);
  return (
    &lt;Route
      {...rest}
      render={routeProps =&gt;
        !!currentUser ? (
          &lt;ComponentToRender {...routeProps} {...componentProps} /&gt;
        ) : (
          &lt;Redirect to={&quot;/login&quot;} /&gt;
        )
      }
    /&gt;
  );
};


export default PrivateRoute
</pre>

Note how the <span class="code-snippet">componentProps</span> variable here is spread into a javascript object.  This is important as it determines how we pass in our components.  Rather than providing the props like a standard component, we need to pass them in as a javascript object.

<span class="font-weight-bold">*src/App.js*</span>

<pre class="p-2 bg-primary text-light">
import ... [As before]
import PrivateRoute from './components/PrivateRoute' 

function App() {
  return (
    &lt;div className=&quot;App&quot;&gt;
      &lt;AuthProvider&gt;
          &lt;Router&gt;
            &lt;Switch&gt;
            &lt;PrivateRoute exact path=&quot;/&quot; component={Home} componentProps=&#123;{ message:"Hello from the home page!" }&#125; /&gt; { /*NEW!*/ }
            &lt;PrivateRoute path=&quot;/user/:userId&quot; component={UserProfile} componentProps=&#123;{ profileText:"Welcome to your profile" }&#125; /&gt; { /*NEW!*/ }
            &lt;Route exact path=&quot;/login&quot; component={Login} /&gt; 
            &lt;Route exact path=&quot;/signup&quot; component={SignUp} /&gt; 
            &lt;/Switch&gt; 
          &lt;/Router&gt;
        &lt;/AuthProvider&gt;
    &lt;/div&gt;
  );
}

export default App;
</pre>


This setup is also very flexible, as components which use different props can be wrapped in the same <span class="code-snippet">PrivateRoute</span> component.  
