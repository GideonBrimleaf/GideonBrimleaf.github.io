---
layout: post
title: Passing Props to Protected React Components
author: Gideon Brimleaf
postHero: /assets/images/kotlin-intellij.png
description: Using props with react components in react router that are protected
---

The React Router is a powerful tool for creating targeted React components for specific routes, mimicking traditional server-rendered views but giving you all the power of dynamic DOM manipulation. We'll be focusing today on how to pass props to components that require authentication to be rendered.

## App Preparation

Building a React app with [create-react-app](https://create-react-app.dev/), [React Router](https://reactrouter.com/web/guides/quick-start) and authentication is extensively covered and so, if you haven't already, I'd recommend checking tutorials such as those by [Maxim Ivanov](https://maksimivanov.com/posts/firebase-react-tutorial/), or [Binh Tran](https://medium.com/@thanhbinh.tran93/private-route-public-route-and-restricted-route-with-react-router-d50b27c15f5e#:~:text=full%20source%20code-,Private%20Route,function%20separately%20in%20utils%20folder.) to get you up and running.  [Google Firebase](https://firebase.google.com/) is a great platform for building database-backed React apps quickly and gives you authentication out of the box.  By that point your app should hopefully look something with a similar structure to the following (you can find the actual code [here](#)):

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

* Auth.js - this component, takes advantage of ReactContext to share if the current user is authenticated or not to all components in the app
* Firebase.js - we will be using Google Firebase to give us some Authentication out of the box
* Home.js - the home component that will require a user to sign up and/or login to view
* Login.js - this will allow the user to login
* PrivateRoute.js - the wrapping component that will render components only when a check has been made to ensure the user has logged in, otherwise it will redirect the user to login
* SignUp.js - this will allow the user to sign up to view private content
* UserProfile.js - another component which only a logged-in user can see

Once you have these components in place you can modify App.js to use React Router to have all our components designated to specific URLs. This would look something like the following:

src/App.Js

<pre class="p-2 bg-primary text-light">
import React from 'react';
import './App.css';
import {BrowserRouter as Router, Route, Switch} from 'react-router-dom'
import { AuthProvider } from './auth/Auth'
import Login from './components/Login'
import Home from './components/Home'
import SignUp from './components/SignUp'
import UserProfile from './components/UserProfile'

function App() {
  return (
    <div className="App">
      <AuthProvider>
          <Router>
            <Switch>
            <Route exact path="/" render={() => <Home message="Hello from the home page!" />} />
            <Route path="/user/:userId" render={(matchProps) => <UserProfile {...matchProps} profileText="Welcome to your profile" />} />
            <Route exact path="/login" component={Login} /> 
            <Route exact path="/signup" component={SignUp} /> 
            </Switch> 
          </Router>
        </AuthProvider>
    </div>
  );
}

export default App;
</pre>

There's a couple of things going on here:

1. All of our components are on public routes at the moment - everyone can see the Home and UserProfile components
2. Our Home and UserProfile components have props that need to be passed out to them.  In order to get a component with props working with React Router, you need to user the render argument, passing in a callback with the component and its props to be returned. 

We want these components behind our authentication wall, which is being provided by Firebase and being shared across all our components by the AuthProvider component.  If you've been following other tutorials you'll probably start by just substituting in the PrivateRoute component for the Route component like so:

src/App.Js

<pre class="p-2 bg-primary text-light">
import ... [As before]
import PrivateRoute from './components/PrivateRoute' 

function App() {
  return (
    <div className="App">
      <AuthProvider>
          <Router>
            <Switch>
            <PrivateRoute exact path="/" render={() => <Home message="Hello from the home page!" />} /> { /*NEW!*/ }
            <PrivateRoute path="/user/:userId" render={(matchProps) => <UserProfile {...matchProps} profileText="Welcome to your profile" />} /> { /*NEW!*/ }
            <Route exact path="/login" component={Login} /> 
            <Route exact path="/signup" component={SignUp} /> 
            </Switch> 
          </Router>
        </AuthProvider>
    </div>
  );
}

export default App;
</pre>

However this isn't quite enough, you'll get some nasty errors as React doesn't know how to handle the render argument (where it was fine with it in the Route component).  So we need to make some changes.  Let's start with our PrivateRoute component:

src/components/PrivateRoute.js

<pre class="p-2 bg-primary text-light">
import React, { useContext } from "react";
import { Route, Redirect } from "react-router-dom";
import { AuthContext } from "./Auth";

const PrivateRoute = ({ component: ComponentToRender, ...rest }) => {
  const {currentUser} = useContext(AuthContext);
  return (
    <Route
      {...rest}
      render={routeProps =>
        !!currentUser ? (
          <ComponentToRender {...routeProps} />
        ) : (
          <Redirect to={"/login"} />
        )
      }
    />
  );
};


export default PrivateRoute
</pre>

So this is a function, which takes in our component that we want to render, checks the context we made to see if the user is authenticated and returns a React Router Route component with our component we want to render in.  What we need to do is add an additional argument to this function which represents the props we need to pass to our component, then we can make sure our component we want to render can access those props:

src/components/PrivateRoute.js

<pre class="p-2 bg-primary text-light">
import React, { useContext } from "react";
import { Route, Redirect } from "react-router-dom";
import { AuthContext } from "./Auth";

const PrivateRoute = ({ component: ComponentToRender, componentProps, ...rest }) => {
  const {currentUser} = useContext(AuthContext);
  return (
    <Route
      {...rest}
      render={routeProps =>
        !!currentUser ? (
          <ComponentToRender {...routeProps} {...componentProps} />
        ) : (
          <Redirect to={"/login"} />
        )
      }
    />
  );
};


export default PrivateRoute
</pre>

Note how the componentProps variable here is spread into a javascript object.  This is important as it determines how we pass in our components.  Rather than providing the props like a standard component, we need to pass them in as a javascript object.

src/App.js

<pre class="p-2 bg-primary text-light">
import ... [As before]
import PrivateRoute from './components/PrivateRoute' 

function App() {
  return (
    <div className="App">
      <AuthProvider>
          <Router>
            <Switch>
            <PrivateRoute exact path="/" component={Home} componentProps={{ message:"Hello from the home page!" }} /> { /*NEW!*/ }
            <PrivateRoute path="/user/:userId" component={UserProfile} componentProps={{ profileText:"Welcome to your profile" }} /> { /*NEW!*/ }
            <Route exact path="/login" component={Login} /> 
            <Route exact path="/signup" component={SignUp} /> 
            </Switch> 
          </Router>
        </AuthProvider>
    </div>
  );
}

export default App;
</pre>


This setup is also very flexible, as components which use different props can be wrapped in the same PrivateRoute component.  
