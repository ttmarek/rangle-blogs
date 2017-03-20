* fuck gifs, make everything static
* Angular vs React, Redux vs ngrx/store
* Who is the source of the complication
* Make it a tutorial
* Nature? A manager? A BI member?

# Tracking Form Analytics With Redux and Google Analytics

* a Redux powered form
* picture of the form
* produce a funnel report showing drop off at each field in the form
* funnel report in Google Analytics
* picture of the funnel report

* Download https://github.com/rangle/analytics-sample-apps
* npm install, and run app
* open up the console to see the redux actions
* Redux actions are firing whenever anyone types anything into the console

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

------------

#### Situation
Say you have a single page app for selling something, the app has four
views: a store view for displaying items, a cart view for displaying
items you want to buy, a payment view for collecting payment
information, and an order complete view to indicate a sucessful order.

```
/store
/cart
/payment
/order-complete
```

#### Complication
You've already integrated basic page-level analytics with Google Analytics into
your app, and you can see the number of visits to each view as a funnel
visualization:

```
Page             Views
/store           3568
/cart             378
/payment          265
/order-complete    11
```

Of the people that land on your store, 11% of them move to view their cart, of
those that viewed their cart, 70% move on to the payment form, and of those that
viewed the payment form only 4% moved on to complete their order. Your previous
experience tells you that a high drop off from the store view to the cart view
is normal, but the drop off at the payment view is uncharacteristically
high. You know that something on the payment view is giving your users grief,
but you don't know what.

#### Solution
Here's an example of what this payment view might look like:

You have a title, a simple form with four form fields, and at the bottom, a
button that remains disabled until all the form fields are filled and
valid. Knowing that your users are dropping off somewhere in the payment view,
one approach to see where is to create a funnel report in Google Analytics.

### Complication
Can only produce funnel reports with page loads/views

### Solution
Can use the analytics.js library to produce virtual page views

### Complication
Need to map Redux Action to virtual page view in analytics.js

### Solution
Use Redux Beacon

### Complication
Need to see events getting logged

### Solution
Use redux-beacon/extensions/logger

### Complication
Need to ensure analytics event only fires once, not every single
time the user types something into an input.

### Solution
eventSchema

### Complication
Everything is fine and dandy, but for whatever reason you need to track events
offline

### Solution
Use redux-beacon/extensions/offline-web

### Situation
Funnel report that tracks user drop off in payment form even when the users are
offline.
