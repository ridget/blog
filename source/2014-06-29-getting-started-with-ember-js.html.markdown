---
title: Getting Started With Ember.js
date: 2014-06-29 06:40 UTC
tags: js, ember.js
---

## Getting started on getting started
Ember is an opinionated JavaScript framework, it subscribes to the Rails philosophy of convention over configuration and makes it very easy to start building well structured,
rich, client side applications.

Ember CLI is a tool used to help build ember applications, and similarly has a strong opinion on how you should structure and work with your code. Today's blog post is going to be using both of these tools.

As a Rails developer, the urge to utilise the ember-rails gem is pretty high, however I'm of the opinion that you should keep your ember application separate from your Rails app and ember cli provides too good a tool to ignore.

Ember CLI is a command line tool thats based on the Ember App Kit project, it is meant to function as a replacement to Ember App Kit. For our purposes all we need to know now is that it gives us a great project structure out of the box (not unlike when you generate a new Rails project with Rails new), and a few handy generators.

To get started, please follow the instructions on (insert ember cli address and hash to getting started), and come back when you're ready to proceed.

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
