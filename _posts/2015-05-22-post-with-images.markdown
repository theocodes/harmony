---
layout: post
title:  "Lotus framework - Part 1"
date:   2015-05-22 18:56:45
disqus: true
categories:
- framework
- ruby
- web
- programming
permalink: lotus-framework-part-1
description: In this series of posts I'll talk about what Lotus is and the ideas behind it, how to build a simple app with it and why it has quickly become my favorite web framework in ruby.
---

[![Lotus Framework logo]({{ site.url }}/assets/images/lotus.png)]({{ site.url }}/{{page.permalink}})

{{page.description}}

---

### Setup

Recently my team and I were assigned a task of breaking down a massive, decade-old rails application into smaller pieces, and make it easier to maintain, implement new features etc. Based on how complex we this monolithic beast has become over the years, Rails didn't seem like the right choice going forward, so we began looking at alternatives. 

In the Ruby world, probably due to how well rails did, it feels like when we shout 'WEB FRAMEWORKS' into a dark cave, we hear back "RAILS", "Sinatra...", "Padri...." until it becomes so low, you can't hear anything anymore.

##### [Lotus](http://lotusrb.org/) is here to change that... 

The feeling I get after having played around with Lotus for a few weeks is that much of the pains I felt with maintaining legacy monolithic applications were also felt by the team behind it.

While still in its infancy in comparison to its well established alternatives, Lotus does a very good job at elegantly setting the scene for complex applications in a way that is manageable, and it does so with the help of pluggable frameworks that encapsulate varies responsibilities.

Lotus provides you with some generators to help you get off the ground but ultimately gives you the freedom to design things however you like. It's just as easy to customize things as it is to follow conventions.

### The project 

In this series, to illustrate what I've been talking about above and share my experience, we'll be creating a TODO tasks app in two parts. A web client that will be responsible for the presentation and all the user sees and a JSON API that will be responsible for creating, updating and deleting tasks.

While a TODO app may not be the most exciting of projects, I find that a web client and an API for it to talk to, offers enough content to illustrate the separation of responsibilities and it's a setup a lot of people might be interested in exploring when considering something other than your typical monolithic application. 

---

#### Part 1

In this first part we'll install lotus, explore the project's structure and some of lotus's built in generators.

**Note:** *ruby 2+ is required*

Go ahead and install Lotus with: 

`gem install lotusrb`

Generate an app and 'cd' into the project with:

`lotus new lotus-todo && cd lotus-todo`

Open the project with an editor or IDE and lets have a look at the files Lotus created for us.

##### Directory structure   

![Lotus Framework logo]({{ site.url }}/assets/images/tree.png)

As you can see, Lotus creates a few directories to point us at the right direction. 

`/apps` As the name suggests, is where we'll put our applications. In our case `web`, which Lotus already created for us, and later our api `todos`.  

`/config` contains 3 `.env` files which we use to set environment variables (more on this later) and `environment.rb` which is where you'll require gems, ruby code (gems, modules, etc..) you want available across the app. By default, you'll also mount your varies apps here. When we come around to building our API app, we'll mount it here so the routers and the rest of the framework is aware of it and loads it for us.  

`/db` Database related stuff such as migrations etc.. 

`/lib` Is where we put use cases. Lotus encourages us to create our applications as gems so you're able to implement features independently of the delivery mechanism (more on this later). 

`/spec` Tests.. of course!

---

We'll talk about the files and sub-directories as we move along. For now...

Let's go ahead and open `lotus-todo.rb`.

{% highlight ruby %}
require 'lotus/model'
Dir["#{ __dir__ }/lotus-todo/**/*.rb"].each { |file| require_relative file }

Lotus::Model.configure do
  # Database adapter
  #
  # Available options:
  #
  #  * Memory adapter
  #    adapter type: :memory, uri: 'memory://localhost/lotus-todo_development'
  #
  #  * SQL adapter
  #    adapter type: :sql, uri: 'sqlite://db/lotus-todo_development.sqlite3'
  #    adapter type: :sql, uri: 'postgres://localhost/lotus-todo_development'
  #    adapter type: :sql, uri: 'mysql://localhost/lotus-todo_development'
  #
  adapter type: :file_system, uri: ENV['LOTUS_TODO_DATABASE_URL']

  ##
  # Database mapping
  #
  # Intended for specifying application wide mappings.
  #
  # You can specify mapping file to load with:
  #
  # mapping "#{__dir__}/config/mapping"
  #
  # Alternatively, you can use a block syntax like the following:
  #
  mapping do
    # collection :users do
    #   entity     User
    #   repository UserRepository
    #
    #   attribute :id,   Integer
    #   attribute :name, String
    # end
  end
end.load!

{% endhighlight %}

There's quite a bit here, so let's take it step by step:

{% highlight ruby %}
require 'lotus/model'
{% endhighlight %}

Remember when I mentioned that Lotus uses pluggable smaller frameworks? Here, because it's about to use the `lotus/model` framework, it requires it.

{% highlight ruby %}
Dir["#{ __dir__ }/lotus-todo/**/*.rb"].each { |file| require_relative file }
{% endhighlight %}

Lotus will load all files you put in `/lib/lotus-todo` for you.
