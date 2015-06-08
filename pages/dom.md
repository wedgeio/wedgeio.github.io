---
layout: default
title: DOM
---

# {{ page.title }}

The Document Object Model allows you to associate an HTML document to the component. Built on top of Nokogiri, you will notice much of the similar behavior. There are 2 opportunities to perform processing on the DOM.

### Class Level Processing

During the html block of the class you can load up an HTML file by providing it's path and then perform one time processing on it which occurs once at the start of the application and then cached:

{% highlight ruby %}
class UserAdd < Wedge::Component
  name :user_add
  html './public/user-add.html' do
    dom.find('h1#title') = "#{SITE_NAME} Title"
  end

  def display
    dom.html
  end
end
{% endhighlight %}

Optionally you can omit the first parameter to html and simply pass a block which populates the class variable `@wedge_dom`:

{% highlight ruby %}
class UserAdd < Wedge::Component
  name :user_add
  html './public/user-add.html' do
    @wedge_dom = Wedge::DOM.new('<html><body><p>This is a dom.</p></body></html>')
  end

  def display
    dom.html
  end
end
{% endhighlight %}

The results of the html block are then cached at the class level and made available every time you instantiate the Wedge component.

### Instance Level Processing

Whenever you instantiate a Wedge component and call an instance method, you are given access to a copy of the class-level cached DOM:

{% highlight ruby %}
class UserAdd < Wedge::Component
  name :user_add
  html './public/user-add.html' do
    @wedge_dom = Wedge::DOM.new('<html><body><p>This is a dom.</p></body></html>')
  end

  def display
    dom.find('p').html "I am modifying the DOM at the instance level"
    dom.html
  end
end

response.write wedge(:user_add).display
{% endhighlight %}

Any changes made by calling an instance method do not carry over when you instantiate a new copy of the Wedge component. Consider this example:

{% highlight ruby %}
class UserAdd < Wedge::Component
  name :user_add
  html './public/user-add.html' do
    @wedge_dom = Wedge::DOM.new('<html><body><p>This is a dom.</p></body></html>')
  end

  def change_content
    dom.find('p').html "I am modifying the DOM at the instance level"
    display
  end

  def display
    dom.html
  end
end

user_add = wedge(:user_add)
response.write user_add.change_content
response.write user_add.display

# New Wedge component will work with a new copy of the DOM
user_add_new = wedge(:user_add)
# This will render the DOM unmodified
response.write user_add.display
{% endhighlight %}

Note `user_add_new` creates a new instance of the `user_add` Wedge component and therefore received a fresh copy of the DOM. No changes made by the original `user_add` Component will show up in this new copy.

### Modifying the DOM

Because the Wedge DOM is built on Nokogiri, a lot of the functionality and behavior is as you would expect with a direct Nokogiri::HTML::Document object, but with some notable changes to accommodate Wedge's functionality. You can search for an element or set of elements in the document:

{% highlight ruby %}
def display
  # Modify a single element
  dom.find('h1#title').html "#{SITE_NAME} Admin"

  # Loop through a set of elements
  dom.find('h2').each do |el|
    el.html "<b>#{el.html}</b>"
  end

  # Add child elements
  dom.find('p').add_child "<span>This will be added to the end of the paragraph tag"
end
{% endhighlight %}

The `find` method is the preferred way of searching through the DOM because it can work on both server-side and client-side Wedge code. However, on server side execution, you can use any Nokogiri HTML Document object call and it will be valid.
