# Tracking Form Analytics With Redux and Google Analytics

In this tutorial we're going to collect analytics on a Redux-powered user
form. You will learn:
 - How to set up a destination funnel report in Google Analytics.
 - How to map Redux actions to Google Analytics page views.

The tutorial assumes some prior exposure to Git, JavaScript (ES2015), and Redux.

### The App

We'll be collecting analytics on the payment form of a simple ecommerce app. To
download the app, open up a terminal and run the following command:

```
git clone git@github.com:rangle/analytics-sample-apps.git
```

**Be Sure to Add in a tag**
```
git checkout tags/whatever
```

Once cloned, navigate to the `shopping-cart` directory and install the project's
dependencies:

```
cd analytics-sample-apps/shopping-cart
```
Then run:
```
npm install
```
<p align="center">
 <img src="http://localhost:6419/one-eternity-later.jpg" width="300">
</p>

Once npm has finished installing the app's dependencies, start the app with the
following command:

```
npm start
```

The app should open up in your browser at the following address:
`http://localhost:3000/`. Take a minute or two to play around and explore the
app:
  1. `/` shows a list of items to buy
  2. `/cart` shows items added to the cart
  3. `/payment` shows a form for collecting payment details
  4. `/order-complete` shows a message indicating a successful order

Our goal is to collect analytics on the payment form. Navigate to the `/payment`
view and open up your browser's JavaScript console.

Refresh the page, type a character into each input field, then click the
disabled `Buy Now` button. Your form and console should look something like
this now:

<p align="center">
 <img src="http://localhost:6419/payment-form-redux-actions.png">
</p>

When you land on the page Redux fires a `ROUTE_CHANGED` action, then an action
for each form field change, and finally an action when a user attempts to
buy something but fails to proceed because of invalid form inputs.

Update the form with valid inputs this time. None of the form fields should have
a red outline, and the `Buy Now` button should be enabled. Once finished, click
the `Buy Now` button. Notice how a Redux action fired whenever an input field
changed, and notice how there is one last `ROUTE_CHANGED` action when you
succesfully filled the form, clicked the `Buy Now` button, and moved to the
`/order-complete` page.

> **Checkstop.**
> Here's all you need to remember at this point:
> * Our goal is to collect analytics on the payment form
> * A Redux action fires when the user sees the payment form
> * A Redux action fires whenever one of the form fields change
> * A Redux action fires when a user tries to submit invalid details
> * A Redux action fires when a user sucessfully moves on to the /order-complete page

### Setting Up Google Analytics

Now that we've had a look at the form, let's see how we can set up a report in
Google Analytics to show the percentage of users that saw the form, filled it
in, and successfully completed an order.

