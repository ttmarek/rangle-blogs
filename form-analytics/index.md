----

* No gifs, make everything static
* Angular vs React, Redux vs ngrx/store
* Who is the source of the complication
* Make it a tutorial
* Nature? A manager? A BI member?

----

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
