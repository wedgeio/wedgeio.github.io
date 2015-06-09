---
layout: default
title: Integrate With Frameworks
---

# {{ page.title }}

1. Require Wedge Gem
1. Link Your Framework To Wedge
1. Add Client Side JS Tags

### Require Wedge Gem

Add to your Gemfile:

{% highlight ruby %}
gem 'wedge'
{% endhighlight %}

Inside your application's initialization:

{% highlight ruby %}
require 'wedge'
{% endhighlight %}

### Link Your Framework To Wedge

Wedge integration is currently supported on the following frameworks:

#### Roda

{% highlight ruby %}
class App < Roda
  # Wedge is included as a plugin
  plugin :wedge, {
    scope: self,
    plugins: [:form]
  }

  route do |r|
    # This will set the necessary routes for handling things like Ajax
    # server calls and Wedge compiled Javascript files
    r.wedge_assets
  end
end

run App
{% endhighlight %}

### Add Client Side JS Tags

If you will be using the client side functionality of Wedge you will need to include the Wedge javascript tag in your HTML template. This HTML tag is automatically generated for you using the method call `Wedge.script_tag`

### Configuration Options

* **scope** - Object - The application Wedge is being linked to
* **plugins** - Array - A  list of plugins to auto load for every Component (see [Plugins](/pages/plugins.html) for full list)
* **cache_assets** - Boolean - stores all compiled Javascript code in cache so it is not recompiled on every page request
* **assets_url** - String - Path to assets. Can be relative path or an absolute URL. This is useful for integrating with content delivery systems such as Cloudfront
