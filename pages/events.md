---
layout: default
title: Events
---

# {{ page.title }}

Events will allow your components react to outside stimulus. Primarily you can listen for user input.

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
