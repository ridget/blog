---
title: getting started with ember, part 2
date: 2014-07-01 19:22 AEST
tags: ember, js
---

## Hamburger, I mean Handlebar Helpers

Before we get started I'd like to give a huge thank you to [@geoffreyd](https://twitter.com/geoffreyd), he pretty much fixed all my code and set me straight on some really good best practices, not to mention spent an enormous amount of time putting up with my help vampirism. Additionally all the code for this post can be found on [github](https://github.com/ridget/transactions/tree/section_2)

So far we've displayed our transactions on the page, but the amount property in our transaction model is just using a number to represent its value and we want to format it.
So we're going to utilise a helper method so it's available across our views.

Helpers in ember are not dissimilar to helpers in Rails, particularly in this use case where we're planning on formatting an existing value.
 The main difference is by utilising ember, we can ensure our helper method updates as the values it depends upon changes.

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

## Routes

This is where most guides would start discussing Ember controllers and creating new transactions from within. We're not going down that path.

A Route in Ember represents state, which is represented with a url. As we're going to be manipulating data, and thus the state of the current page, we're going to store our actions relating to creating a new transaction in a route. To start though we need to specify a new route for the user to hit.

    Router.map(function() {
      this.resource('transactions', { path: '/' }, function(){
        this.route('new'); 
      });
    });

Here we have made `/transactions/new` available. This, by convention, will ensure Ember generates a `TransactionsNewRoute`, but we're going to create one of our own and start handling user input. To take user input we need a form, so let's create a new template for our transactions form:

    <form {{action "createTransaction" on="submit"}}>
      <label for="transaction-name">Name:</label>
      {{input type="text" value=name id="name"}}

      <label for="transaction-amount">Transaction Amount:</label>
      {{input value=dollarValue id="comp-currency"}}

      <button type="submit">Save Transaction</button>
    </form>

In Ember, if we don't assign a string to a value, that then becomes available to the context object within the template, bear this in mind when we start to look at our Route object later on.

So now that we have a form, let's make our form save our transactions with a Route:

    import Ember from 'ember';

    export default Ember.Route.extend({
        model: function(){
            return this.store.createRecord("transaction", { amount: 0, name: 'New Transaction' });
        },

        actions: {
          createTransaction: function(){
            var self = this;
            var transaction = this.controller.get('model');
            // TODO: set date format the way you want it.
            transaction.set('occurredOn', new Date);
            transaction.save().then(function(){
              self.transitionTo('transactions.index');
            });
          }
        }

    });

Ok, there's a lot going on here, so lets break it down.

First up, we instantiate a brand new instance of transaction within the route, place it inside the model hook, and set it with some defaults using the `createRecord` method. 
This then sends the model to the controller, which if you remember from our first post talks exclusively to the template. 

Next is our actions, specifically our `createTransaction` action. We can define actions within a Route object, a Controller or a View. To make understanding when to use which earlier, if you're dealing with data changes, use the Route, if you're dealing with changes within the object or collection, use a Controller, if you're dealing with interactions between views, use the View.

Within our `createTransaction` action, we retrieve our controller model, which seems a bit odd as we've defined it above. However we need to explicitly retrieve the model object as it stands from the controller, which as we reminded ourselves above, talks to the template. So here, we get the object as it stands in our form. We then save our new transaction, and make use of promises to 
return to the transactions index once complete.

There's one piece to puzzle missing, how does our `dollarValue` from our template, map to the `amount` on our model. The answer is in fact, in our model.

## Computed Properties

One of Ember's most powerful features is computed properties.

It allows you to declare functions as properties on an object. When we create a computed property we defined the properties of the object it depends on at the end of the function, we can also
use this to build a computed property that depends on another computed property. Awesome!

In our case below we have a computed property of `dollarValue` to map dollar values to cents in our database. When we use this property in our form, it sets the amount to the unformatted amount entered in. So when it comes time to saving the form, it automatically does all the adjustments for us. 

    var Transaction = DS.Model.extend({
        name: DS.attr('string'),
        amount: DS.attr('number'),
        occurredOn: DS.attr('date'),
        dollarValue: function(key, value, previousValue) {
            var currency;
            if (value !== undefined) {  // set was called
                var amount = Math.round(value * 100);
                currency = accounting.unformat(amount);
                this.set("amount", currency);
            } else {
              currency = this.get('amount');
            }

            // Get the value and return for either set or get.
            amount = (currency/100).toFixed(2);
            return accounting.formatMoney(amount, "");
        }.property('amount')
    });

## One more thing

There's just one thing we need to do, and that's instantiate an Ember Object Controller so our form gets all the attributes proxied through. So let's create a new `TransactionsNew` controller:

    import Ember from "ember";

    // Make TransactionsNewController an ObjectController,·
    // so that it will proxy properties. !important
    export default Ember.ObjectController.extend({

    });
