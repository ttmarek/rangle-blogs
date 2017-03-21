----

* No gifs, make everything static
* Angular vs React, Redux vs ngrx/store
* Who is the source of the complication
* Make it a tutorial
* Nature? A manager? A BI member?
* Google Analytics Funnel Report Back Filling

----

# Tracking Form Analytics With Redux and Google Analytics

In this tutorial we're going to collect analytics on a Redux-powered user
form. You will learn:
 - How to set up a destination funnel report in Google Analytics.
 - How to map Redux actions to Google Analytics page views.
 - How to track analytics in environments with intermittent internet access.

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
to create a new goal:
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
in each input field. We also expect that some users might mistakenly input
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

```
Redux Action Type              Virtual Page Load
-----------------              -----------------
ROUTE_CHANGED                  /payment
NAME_ENTERED                   /name-entered
EMAIL_ENTERED                  /email-entered
PHONE_NUMBER_ENTERED           /phone-number-entered
CREDIT_CARD_NUMBER_ENTERED     /cc-number-entered
BUY_NOW_ATTEMPTED              /buy-now-attempted
ROUTE_CHANGED                  /order-complete
```

### Redux Beacon

If we want to use Google Analytics's funnel reporting feature, we need a way to
map our app's Redux actions to Google Analytics page views. Like everything in
life, there's an npm package for that,
[Redux Beacon](https://www.npmjs.com/package/redux-beacon).

<p align="center">
 <img src="http://localhost:6419/superhero-redux-beacon.png">
</p>

Add the Google Analytics tracking Id to our project site.

Install the package by running the following command somewhere in your project directory.
```
npm install redux-beacon@0.2.x --save
```

Create a new file called `analytics.js` in `shopping/cart/src`

First map the ROUTE_CHANGED actions to page views:

```js

```

Then create the middleware, import the logger extension, import the GA target,
see the extensions firing.

Then map the NAME_ENTERED action to the page view

see the logs, notice how an


* introduce redux beacon
* set up redux beacon, follow instructions on the target docs page
* set up the redux beacon logger
* create an event definitions map for landing on the payment page
* see the console log
* create an event definitions map for the entering in a name
* see the console log
* notice that you're firing an event whenever the name field gets updated
* explain why this is a bad thing and how the funnel report will look
* create an eventSchema so only one event gets fired
* create event definitions for the rest of the form fields
* last is to create an event definition for buy now attempted
* You're done
* create a factory for the event definitions that are similar

### Offline Events

* offline events explain the situation that it would be handy
* require the redux beacon extensions offline web
* set up the extension
* open the app, then go offline, fill out the form, then go back online
* Ta Da! analytics tracking when offline

* One approach to use Redux Beacon to track form analytics