Sign up for Google Analytics if you haven't already, and
[create a new web property](https://support.google.com/analytics/answer/1008015?hl=en).
Make a note of your property's [tracking Id](https://support.google.com/analytics/answer/1008080).

Follow the instructions
[here](https://support.google.com/analytics/answer/1032415?hl=en)
to create a new goal.
  1. In Goal Setup, select `Custom`.
  2. In Goal Description, enter in `Order Made` for the goal name.
  3. In Goal Description, select `Destination` for the goal type.
  4. Lastly, fill in Goal Details to match the following image then click `Save`.

<p align="center">
 <img src="http://localhost:6419/funnel-setup-ga.png" width="700">
</p>

Let's review. Our goal is to reach a _destination_, which is the
`/order-complete` page. We set up a _funnel_ report that shows the six steps we
expect the user to take before reaching the `/order-complete` page. We first
expect the user to land on the `/payment` page. Then we expect the user to fill
in each input field. We also expect that some users might mistakenly put in
invalid information and attempt to buy something anyway.

Notice anything strange? Funnel reports in Google Analytics expect that each
step a user takes towards a goal is a whole new page. This is a remanent from
the old days when single page apps weren't really a thing. Back then, whenever
anything major changed in a website, there was a page load. Now, in modern apps
like the one we're working on, we're using JavaScript to dynamically display
different views, update the address bar, and manage the browser history. Our
user might travel across different pages and fill in forms, but from Google
Analytics's perspective nothing is happening. We need a way to _fake_ a page
load when these things happen. Or more specifically, we need a way to map our
app's Redux actions to Google Analytics page views.

Based on the funnel report we set up, here's a map of Redux actions to the page
loads we need to fake:


| Redux Action Type          |   Virtual Page Load      |
| -----------------          |   -----------------      |
| ROUTE_CHANGED              |   /payment               |
| NAME_ENTERED               |   /name-entered          |
| EMAIL_ENTERED              |   /email-entered         |
| PHONE_NUMBER_ENTERED       |   /phone-number-entered  |
| CREDIT_CARD_NUMBER_ENTERED |   /cc-number-entered     |
| BUY_NOW_ATTEMPTED          |   /buy-now-attempted     |
| ROUTE_CHANGED              |   /order-complete        |


Thankfully, there's an npm package to help us with this exact problem.

### Redux Beacon

<p align="center">
 <img src="http://localhost:6419/superhero-redux-beacon.png">
</p>

[Redux Beacon](https://www.npmjs.com/package/redux-beacon) is a framework
agnostic library for mapping Redux actions to analytics events. In this next
part, we'll look at how we can leverage its API to solve our form analytics
problem.

Let's start off by installing the library and saving it to our project's
`package.json` file. Open a terminal, `cd` into the `shopping-cart` directory
and run the following command:

```
npm install redux-beacon@0.2.x --save
```

Once that's done, follow the instructions
[here](https://support.google.com/analytics/answer/1008080?hl=en) to
add your Google Analytics tracking snippet to the
`shopping-cart/public/index.html` file. This should be the same Google
Analytics property we set up in the previous section. If the app is
still running then saving the file should trigger a site
rebuild. Otherwise, call `npm start` to get the site up and running
again.

At this point, you should see one active user in your Google
Analytics [Real-Time](https://support.google.com/analytics/answer/1638635?hl=en)
dashboard.

Next, create a new file called `analytics.js` in `shopping-cart/src` and add the
following code to it:

```js
// shopping-cart/src/analytics.js

import { createMiddleware } from 'redux-beacon';
import { GoogleAnalytics } from 'redux-beacon/targets/google-analytics';
import { logger } from 'redux-beacon/extensions/logger';

const pageview = {
  eventFields: action => ({
    hitType: 'pageview',
    page: action.payload,
  }),
};

const eventsMap = {
  ROUTE_CHANGED: pageview
};

export const middleware = createMiddleware(eventsMap, GoogleAnalytics, { logger });
```

#### &#x25BC; Deep Dive

Let's go through this block by block. Feel free to skip this section if the API
seems obvious to you.

```js
import { createMiddleware } from 'redux-beacon';
import { GoogleAnalytics } from 'redux-beacon/targets/google-analytics';
import { logger } from 'redux-beacon/extensions/logger';
```

Here, we're importing various functions provided by Redux Beacon. The second and
third imports are relative imports. Redux Beacon was built this way to help
minimize bundle sizes.

```js
const pageview = {
  eventFields: action => ({
    hitType: 'pageview',
    page: action.payload,
  }),
};
```

This block is called an _event definition_ in Redux Beacon lingo. It
is a plain old JavaScript object (POJO) with a special `eventFields`
method. The object returned by the `eventFields` method is the
analytics event that Redux Beacon will push to a target.

```js
const eventsMap = {
  ROUTE_CHANGED: pageview
};
```

This block is called an _event definitions map_ in Redux Beacon lingo. This is
where we link Redux actions to event definitions. In this case when the
`ROUTE_CHANGED` action fires, Redux Beacon will call the `pageview`'s
`eventFields` method, pass it the action object, then push the resulting event
object to Google Analytics. Let's have a look at an example:

 1. The app dispatches the `{ type: 'ROUTE_CHANGED', payload: '/cart' }` action
 2. Redux Beacon calls `pageview.eventFields` with the action
 3. The `eventFields` method returns `{ hitType: 'pageview' page: '/cart' }`
 4. Redux Beacon hits Google Analytics with a page view (`/cart`)

```js
export const middleware = createMiddleware(eventsMap, GoogleAnalytics, { logger });
```

This last line creates the Redux Beacon middleware that we're going to apply to
the app's Redux store. We're first passing in the _event definitions map_,
followed by the _target_ for the resulting events, and finishing off with a
logging extension so we can see the anaylitcs events that Redux Beacon
generates.

#### &#x25B2; Deep Dive

Now that we have Redux Beacon set up all we have to do is apply the
middleware when creating the Redux store. Add the following line to
the top of the `src/App.js` file.

```js
// src/App.js (somewhere near the top of the file)
import { middleware as analyticsMiddleware } from './analytics';
```

Then, scroll down to were `createStore` is being called, and apply the
middleware.

```js
// src/App.js (should be around line 35)
const store = createStore(
  reducer,
  applyMiddleware(createLogger(), analyticsMiddleware)
);
```

nSave the file, then mozy over to your browser and refresh the shopping
cart app. Navigate from the root page to the cart page then to the
payment page. Have a look at the console. Like before, you should see
Redux actions for each route change, but now, you should also be
seeing the associated analytics events above each Redux action.

Go back to the Real-Time view in Google Analytics, this time, look at
the `Behaviour` view, and click on `Pageviews (last 30min)`. You
should see `/`, `/cart`, and `/payment` listed along with the number
of times each route was visited.

<p align="center">
 <img src="http://localhost:6419/tada.png">
</p>

With one simple event definition we have managed to map every
`ROUTE_CHANGED` action to a Google Analytics page hit. This includes
the two page hits we need to record for our funnel report (`/payment`,
and `/order-complete`).

| Redux Action Type          |   Virtual Page Load      |
| -----------------          |   -----------------      |
| ~~ROUTE_CHANGED~~          |   ~~/payment~~           |
| NAME_ENTERED               |   /name-entered          |
| EMAIL_ENTERED              |   /email-entered         |
| PHONE_NUMBER_ENTERED       |   /phone-number-entered  |
| CREDIT_CARD_NUMBER_ENTERED |   /cc-number-entered     |
| BUY_NOW_ATTEMPTED          |   /buy-now-attempted     |
| ~~ROUTE_CHANGED~~          |   ~~/order-complete~~    |

To track the remaining page views needed for the funnel report we need
to create an event definition for the each form field Redux actions,
and an event definition for when users try to submit the form with
invalid inputs.

In `src/analytics.js` add the following function below the `pageview`
event definition.

```js
function createPageview(route) {
  return {
    eventFields: () => ({
      hitType: 'pageview',
      page: route,
    }),
  };
}
```

This is a factory that when given a route returns an event definition
for an associated Google Analytics page hit.

Next, update the `eventsMap` as follows.

```js
const eventsMap = {
  ROUTE_CHANGED: pageview,
  NAME_ENTERED: createPageview('/name-entered'),
  EMAIL_ENTERED: createPageview('/email-entered'),
  PHONE_NUMBER_ENTERED: createPageview('/phone-number-entered'),
  CREDIT_CARD_NUMBER_ENTERED: createPageview('/cc-number-entered'),
};
```

Here we're using the `createPageview` factory to create an event
definition for each input field event.

Go back to your browser, navigate to the `/payment` page in the app
and fill in the form. You should now see analytics events being logged
to the console whenever the values of the input fields change. This is
what we wanted right? Yes, and no. We wanted to fake a page view
whenever a user entered something into a form field. But, with our
current set up, we are hitting Google Analytics with page views
whenever the user types something into a given form field. So if the
user enters in `John` in the name field, we are hitting Google
Analytics with four hits to `/name-entered`, when we only need to send
one.

Redux Beacon's event definitions have one other property that can help
us with this problem. Update the `createPageview` factory as follows.

```js
function createPageview(fieldName, route) {
  return {
    eventFields: (action, prevState) => ({
      hitType: 'pageview',
      page: prevState[fieldName].length === 0 ? route : '',
    }),
    eventSchema: {
      page: value => value === route,
    },
  };
}
```

explain eventSchema
And update the eventsMap as follows.

```js
const eventsMap = {
  ROUTE_CHANGED: pageview,
  NAME_ENTERED: createPageview('name', '/name-entered'),
  EMAIL_ENTERED: createPageview('email', '/email-entered'),
  PHONE_NUMBER_ENTERED: createPageview('phoneNumber', '/phone-number-entered'),
  CREDIT_CARD_NUMBER_ENTERED: createPageview('ccNumber', '/cc-number-entered'),
};
```

last finish off

```js
const eventsMap = {
  ROUTE_CHANGED: pageview,
  NAME_ENTERED: createPageview('name', '/name-entered'),
  EMAIL_ENTERED: createPageview('email', '/email-entered'),
  PHONE_NUMBER_ENTERED: createPageview('phoneNumber', '/phone-number-entered'),
  CREDIT_CARD_NUMBER_ENTERED: createPageview('ccNumber', '/cc-number-entered'),
  BUY_NOW_ATTEMPTED: {
    eventFields: () => ({ hitType: 'pageview', page: '/buy-now-attempt' })
  }
};
```
