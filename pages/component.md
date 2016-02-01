---
layout: default
title: Components
---

# {{ page.title }}

At the base of Wedge is the Component. All plugins of Wedge such as the form plugin derive from the Component object. Components provide a base amount of functionality to attach an HTML DOM, manipulate it, and interact with the user.

### Declare A Component

A very simple component to load an html page './public/user-add.html' and return it's contents on invoking a display call:

{% highlight ruby %}
class App
  class UserAdd < Wedge::Component
    name :user_add
    html './public/user-add.html'

    def display
      dom.html
    end
  end
end
{% endhighlight %}

You can instantiate this Wedge component and invoke the `display` method like so:

{% highlight ruby %}
wedge(:user_add).display
{% endhighlight %}

### Modify the DOM

Optionally, you can pass a block to the html method which will allow you to do some pre-processing on the DOM:

{% highlight ruby %}
class App
  class UserAdd < Wedge::Component
    name :user_add
    html './public/user-add.html' do
      dom.find('h1#title').html = "Welcome To My Site"
    end

    def display
      dom.html
    end
  end
end
{% endhighlight %}

This block is only run once at the start of your application. It is ideal to perform processing on the component that only needs to happen once at application startup. For example if you have a drop down of states or provinces, you can populate the list here. The state of the DOM at the end of this block is what will be available when an instance of this Wedge Component is created.

You can still modify the DOM inside instance methods such as `display`, however any changes made to the DOM here are instance changes and will not persist between component instances:

{% highlight ruby %}
def display
 dom.find('span#welcome-message').html "Welcome #{current_user.full_name}"
 dom.html
end
{% endhighlight %}

### Create and Use Templates

Templates are reusable pieces of HTML that get cached on the class level. You can use the `tmpl` method to both set and get a template. Templates are best created in the `html` block. When created inside this block, templates will be available on both client and server side.

{% highlight ruby %}
class App
  class UserList < Wedge::Component
    name :user_list
    html './public/user-list.html' do
      tmpl :user_list_item, dom.find('table.user-list>tbody>tr')
    end

    def display
      Models::User.all.each do |user|
        # Creates a copy of the template DOM
        t = tmpl :user_list_item
        t.find('td').html user.full_name
        dom.find('table.user-list>tbody').append t
      end
      dom.html
    end
  end
end
{% endhighlight %}

### Invoking Components Inside Other Components

Wedge components are heirarchical so you can call one component from inside another:

{% highlight ruby %}
class App
  class UserAdd < Wedge::Component
    name :user_add
    html './public/user-add.html'

    def display
      dom.find('div#user-notes').content = wedge(:user_notes).display
      dom.html
    end
  end
end
{% endhighlight %}
