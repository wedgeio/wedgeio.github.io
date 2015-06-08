---
layout: default
title: Plugins - Form
---

# {{ page.title }}

Forms are used as an intermediary data formatting and validation tool between your front end UI and your database models. Forms have multiple userful purposes:

- Data Validation - ensure that the submitted data is in the expected format and provide an error reporting data structure if not.
- Data Filtering - ensure that only the fields you expect to be updated get sent to your database model.
- Data Presentation - pre-process data to display to the user in a more readable format

### Data Validation

Usually, a Wedge Form will correspond to a subset of data from a table or set of tables in your database. Imagine for example a signup process for a user. It may be a multi-step process so you collect a little bit of information from the user in each step. Putting this data validation in the model wouldn't make sense because each step validates a different group of data. So a form provides the intermediary to take a lot of the workload off of your model. Here is a sample HTML form in a registration process:

{% highlight html %}
<form>
  <strong>User Registration - Step 1</strong><br /><br />
  E-Mail: <input type="text" name="user[email]" /><br />
  Password: <input type="password" name="user[password]" /><br />
  Confirm Password: <input type="password" name="user[password_confirm]" /><br />
  <button type="submit">Continue</button>
</form>
{% endhighlight %}

We now need to define a form to accept this data and validate it's contents. You specify each field you want this form to handle by setting them with `attr_accessor`. You then define a `validate` method to handle your validations:

{% highlight ruby %}
class UserStep1 < Wedge::Plugins::Form
  name :user_step1_form
  attr_accessor :email, :password, :password_confirm

  def validate
    assert_present :email
    assert_present :password
    assert_present :password_confirm
    assert_equal :password, password_confirm
  end
end
{% endhighlight %}

Note that all forms must have a Wedge configuration name that ends in `_form`. Now all that is left is to connect the submission data to the form. So where you receive the submit data you simply initialize the form and run the `valid?` method to check whether all your validations pass. If they pass you can use the form's `model_attributes` method to return a hash which a supporting database library such as Sequel or Active Record can understand to update a database entry. If the tests do not pass, the form's `errors` hash is populated with all the fields that have issues.

{% highlight ruby %}
step1_form = wedge(:user_step1_form, params[:user])
if step1_form.valid?
  new_user = Models::User.create( step1_form.model_attributes )
else
  return step1_form.errors
end
{% endhighlight %}

### Data Filtering

Forms also allow you to ensure that you only receive the data you are expecting. Potentially malicious submissions may try to submit extra data and without proper filtering of unwanted fields you may update unintended information in the database. Consider the HTML form above was modified:

{% highlight html %}
<form>
  <strong>User Registration - Step 1</strong><br /><br />
  E-Mail: <input type="text" name="user[email]" /><br />
  Password: <input type="password" name="user[password]" /><br />
  Confirm Password: <input type="password" name="user[password_confirm]" /><br />
  Override Auth Level: <input type="text" name="user[auth_level]" value="admin" /><br />
  <button type="submit">Continue</button>
</form>
{% endhighlight %}

Adding a new field `user[auth_level]` could update the auth_level entry for the user in the database if you pass the hash directly. However, since you are running this data through a Wedge Form, and the form does not have an `attr_accessor` for auth_level, this entry in the submission data will be ignored and not passed in `model_attributes`.

### Data Formatting

Forms can be useful for performing some pre-processing on data before it gets sent to the user interface. Here is a form that has both monetary values and date values which we want to format differently than how the form received the data (usually this data comes from a database). By overriding the accessor method we can perform the necessary processing.

{% highlight ruby %}
class Reservation < Wedge::Plugins::Form
  name :reservation_form
  attr_accessor :fee, :reserved_at

  # Format this into something like $X,XXX.XX
  def fee
    ('%.2f' % (super)).gsub(/(\d)(?=(\d{3})+(\.\d*)?$)/, '\1,')
  end

  # Format this into something like mm/dd/YYYY
  def reserved_at
    super.strftime('%m/%d/%Y')
  end
end
{% endhighlight %}

These formattings will not be applied however when you call `model_attributes` since usually the formatting converts the data away from a format that a database would be able to work with.

### A Complete Example
