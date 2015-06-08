---
layout: default
title: Client/Server Components
---

# {{ page.title }}

One of the biggest benefits of Wedge comes from it's integration with [Opal](http://opalrb.org/). With Opal, Wedge components can not only exist natively as Ruby classes server side, but can also be available as Javascript classes on the client. This allows one set of Ruby code to control actions both on the client and the server. Before proceeding with this page be sure you have properly wired up Wedge to your application with the [installation guide](/pages/install.html)

### Component Example

To illustrate this functionality on the most basic level, here is a component which will fire 2 sets of code depending on if the method is being called client or server side:

{% highlight ruby %}
class Root < Wedge::Component
  name :root
  html './public/index.html'

  def display
    if server?
      dom.html
    else
      dom.find('p').append '<span>This span added client side.'
    end
  end
end
{% endhighlight %}

With one command you can accomplish 2 tasks: render the server side code and execute the client side code upon the page fully loading in the user's browser. This can be seen in this Roda example:

{% highlight ruby %}
route do |r|
  r.root do
    wedge(:root).to_js(:display)
  end
end
{% endhighlight %}

The `to_js` method takes care of all the work for you. It generates all the necessary Javascript through the Opal compiler to allow all client side code of the Wedge component to run. It also gives you client side access to things like HTML templates (see [DOM](/pages/dom.html)).

### Using Events

At this point it is a good idea to check out the [Events](/pages/events.html) documentation since most of the time events are used to coorindate interaction between client side stimulus and server side functionality. Generally you will be looking out for button clicks or form submissions. You can wire up client side methods to fire upon these events and then invoke server side methods with data through Ajax.

### Server Calls From Client

Calling server methods from client side methods is seamless through Wedge. It is all done with plain ruby and even support passing basic data types and structures including: Hashes, Strings, Integers, Floats. There are 3 basic types of methods you can have:

* Server Only
* Server and Client
* Client Only

Server only methods are wrapped in an `on :server` block. Server and Client methods and Client Only methods go outside of the `on :server` block. You also have some helper methods which can assist you in deciding which code to execute:

* **client?** - will return true if this method is currently executing client side
* **server?** - will return true if this method is currently executing server side
* **from_client?** - will return true if this method was called from the client side
* **from_server?** - will return true if this method was called from the server side

Here is an example of a client and server side method interacting:

{% highlight ruby %}
class Root < Wedge::Component
  name :root
  html './public/index.html'

  def display
    # There's no immediate client side code to run here
    return unless server?
    dom.html
  end

  on :server do
    # This can only execute on server side
    def server_side_method data = {}
      if data[:foo] == 'bar'
        return {success: true}
      else
        return {success: false}
      end
    end
  end

  # capture a client side event when a button is clicked
  on :click, '#my-button' do |el, evt|
    evt.prevent_default
    # Invoke some client side method
    client_side_method
  end

  def client_side_method
    some_data = { foo: 'bar' }
    # this call a server side method and assess it's result
    server_side_method some_data do |res|
      if res[:success]
        dom.find('#status-message').html 'Server side method was a success'
      else
        dom.find('#status-message').html 'Server side method was a failure'
      end
    end
  end
end
{% endhighlight %}

Breaking this down step by step, we start off with a button on the HTML page with the ID 'my-button' being clicked. Because we declared a listener for this event in our component, it gets captured and the code in the event block executes. First we disable the normal behavior of the button with `evt.prevent_default` although this is an optional step. Next, we invoke the client side method appropriately called `client_side_method`.

Inside `client_side_method` we create a simple hash. Next we call `server_side_method` on the server and pass it the hash we created. Notice how this method call is accompanied with a block. This is because server side calls are made via Ajax and therefore asynchronous to the rest of the code execution. So, if we're waiting for return data from the server in order to take further action, we need to know what this return data is before we continue. Once we get a response from the server, the code inside the block will execute. We expect the code block to return a hash with a key `:success` which will let us know if the server performed the task we wanted it to. Depending on the value of the `:success` entry in the hash we will display a message to the HTML page. Inside client side methods you have access to the (Wedge::DOM)[/pages/dom.html] and can manipulate the DOM as you please.

Because `server_side_method` was wrapped in the `on :server` block, Wedge knows when you call this method from the client to do all the Ajax legwork to move data back and forth from the client. Now you can see how easy client/server interaction is with Wedge.

### Forms With Client/Server Interaction

One of the most useful purposes of Wedge is to send form submissions to the server, perform some server processing such as saving the data to a database, and then give the user some sort of feedback. Be sure to check the [Form Plugin](/pages/plugins/form.html) for examples on how to perform this.
