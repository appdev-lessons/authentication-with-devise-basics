# Authentication with Devise: The Basics

The [Devise gem](https://github.com/plataformatec/devise) is probably the most popular authentication library in the Rails ecosystem.

## Add sign-in/sign-out

- Add `gem "devise"` to your Gemfile and `bundle install`. (We included the gem in most of our projects, so you shouldn't need to do this step.)
- Run `rails generate devise:install`
- Since we haven't learned how to implement DELETE routes in Rails yet, we need to change part of the configuration in the `config/initializers/devise.rb` file from `:delete` to `:get`:

```ruby
# config/initializers/devise.rb, Line 269

config.sign_out_via = :get
```

Devise will give you some setup instructions. We don't need to worry about most of them, but we do need to set a root URL. Usually, you will point the root URL to the index action of some important resource in your application: In `config/routes.rb`:

```ruby
root "photos#index"
```

This is just a shortcut for setting a root URL the old way,

```ruby
get "/", :controller => "photos", :action => "index"
```

Next, we need to secure one of our models with Devise.

### Generate a new model with Devise

Use the following command to generate a User model with Devise built-in. Replace the column names with ones that are relevant to your application. 

<aside markdown="1">
Recall `rails g` is shorthand for `rails generate`.
</aside>

```
rails g devise user username:string avatar_url:string
```

Examine the generated migration file and make any desired changes (i.e., default values). Now,

```
rake db:migrate
```

and restart your server.

## Benefits

The big benefits that we now get for free from Devise are:

### RCAVs

RCAVs that handle sign up, sign in, and sign out -- all done, for free!
  
Visit `/rails/info` and search for "user" to see the routes that were automatically written. It's up to use to link to these routes in our UI wherever we think it appropriate; e.g. in the navbar.

The only routes you need to know for now are:

- `GET /users/sign_up`
- `GET /users/sign_in`
- `GET /users/sign_out`
- `GET /users/edit`

<aside markdown="1">

Note that in the first steps for setting up Devise, we had to change the `config/initializers/devise.rb` file to make sign-out at `/users/sign_out` happen via a GET request rather than DELETE, since we haven't learned about DELETE requests yet.
</aside>

### `current_user`

The `current_user` helper method, available within all views and controllers, that will retrieve the row from the Users table for whoever is currently signed in, so we can do things like:

```erb{1:(7-27)}
<% if current_user.present? %>
  <a href="/users/edit">Edit profile</a>

  <a href="/users/sign_out">Sign out</a>
<% else %>
  <a href="/users/sign_up">Sign up</a>
  
  <a href="/users/sign_in">Sign in</a>
<% end %>
```

and:

```erb{1:(43-72)}
<input type="hidden" name="query_user_id" value="<%= current_user.id %>">
```

### `authenticate_user!`

The `before_action :authenticate_user!` filter that we can use in our controllers to ensure someone is signed in before accessing any actions in that controller.

You can add this line to your `ApplicationController` to force visitors to sign in before visiting _any_ pages:

```ruby{5}
# app/controller/application_controller.rb

class ApplicationController < ActionController::Base
  skip_forgery_protection
  before_action :authenticate_user!
end
```

If you want to skip this action on any of your controller actions that are inheriting from `ApplicationController`, you can add this to that specific controller and action (the `:only =>` specifies the action that will be skipped):

```ruby
class UsersController < ApplicationController
  skip_before_action(:authenticate_user!, { :only => [:index] })
  # ...
```

There are lots more (password reset emails, etc), but these are the first ones that we care about.

## Customizing Devise Views

<div class="bg-blue-100 py-1 px-5" markdown="1">

There are a number of reasons to get your hands on the Devise view templates. Perhaps the most relevant, is if your `User` model has attributes that you want to edit besides the email and password. For instance, if you generated it with something like:

```
rails g devise user username:string avatar_url:string
```

and want to be able to create and update the `username` and `avatar_url` attributes.
</div>

Devise does an incredible amount of work for us out of the box, but at some point, we will want to customize it; at the very least, we will want to make the sign-up and sign-in forms look nicer.

Assuming we are using `before_action :authenticate_user!` in our `ApplicationController` to force visitors to sign up or sign in before doing anything else, then the sign in page is our landing page, after all (try going to Twitter, Facebook, etc when signed out -- the landing page is really a sign-in page with some extra info thrown in). So it would be nice to be able to make it pretty.

More importantly, we will likely want to allow users to provide more information when they sign up or edit their profiles than simply their email addresses and passwords.

### Step One: Get Our Hands On The View Templates

The first problem is that we don't even have any code to edit in order to customize the view templates for sign-in, sign-up, edit profile, etc. That's because, by default, these templates are wrapped up inside the Devise gem.

It's really easy to have Devise generate copies of these templates that we can edit, though. And then our edited versions will take precedence. At your bash prompt, simply run:

```
rails g devise:views -b form_for
```

(Don't leave off `-b form_for` from that command, or your view templates will look different than in this lesson.)

There will now be a folder in your `app/views` folder called `devise`, with a whole bunch of stuff in it. What we are most interested in are the contents of the `registrations` and `sessions` subfolders:

- `registrations/new.html.erb` is the sign up form, 
- `registrations/edit.html.erb` is the edit profile form, and 
- `sessions/new.html.erb` is the sign in form.

### Step Two: Modify The Markup

Devise is using some helper methods we haven't yet learned about to draw the `<form>` and `<input>` elements:

```erb
<!-- app/views/devise/registrations/new.html.erb -->

<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <div class="field">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> characters minimum)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "new-password" %>
  </div>
```

You don't need to worry about understanding _how_ this `form_for` form helper method works; that will come later, if you continue with Rails. All you need to know know is:

- you can add custom styling in `<div class="">` around the embedded Ruby tags, and
- the `<%= f.label ...` and `<%= f.____field ...` lines allow you to add labels and input fields to the table.

#### Add HTML Around The Helpers

You can write whatever HTML you want *around* these helpers; for example, Devise has already wrapped the label/input pairs inside a `<div>`:

```erb{4,7}
<!-- app/views/devise/registrations/new.html.erb -->

<!-- ... -->
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>
  <!-- ... -->
```

Peruse the [Bootstrap conventions for form markup](https://getbootstrap.com/docs/5.3/forms/overview/) to get some inspiration for modifying these classes.

#### Add CSS Classes To The Helpers

You can also add CSS classes to the input tags, like so:

```erb{1:(67-90)}
<%= f.email_field :email, autofocus: true, autocomplete: "email", class: "my-custom-class" %>
```

#### Add Additional Input Helpers
    
There are various types of `____field` helpers; the most common are

 - `f.text_field`
 - `f.email_field`
 - `f.password_field`
 - `f.number_field`
 - `f.check_box`

You can add as many of these as you need for the additional columns you've included in your user model, e.g. on the sign-up form:

```erb{10-13}
<!-- app/views/devise/registrations/new.html.erb -->

<h2>Sign up</h2>
<!-- ... -->
  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "new-password" %>
  </div>

  <div class="field">
    <%= f.label :username %><br />
    <%= f.text_field :username, autofocus: true %>
  </div>
<!-- ... -->
```

and on the edit profile form:

```erb{14-17}
<!-- app/views/devise/registrations/edit.html.erb -->

<h2>Edit <%= resource_name.to_s.humanize %></h2>
<!-- ... -->
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
    <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
  <% end %>

  <div class="field">
    <%= f.label :username %><br />
    <%= f.text_field :username, autofocus: true %>
  </div>
<!-- ... -->
```

### Step Three: Allow Additional Parameters Through Security

The last step we need to take is to whitelist these additional attributes as things that we will allow users to modify about themselves. To do this, you need to go to your `ApplicationController` and add the following code:

```ruby{4-10}
class ApplicationController < ActionController::Base
  skip_forgery_protection
  before_action :authenticate_user!
  before_action :configure_permitted_parameters, if: :devise_controller?

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, :keys => [:username, :avatar_url])

    devise_parameter_sanitizer.permit(:account_update, :keys => [:avatar_url])
  end
end
```

You need to add the name of each column you want to be able to modify to the respective array of symbols. In the example, I have whitelisted both `:username` and `:avatar_url` to be modified upon sign-up, but only `:avatar_url` can be modified upon account update. You have to decide what makes sense in your app and modify the two forms accordingly:

- `app/views/devise/registrations/new.html.erb`
- `app/views/devise/registrations/edit.html.erb`

Then add to the `configure_permitted_parameters` method in `ApplicationController`.

---
