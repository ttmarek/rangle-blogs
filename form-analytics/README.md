----

* No gifs, make everything static
* Angular vs React, Redux vs ngrx/store
* Who is the source of the complication
* Make it a tutorial
* Nature? A manager? A BI member?

----

# Tracking Form Analytics With Redux and Google Analytics

In this tutorial we're going to collect analytics on a Redux-powered user
form. You will learn:
 - How to set up a destination funnel report in Google Analytics.
 - How to map Redux actions to Google Analytics page views.
 - How to track analytics in environments with intermittent internet access.

We'll be collecting analytics on the payment form of a simple ecommerce app. To
download the app, open up a terminal and run the following command:

```
git clone git@github.com:rangle/analytics-sample-apps.git
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
 <img src="http://localhost:6419/one-eternity-later.jpg" width="500">
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

Our goal is to collect analytics on the payment form. Let's take a closer look
at the `/payment` view, navigate there and open up your browser's JavaScript
console.

Refresh the page, type a character into each input field, then click once
on the disabled `Buy Now` button. Your form and console should look something
like this now:

<p align="center">
 <img src="http://localhost:6419/payment-form-redux-actions.png">
</p>

When you land on the page Redux fires a `ROUTE_CHANGED` action, then an action
for each change to a form field, and finally an action when a user attempts to
buy something but fails to proceed because of invalid form inputs.

Revist the form, except this time fill it in with valid inputs. Clicking the
`Buy Now` button should take you to the `/order-complete` page. Notice how a
Redux action fires whenever an input field updates, and notice how there is one
last `ROUTE_CHANGED` action when you succesfully fill the form and move to the
`/order-complete` page.

> **Checkstop.**
> Here's all you need to remember at this point:
> * Our goal is to collect analytics on the payment form
> * A Redux action fires when the user sees the payment form
> * A Redux action fires whenever one of the form fields change
> * A Redux action fires when a user tries to submit invalid details
> * A Redux action fires when a user sucessfully moves on to the /order-complete page

Now that we've had a look at the form, let's see how we can set up a report in
Google Analytics to show the percentage of users that saw the form, filled it
in, and successfully moved on to complete an order.

Sign up for Google Analytics if you haven't already, and
[create a new web property](https://support.google.com/analytics/answer/1008015?hl=en).
Make a note of your property's [tracking Id](https://support.google.com/analytics/answer/1008080).

Follow the instructions
[here](https://support.google.com/analytics/answer/1032415?hl=en)
to create a new _custom_ goal.

* Enter in `Order Made` for the goal name.
* Select `Destination` as the goal type.


<p align="center">
 <img src="http://localhost:6419/funnel-setup-ga.png">
</p>


<p align="center">
 <img src="http://localhost:6419/superhero-redux-beacon.png">
</p>

```
npm install --save redux-beacon
```

* Get a Google Analytics account if you don't have one
* Create a new property
* Create a new goal
* Create a funnel for the goal
* Destination
* Add the routes to the funnel report
* Explain the reasoning behind all the routes
* Complication: Need to mape the Redux actions to GA page views

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

* offline events explain the situation that it would be handy
* require the redux beacon extensions offline web
* set up the extension
* open the app, then go offline, fill out the form, then go back online
* Ta Da! analytics tracking when offline

* One approach to use Redux Beacon to track form analytics
