# Welcome to Renee!

*Renee is the super-friendly Rack based web framework.*

    :::ruby
    run Renee {
      path('/') { halt "Hello Renee!" }
    }

This site was been built using Renee and is [available on Github](https://github.com/renee-project/renee-site).

## Concept (Why Renee?)

**Renee is a new way to think about writing web applications.**

### Hierarchical

Traditionally, routing and controller logic have been separate. In Rails, for instance, your path is matched to a controller and an action. This does not reflect the hierarchical nature of REST.

Consider a simple example. The route `/posts/45/comments`. Typically, you'd expect this to load the post with the id 45, and then load the comments on that post. In [Rails](http://rubyonrails.org/), your code to load a post and understand that parameter would have to be in both your posts controller and your comments controller. [Sinatra](http://www.sinatrarb.com/) does no better as it searches linearly though a list of path to find a matching path, and then executes the block associated with it.

To model this same idea in Renee, you could do the following:

    :::ruby
    run Renee do
      path 'posts' do
        var Integer do |id|
          post = Posts.find(id)
          path 'comments' do
            get { render! "comments", :comments => post.comments }
          end
        end
      end
    end

Suddenly, you have access to the previously referred to part of the path, namely, the `/posts/45` part. It's a locally scoped variable, as is the id, so you don't have to worry about anyone outside of it's scope having access to it.

To find out more, take a look at the [routing methods](/routing) available to you.

### Composability

Renee lives and breathes inside of [Rack](http://rack.rubyforge.org/). Let's take a look at the example above and understand a little better what's going on. We'll modify it slightly for the sake of clarity:

    :::ruby
    run Renee do
      p request.path_info     # printing
      path "posts" do
        p request.path_info   # printing
        var Integer do |id|
          p request.path_info # printing
          halt :ok
        end
      end
    end

If you run a request with the path `/posts/12` through here, you'll get three print statements:

    :::ruby
    "/posts/123"
    "/123"
    ""

The PATH_INFO is being consumed by each scope. If you don't halt, don't worry, your request will get put back together again after it falls out of each block. This let's you move part of your application around without fearing how the route is being consumed.

### Rack integration

Renee loves Rack. To run a arbitrary rack end point, you can use `#run!` to stop execution and pass off your request to an Rack application. An example:

    :::ruby
    run Renee do
      path "posts" do
        run! PostsEndpoint # this can be any Rack application
      end
    end

To find out more about integrating with rack, take a look at [rack integration](/rack-integration) to find out more!

## Getting started

### Installation

Renee is gem-based. If you're using rubygems, you can simply:

    $> gem install renee

If you're using [Bundler](http://gembundler.com/), you can add

    :::ruby
    gem 'renee', '~> 0.0.1'

to your `Gemfile`.

### Overview

Renee has (hopefully) a small number of keywords divided between several components:

* *Routing* is done either on the path, the query string, or other parts of the request headers.
* *Responding* makes it easy to respond to a request.
* *Rendering* gives you access to [Tilt](https://github.com/rtomayko/tilt) for rendering templates.
* *Rack interaction* makes it easy to call into [Rack](http://rack.rubyforge.org/)-based applications.
* *Request context* gives you access to the request and gives the basis for responding.

## Usage

Using Renee is as simple as understanding how to *configure settings*, *define routes*, and *respond to requests*.
Renee usage in a nutshell:

    :::ruby
    run Renee {
      path 'blog' do
        get  { render! "posts/index" }
        post { Blog.create(request.params); halt :created }
        var do |id|
          @blog = Blog.get(id)
          get { render! "posts/show" }
          put { @blog.update(request.params); halt :ok }
        end
      end
    }.setup {
      views_path "./views"
    }

Check out detailed guides for each aspect below:

[&#8618; Read about Configuration](/settings)

[&#8618; Read about Responding and Rendering](/responding)

[&#8618; Read about Routing](/routing)

[&#8618; Read about Route generation](/route-generation)

## API documentation

Renee is also well-documented with YARD:

[&#8618; renee](/doc/meta/index.html)

[&#8618; renee-core](/doc/core/index.html)

[&#8618; renee-render](/doc/render/index.html)

## Development

Renee's structure is pretty simple so far. The basic Rack DSL is contained in
[renee-core](https://github.com/renee-project/renee/tree/master/renee-core). This gem has no other dependencies other than Rack.

The rendering side is in [renee-render](https://github.com/renee-project/renee/tree/master/renee-render),
which depends on [Tilt](https://github.com/rtomayko/tilt).

The kitchen-sink gem which incorporates all of the others is [renee](https://github.com/renee-project/renee/tree/master/renee).
Please, any bugs, any ideas, I'd love to hear any of it. Love, [Team Renee](/team-renee). &hearts;