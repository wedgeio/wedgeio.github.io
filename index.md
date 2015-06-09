---
layout: default
title: Home
---

What Is Wedge
=============

Wedge was built from a desire to create an easy interface for interacting with users. With Wedge you have a lightweight set of tools to organize and present information, provide maximum code reusability, and interact between client and server actions seamlessly. Wedge is not a framework in itself. Rather it can be plugged in to the framework of your choice to help with rendering contact and accepting user feedback. Wedge also seeks to blur the lines between client and server side coding. With Wedge, you can build methods in plain Ruby which will execute server or client side.

Integration
===========

Wedge currently integrates into the Roda framework. Add it to any of your apps:

### Require in Gemfile

{% highlight ruby %}
gem 'wedge'
{% endhighlight %}

### Add To Roda

Wedge seamlessly integrates into Roda as a plugin. Simply invoke the plugin when declaring your Roda app. At this point you can pass configuration parameters as well. In your route you will need to also invoke the wedge_assets method so that the Wedge library can properly respond to requests.

{% highlight ruby %}
class App < Roda
  plugin :wedge, {
    scope: self,
    plugins: [:form]
  }

  route do |r|
    r.wedge_assets
  end
end

run App
{% endhighlight %}

### Create Your Own Wedge Components

All Wedge objects, even plugins, inherit from `Wedge::Component`. Each component is meant to render one block of user interaction. Components can also be called from within other components so you can build more complex interfaces and behaviors using simpler building blocks. Below is a very simple example of a Wedge component however there are many more options that make even the base Component object very powerful.

{% highlight ruby %}
class App
  class SampleComponent < Wedge::Component
    name :sample

    def display
      'This is a sample component'
    end
  end
end
{% endhighlight %}

### Invoke This Component In Your Route

Wedge in Roda reveals the `wedge` method. This method will instantiate a Wedge object with the config name matching the symbol you pass to the method as the first parameter. After that you can invoke any methods on this Wedge object.

{% highlight ruby %}
class App < Roda
  ... Roda setup goes here ...
  route do |r|
    r.wedge_assets
    r.root do
      wedge(:sample).display
    end
  end
end
{% endhighlight %}

### Explore Wedge

1. [Integrate With Frameworks](/pages/install.html)
1. [Component Basics](/pages/component.html)
1. [Client/Server Components](/pages/component-advanced.html)
1. [Events](/pages/events.html)
1. [Plugins](/pages/plugins.html)
    * [Form](/pages/plugins/form.html)
