---
title: getting started with ember, part 2
date: 2014-07-01 19:22 AEST
tags: ember, js
---

## Hamburger, I mean Handlebar Helpers

So far we've displayed our transactions on the page, but the amount property in our transaction model is just using a number to represent its value and we want to format it.
So we're going to utilise a helper method so it's available across our views.

Helpers in ember are not dissimilar to helpers in Rails, particularly in this use case where we're planning on formatting an existing value.
 The main difference is by utilising ember, we can ensure our helper method updates as the value/s it depends upon changes.

Today we're going to register a new helper and convert our transaction amount to something resembling currency. 

Ember CLI gives us a few options to achieve our goals, but in this case what we need is the following:

    import Ember from "ember";
    export default Ember.Handlebars.makeBoundHelper(function(amount, options){
      var amount = (amount/100).toFixed(2);
      var currencyAmount = "$"+ amount + "";
      return currencyAmount;
    }); 

Insert the above into `app/helpers/to-currency.js`.

We use `makeBoundHelper` here instead of just helper or `registerBoundHelper` due to ember-cli.
Ember-cli utilises the `makeBoundHelper` and makes it accessible to the ember container, this is outside the scope of this post, but if you'd like to dig a little into our applications internals within the console, run `Transactions.__container__` within your browser.

The alternative to using this is using `registerBoundHelper` is that we'd need to take the extra step to define the helper function, then register our helper within `app.js`.

If we reload our application now we should see our transaction amount looking all pretty.

## Controllers

In the previous post, we didn't really touch on controllers, however it would be really nice if we started creating some new transactions.

You can think of controllers in ember as the equivalent of view decorators in Rails. 
They interface between models and templates. 

Note, templates are connected to controllers, not models, the controller will proxy properties from its underlying model through to the template.
As far as the template is concerned, it doesn't talk to the model.

Controllers in ember come in three tasty flavours, `ObjectController` , `ArrayController` and plain old `Controller`, ember will inspect the model returned from your route to determine which one it should generate, but today it's up to us to determine which one to use when working with our transactions.



