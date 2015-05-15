---
layout: default
title: A Basic User Input Form With Wedge
---

# {{ page.title }}

In this example we will show how to create a basic Roda app with Wedge integration. The wedge component will be responsible for taking input for a new user, validating the input data client side, and adding the new user into the database. It is recommended you download the final source from here to navigate through all these files as they are explained.

### Creating The Roda App

First create the Gemfile with all the libraries we will need for this simple example:

{% highlight ruby %}
source 'https://rubygems.org'

gem 'thin' # web server
gem 'roda' # roda framework
gem 'wedge' # wedge components
gem 'sequel' # database interaction
{% endhighlight %}

Next we will make the config.ru file which will declare our Roda app, define the route to our wedge component to interact with the user, and run the app.

{% highlight ruby %}
require 'roda'
require 'wedge'

class App < Roda
  plugin :wedge, {
    scope: self,
    plugins: [:form]
  }

  route do |r|
    r.root do
      wedge(:user_add, :js).display
    end
    r.wedge_assets
  end
end

Dir['./components/**/*.rb'].sort.each { |rb| require rb }

run App
{% endhighlight %}

Now we need to create the actual user_add component. Notice in the code above we created a components folder. This is a good place to put all the Wedge components you will build although folder structure is completely up to you. Create a new user_add.rb file in this folder for our new Wedge component:

{% highlight ruby %}
class App
  class UserAdd < Wedge::Component
    config.name :user_add
    config.html './public/user-add.html'

    def display
      dom.html
    end
  end
end
{% endhighlight %}

Wedge components can take HTML files during configuration. If you provide a path to an HTMl file, the component will automatically load this up as a `Wedge::DOM` object which is based on a `Nokogiri::HTML` object. This gives you an object oriented approach to modifying the HTML page and then rendering out to a browser. You will need to create this HTML page at ./public/user-add.html:

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
  <title>Add A User</title>
</head>
<body>

<div id="status-message"></div>

<form id="user-add-form">
  First Name: <input type="text" name="user[first_name]" value="" />
  <br /><br />
  Last Name: <input type="text" name="user[last_name]" value="" />
  <br /><br />
  E-Mail: <input type="text" name="user[email]" value="" />
  <br /><br />
  <button type="submit">Add User</button>
</form>

</body>
</html>
{% endhighlight %}

With this our basic app is set up. You can run this app by starting your thin server using the command `bundle exec thin start`. It will by default create a server on localhost port 3000. Visiting this page you will see a simple form that takes 3 inputs for a user's first name, last name and email. However, this form currently doesn't do anything if it is submitted. We'll have to add a few things to get that to happen.
