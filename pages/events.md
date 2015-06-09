---
layout: default
title: Events
---

# {{ page.title }}

Events will allow your components react to outside stimulus. In most cases, events will be responding on the client side so reading the notes on [Client/Server Components](/pages/component-advanced.html) will be helpful for this document. Events in Wedge take a similar approach to how you would see events in JQuery. You have a set of standard events that you can handle or you can declare your own custom events. Usually you are listening for some sort of user input on your rendered HTML page.

### Listen For An Event

Events are declared by invoking the `on` method and passing relevant event parameters and a block.

{% highlight ruby %}
class App
  class UserAdd < Wedge::Component
    name :user_add
    html './public/user-add.html'

    def display
      dom.html
    end

    on :click, '#some-button' do |el, evt|
      dom.find('#action-text').html 'Some button was clicked!'
    end
  end
end
{% endhighlight %}

Here we are listening for the `:click` event on an element in the HTML page with the ID 'some-button'. When an event is triggered, a DOM reference to the element that was triggered along with a reference to the event itself is passed to the event block. Examining and manipulating these objects can be helpful in the processing of the event. Here is an example:

{% highlight ruby %}
on :click, '.action-button' do |el, evt|
  evt.prevent_default
  if el.html == 'Save User'
    save_user
  elsif el.html == 'Delete User'
    delete_user
  else
    puts "Unable to determine action"
  end
end
{% endhighlight %}

Let's assume 'action-button' is a class for anchor tags. Anytime one of these action-buttons gets clicked, this event will trigger. The first thing we do, since we have access to the event object, is to prevent the default behavior of the link. Next, since we have access to the element that was clicked, we can examine the contents of the anchor tag and see what the text is. Depending on the value of the text we can perform different actions.

### The :submit event

One special event is the `:submit` event. This one behaves differently than other events because it has the specific purpose of taking form submissions and populating Wedge Forms with the data. Here is a structure of a submit event:

{% highlight ruby %}
on :submit, 'form#form-id', form: :my_form, key: :user do |form, el|
  if form.valid?
    # do something with the data
  end
end
{% endhighlight %}

The submit event starts like most events in that the `on` block takes :submit as it's first parameter and the ID of the element to watch for submissions as it's second parameter. However there are more options that you can pass to the submit event:

* **form** - the Wedge Form name which you want to populate the submitted form data with
* **key** - The prefix in the form field names that holds the data for this Wedge Form. For example if you name all your input fields like so: user[first_name], user[last_name] ... then your key would be 'user'

For more details on the `:submit` event see the [Form Plugin](/pages/plugins/form.html) documentation.
