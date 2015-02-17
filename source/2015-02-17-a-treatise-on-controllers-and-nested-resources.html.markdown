---
title: A Treatise on Controllers and Nested Resources
date: 2015-02-17 19:51 AEST
tags: rails, controllers
---

## Decouple yourself from only one controller per resource

I have an uneasy relationship with conditionals, but I am particularly wary of anything that is more than an `if` and `else`, especially in a controller. 

That's not to say I don't believe they have their place, but I'll always look to try and reach for another option, or hoist the conditional up, rather than maintain them in a given method. You see, the issue I have is that complex conditionals, introduce multiple collaborators into a method, reduce the ability to reason about code, and make the system harder to change. 

[Steve Klabnik](http://blog.steveklabnik.com/posts/2011-12-30-active-record-considered-harmful) also gives some insight into the issue:

> ActionController relies on instance variables to pass information from the controller to the view. Have you ever seen a 200 line long controller method? I have. Good luck teasing out which instance variables actually get set over the course of all those nested ifs.

Let's take a look at the below:

	# app/controllers/posts_controller.rb
	class PostsController < ApplicationController
	  def index
	    ...
	    case
	    when @user
	      @user.posts
	    when @company
	      @company.posts
	    else
	      Post.all
	    end
	  end
	  ...
	end

We can reason that the `index` method has the role of returning a list of posts, and certainly, for the time being, whilst I feel uneasy about the case statement, at least it's returning a collection of the same kind of object. However, it feels *off* and is certainly harder to test (yes, I do test my controllers), not to mention, flies in the face of [tell don't ask](https://pragprog.com/articles/tell-dont-ask). 

What happens when we start to introduce logic into our views as a result of this case statement though? Where we check for the presence of an instance variable, to determine what and what not to display? If I start reaching for a presenter, this still leaves the underlying problem in our action.

What happens when we introduce authorization and roles into the mix? All of a sudden our method is now involved in determining what to display by virtue of a users role, as well as their scope.

Whilst the tight coupling between our controllers and our views certainly doesn't help, I'm of the strong opinion that by increasing the controllers, we can reduce logic in our views as well as increase code clarity. This is not a new or original concept, [Rails anti-patterns](http://www.amazon.com/Rails-AntiPatterns-Refactoring-Addison-Wesley-Professional/dp/0321604814), describes the same issue 

> At some point - a point that comes more quickly than you might think - it becomes beneficial not to try to keep the same controller for each difference resource path but to instead create a new controller for each nesting.

Whilst our contrived example may not be at that point just yet, it won't be long before it is.

So let's split these controllers up

	# app/controllers/posts_controller.rb
	class PostsController < ApplicationController
	  def index
	    Post.all
	  end
	end

	# app/controllers/users/posts_controller.rb
	class PostsController < ApplicationController
	  def index
	    @user.posts
	  end
	end

	# app/controllers/companies/posts_controller.rb
	class PostsController < ApplicationController
	  def index
	    @company.posts
	  end
	end

At the cost of more classes, our *extremely* simple example is now easier to reason about and to change. Shared behaviour can be included via concerns, and views either replaced completely, composed of partials, or we can wrap our collections with presenter objects to properly house the logic that remains in the view layer. We are now further enabled to make changes to our code, without being too concerned about our other actors and we have removed condtionals from our controller actions and pushed the decision making process up to our router. 

## A Note on CanCan

`load_and_authorize` hides the logic that can determine which collection of a given resource to display away. So in our contrived example, we wouldn't necessarily have a case statement, unless we needed a scope on one of our actions. Favouring explicitness over implicitness aside, it obfuscates the conditional, but still has all the downsides. Any changes we make to our controller action, now has to deal with the additional actors, as well as is now harder to reason about when or if a particular parent resource is loaded. This is why im considering a move to [pundit](https://github.com/elabs/pundit), but that's another post.

