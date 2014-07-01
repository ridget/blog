---
title: Getting Started With Ember.js
date: 2014-06-29 06:40 UTC
tags: js, ember.js
---

## Ember CLI
Ember is an opinionated JavaScript framework, it subscribes to the Rails philosophy of convention over configuration and makes it very easy to start building well structured,
rich, client side applications.

Ember CLI is a tool used to help build ember applications, and similarly has a strong opinion on how you should structure and work with your code. Today's blog post is going to be using both of these tools.

As a Rails developer, the urge to utilise the ember-rails gem is pretty high, however I'm of the opinion that you should keep your ember application separate from your Rails app and ember cli provides too good a tool to ignore.

Ember CLI is a command line tool thats based on the Ember App Kit project, it is meant to function as a replacement to Ember App Kit. For our purposes all we need to know now is that it gives us a great project structure out of the box (not unlike when you generate a new Rails project with Rails new), and a few handy generators.

To get started, please follow the instructions at [ember cli](http://iamstef.net/ember-cli/), and come back when you're ready to proceed.

We're going to be making a simple budgeting application, we'll take incoming transactions, assign them a name, description and a price, and assign them to a category.

To start simply run `ember new mybudget` this will create a new `mybudget` folder and generate the application structure. Already we can run `ember server` from within our application and navigate to our ember app. 

To help us identify what's going on in our application, we're going to navigate to our `app.js` file and add the following:

    var App = Ember.Application.extend({
      ...
      LOG_TRANSITIONS: true,
      LOG_TRANSITIONS_INTERNAL: true
    });

This will assist us in getting a better understanding of what's going on under the hood.

## The Router
Next we're going to start setting up some of our routes within `app/router.js`. The Router in an ember.js app, as opposed to individual route objects, is much like a router in rails. We specify which form our routes will take in our application. The main point of difference between a router in Rails and a router in Ember is that a route in ember represents any one of the possible states of your application, which in turn provides a url.

To start we're going to add our first real route, which will display an index of transactions:

    Router.map(function() {
      this.resource('transactions', { path: '/' });
    });

Note the addition of the `path` option, this tells ember to render the transactions template when visiting the root path of your application.

Now I'm going to segway a little into one of the nicer features of ember. 
When you give it a route like the above, it tells ember that it needs an `IndexRoute`,
a `TransactionsRoute` and a `TransactionsIndexRoute` as well as controllers and templates following the same naming convention eg `TransactionsIndexController` and  `posts/index`. 

What ember will do for you is generate those required routes, controllers and templates for you, so you can continue to develop your application until you need to override the default behaviour within those files.

## The Model

All this routing needs a model to back it. Once again there's some similarities with Rails here so go ahead and create a model in `app/models/transaction.js`:

    import DS from 'ember-data';

    var Transaction = DS.Model.extend({
      name: DS.attr('string'),
      amount: DS.attr('number'),
      occurredOn: DS.attr('date')
    });

    Transaction.reopenClass({
      FIXTURES: [
      {
        id: 1,
          name: "GYG Burritos",
          amount: "1200",
          occurredOn: "12/04/2014" 
      }
    ]});

    export default Transaction;


We've imported ember data, using the import statement at the top of the page, this is an ember-cli feature as it uses ES6 module functionality.
The scope of this is a bit beyond this post, but know that ES6 modules are not fully supported in a lot of browsers, so ember uses some crafty magic to make these available to use in the browser.

We've also defined our model, given it some attributes with different types and created an initial fixture for us to work with.
To utilise fixtures, open up `app/adapters/application.js` and add the following:


    import DS from 'ember-data';
    export default DS.FixtureAdapter.extend();

All this basically says to ember is that for the moment, utilise fixtures as our data source.

## A Route

Now we want to make sure when we visit the index of our application we see our list of transactions.
To accomplish this, we need to use a route. A route helps define what model should be available to your template when we view our transactions, this is achieved with the `model` hook in the route.

For our first route, we're going to create the `TransactionsRoute` in `app/routes/transactions.js`

    import Ember from 'ember';
    var TransactionsRoute = Ember.Route.extend({
      model: function(){
        return this.store.find('transaction');
      }
    });

    export default TransactionsRoute;

Note the import of Ember, this is another ember-cli requirement that whenever we need ember-data or Ember in our files, we import either of those depending on our needs.

We also return our available transactions in our model hook

## A template

We've setup our routes, our model and some initial data, now we need to display it. So let's setup a `transactions` template in `app/templates/transactions.hbs` 


    <ul>
      {{#each}}
        <li>{{name}} {{amount}} {{occurredOn}}</li>
      {{/each}}
    </ul>

The `each` here loops through an array of records, and although we only have one transaction to return will ensure our data is displayed nicely and will support additional records.

Now if we run `ember server` from the console we should see our fixture data displayed for us. 

I should take a step back here and explain that even though ember knows to go and find the transactions template, it's the `{{outlet}}` statement in `app/templates/application.hbs` which enables the template to bubble up to be displayed.

We'll be using this same functionality when we dive further into our transaction app functionality, but for now our list of transactions can remain in our transactions template.

We'll continue on from here in the next blog post, but for now if you'd like to take a look at the source code, you can find it on [github](http://github.com/ridget/transactions)

## A Note on Controllers

We didn't look too closely at ember controllers in this post, this will come up in the next post in the series, but the reason being is that ember has created all the controllers we have needed and so far we haven't had to extend their base functionality in any way. 
