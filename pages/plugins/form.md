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

Note that all forms must have a Wedge configuration name that ends in `_form`. Now all that is left is to connect the submission data to the form. So where you receive the submit data you simply initialize the form and run the `valid?` method to check whether all your validations pass. If they pass you can use the form's `model_attributes` method to return a hash which a supporting database library such as Sequel or Active Record can understand to update a database entry. Alternately if you want to dump the data into a format that you can send into another form object, you would use the `attributes` method instead. If the tests do not pass, the form's `errors` hash is populated with all the fields that have issues.

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

Here is a quick expanded example of the User Signup Step 1 example above.

Our HTML form would go in `./public/user-signup-1.html`:

{% highlight html %}
<form>
  <strong>User Registration - Step 1</strong><br /><br />
  <span id="status-message">Status Message Here</span>
  E-Mail: <input type="text" name="user[email]" /><br />
  Password: <input type="password" name="user[password]" /><br />
  Confirm Password: <input type="password" name="user[password_confirm]" /><br />
  <button type="submit">Continue</button>
</form>
{% endhighlight %}

Our Wedge Form would go in `./app/forms/signup/step1.rb`:

{% highlight ruby %}
module Forms
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
end
{% endhighlight %}

Our Wedge Component that will coordinate the signup process would go in `./app/components/signup/step1.rb`:

{% highlight ruby %}
require './app/forms/signup/step1.rb'

module Components
  class UserStep1 < Wedge::Component
    name :user_step1
    html './public/user-signup-1.html'

    def display
      dom.html
    end

    on :server do
      def save attributes
        form = wedge(:user_step1_form, attributes)
        return {success: false, errors: form.errors} unless form.valid?
        model_attributes = form.model_attributes
        # The password confirm field does not go into the database
        model_attributes.delete :password_confirm
        user = Models::User.create model_attributes
      # In the case that validations in the database model fail
      rescue Sequel::ValidationFailed
        {success: false, errors: user.errors}
      end
    end

    on :submit, 'form', form: :user_step1_form, key: :user do |form, el|
      # disable submit button until ajax submission is complete
      button = el.find('button[type="submit"]')
      button.prop("disabled", true)

      if form.valid?
        save form.attributes do |res|
          if res[:success]
            dom.find('#status-message').html 'User was created successfully.'
          # Server side save failed (likely caused by database validation failure)
          else
            form.display_errors errors: res[:errors], tmpl: tmpl(:field_error)
          end
          button.prop("disabled", false)
        end
      # Client side validation failed
      else
        form.display_errors tmpl: tmpl(:field_error)
        button.prop("disabled", false)
      end
    end
  end
end
{% endhighlight %}

And lastly invoke `display` method on the Wedge component from your app's routing block:

{% highlight ruby %}
route do |r|
  r.on 'signup/step1'
    wedge(:user_step1).to_js(:display)
  end
end
{% endhighlight %}

### Nested Forms

Forms can be nested within each other much like you can have nested attributes in a database model. Consider a user object that has an address:

{% highlight ruby %}
module Forms
  class Address < Wedge::Plugins::Form
    name :address_form
    attr_accessor :address1, :address2, :city, :state, :zipcode
  end

  class User < Wedge::Plugins::Form
    name :user_form
    attr_accessor :email, :name, :password
    # nested form
    attr_accessor address: :address_form
  end
end
{% endhighlight %}

To specify an attribute that is a nested form all you have to do is pass the attribute name and form name as a hash as seen in the example above. You can directly populate values in this nested form by observing the proper naming convention in your form fields:

{% highlight html %}
<form>
  <strong>User Edit</strong><br /><br />
  E-Mail: <input type="text" name="user[email]" /><br />
  Name: <input type="text" name="user[name]" /><br />
  Password: <input type="password" name="user[password]" /><br />
  <br />
  Address 1: <input type="text" name="user[address][address1]" /><br />
  Address 2: <input type="text" name="user[address][address2]" /><br />
  City: <input type="text" name="user[address][city]" /><br />
  State: <input type="text" name="user[address][state]" /><br />
  Zipcode: <input type="text" name="user[address][zipcode]" /><br />
  <button type="submit">Continue</button>
</form>
{% endhighlight %}

When using the `model_attributes` call as well, the data for nested attributes will be structured in a format that will work with nested attributes support in Sequel or Active Record.

### Method Reference

- **model_attributes** - returns a Hash of all data in a format compatible with ORM libraries such as Sequel and ActiveRecord.
- **attributes** - returns a Hash of all data in a format compatible with Wedge Forms
- **valid?** - run validation on current form. Will return true if all validations pass. Will return false and populate the `errors` hash on the form if any validations do not pass.
- **display_errors** - add user-friendly error messages to the DOM from the current values in the form's errors hash. This should be called after a `valid?` method call.

### Validation Reference

The following methods are available for validation:

- **assert_present** - converts variable to a string and checks that it is not `empty?`
    - *assert_present :name*
- **assert_numeric** - checks that variable is a positive or negative number (non-decimal)
    - *assert_numeric :year*
- **assert_decimal** - checks that variable is a positive or negative decimal
    - *assert_numeric :fee*
- **assert_url** - checks that variable is a valid http or https URL
    - *assert_url :return_url*
- **assert_email** - checks that variable follows valid e-mail format
    - *assert_email :email*
- **assert_member** - checks that variable is a member of an array
    - *assert_member :status, ['active','pending']
- **assert_length** - checks that length of variable converted to string is within the range length
    - *assert_length :password, 8..16
- **assert_equal** - checks that variable is equal to a value by `===` operator
    - *assert_equal :auth_level, 5*
- **assert** - generic validation, provide your own conditional which will return true or false. For second parameter pass a 2 item array with the first item being the column being tested and the 2nd item being a string for the error or a symbol of pre-set errors (such as `:not_valid`)
    - *assert votes.to_i > 0, [:votes, :not_valid]*
