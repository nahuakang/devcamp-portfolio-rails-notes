# DevCamp Ruby on Rails Notes 9
## Section 8: User Authentication with Devise
### Install Gems
```
$ git pull
$ git status
$ git checkout master
$ git checkout -b authentication
```
Go to [https://rubygems.org/](https://rubygems.org/) and [search for `devise`](https://rubygems.org/search?utf8=%E2%9C%93&query=devise). 

Do the following inside `Gemfile`:
```ruby
# Add devise for Udemy lecture 88
gem 'devise', '~> 4.7', '>= 4.7.1'
```
Then update with:
```ruby
$ bundle install
```

### 1. Use Devise to Implement Registrations & Login

Read [devise's homepage](https://github.com/heartcombo/devise) for information.

We see that `devise` has its own generator so let's do it in the terminal:
```
$ rails generate devise:install
Running via Spring preloader in process 68781
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

First let's take a look at this file: `config/initializers/devise.rb` and make one single change:
```ruby
config.mailer_sender = 'support@nahua.com'
```
Add this to `development.rb`:
```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
Add these to `application.html.erb`:
```html
...html code...
  <body>
    <p class="notice"><%= notice %></p>
    <p class="alert"><%= alert %></p>
    <%= yield %>
  </body>
...html code...
```
Finally, run
```ruby
$ rails g devise:views
Running via Spring preloader in process 69324
      invoke  Devise::Generators::SharedViewsGenerator
      create    app/views/devise/shared
      create    app/views/devise/shared/_error_messages.html.erb
      create    app/views/devise/shared/_links.html.erb
      invoke  form_for
      create    app/views/devise/confirmations
      create    app/views/devise/confirmations/new.html.erb
      create    app/views/devise/passwords
      create    app/views/devise/passwords/edit.html.erb
      create    app/views/devise/passwords/new.html.erb
      create    app/views/devise/registrations
      create    app/views/devise/registrations/edit.html.erb
      create    app/views/devise/registrations/new.html.erb
      create    app/views/devise/sessions
      create    app/views/devise/sessions/new.html.erb
      create    app/views/devise/unlocks
      create    app/views/devise/unlocks/new.html.erb
      invoke  erb
      create    app/views/devise/mailer
      create    app/views/devise/mailer/confirmation_instructions.html.erb
      create    app/views/devise/mailer/email_changed.html.erb
      create    app/views/devise/mailer/password_change.html.erb
      create    app/views/devise/mailer/reset_password_instructions.html.erb
      create    app/views/devise/mailer/unlock_instructions.html.erb
```

We see many `views/devise/` folders created. We'll first deal with `views/devise/registrations` (sign-up to your application) and `views/devise/sessions` (sign-in to your application).

Next, we'll do (NOTE: Don't use "`MODEL`"):
```ruby
$ rails generate devise User
Running via Spring preloader in process 69398
      invoke  active_record
      create    db/migrate/20200125180553_devise_create_users.rb
      create    app/models/user.rb
      insert    app/models/user.rb
       route  devise_for :users
```
There's a migration file and model file.

Inside `config/routes.rb`, we see a new route:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  devise_for :users # THIS ENCAPSULATED THE CREATION ALL KINDS OF ROUTES
  ...other routes...
end
```
Inside `app/models/user.rb`, we see:
```ruby
# app/models/user.rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```
We won't touch the other modules such as mentioned in the comment.

And there's the `[timestamp]_devise_create_users.rb` file, to which we'll add `t.string :name`:
```ruby
# frozen_string_literal: true
class DeviseCreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Custom fields for Udemy course 	# <- ADD THIS
      t.string :name 						# <- ADD THIS

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at
      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```
Do:
```ruby
$ rails db:migrate
$ rails s
```
Go to [http://127.0.0.1:3000/users/sign_up](http://127.0.0.1:3000/users/sign_up) and try sign up with:
```
Email: test@test.com
Password: test123456
```

### 2. Customize Devise Routes
We might not like the path: `localhost:3000/users/sign_up` or `localhost:3000/users/sign_in`. 

Let's go into `config/routes.rb` and customize `sign_up`, `sign_in`, `sign_out`:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  devise_for :users, path: '', path_names: { sign_in: 'login', sign_out: 'logout', sign_up: 'register' }
```

Now visit:
 1. [http://127.0.0.1:3000/register](http://127.0.0.1:3000/register)
 2. [http://127.0.0.1:3000/login](http://127.0.0.1:3000/login)

And they work with just a single line of code.

### 3. Customize Logout Button
So far, we don't have `logout` action yet. So what's the route to `logout`?
```
$ rake routes | grep logout
destroy_user_session 	DELETE 		/logout(.:format) 	devise/sessions#destroy
```
Now we know it's `destroy_user_session`. So the path to this action will be: `destroy_user_session_path`.

And let's take a look at the routes for `register` and `login`:
```
$ rake routes | grep login
new_user_session 		GET    /login(.:format)  	devise/sessions#new
user_session 			POST   /login(.:format)		devise/sessions#create
$ rake routes | grep register
new_user_registration 	GET    /register(.:format)	devise/registrations#new
```

We don't have to change the routes. But we should add `logout` button to `application.html.erb`:
```html
...html code...
  <body>
    <p class="notice"><%= notice %></p>
    <p class="alert"><%= alert %></p>

    <% if current_user %>
      <%= link_to "Logout", destroy_user_session_path, method: :delete %>
    <% else %>
      <%= link_to "Register", new_user_registration_path %>
      <%= link_to "Login", new_user_session_path %>
    <% end %>

    <%= yield %>
  </body>
...html code...
```

### 4. Add Customize Attributes
We want the ability for users to have a `name` instead of going just by `email`, and if we check `schema.rb`, we'll see that under the table `users`, we do have a `name` attribute:
```ruby
# db/schema.rb
...Rails code...
  create_table "users",...
    ...
    t.string "name"
    ...
...Rails code...
```
Let's go to `app/views/devise/registration` to find `edit.html.erb` and `new.html.erb`. Inside `edit.html.erb`, copy one of the `.field`:
```html
...html code...
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <div class="field">      # <- ADD THIS BY EDITING THE ONE ABOVE
    <%= f.label :name %><br />
    <%= f.text_field :name %>
  </div>
...html code...
```

Now, inside `app/views/devise/registration/new.html.erb`:
```html
...html code...
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <div class="field">      # <- PASTE IN THIS
    <%= f.label :name %><br />
    <%= f.text_field :name %>
  </div>
...html code...
```
Now on [http://127.0.0.1:3000/register](http://127.0.0.1:3000/register), we will see the option of `Name`. It won't work yet by itself alone. This is because the SQL in the backend does not yet process the `name` yet when it creates a new `User`!

We see "`Unpermitted parameter: name`" error.  We must tell Rails that the `params` need to `permit` certain attributes. `Devise` controller is not accessible directly to us. We can't find it under `app/controllers/`.

So how do we fix it? Inside `application_controller.rb`, which is the master controller for the entire app.
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # NOTE: DO NOT USE before_filter!!! Won't work.
  before_action :configure_permitted_parameters, if: :devise_controller?

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
    devise_parameter_sanitizer.permit(:account_update, keys: [:name])
  end
end
```
Now `name` would work for `register` and `edit`.


### 5. Use Controller Concerns for Customize Attributes
We should follow the single responsibility rule and make sure the whitelist parameters are not directly under `ApplicationController`. We should use controller concerns to isolate the whitelist parameters.

Under `app/controllers/concerns/` create a new file `devise_whitelist.rb` (for helper code, we do `module` instead of `class`). We have to call the `module` as `DeviseWhitelist` in *CamelCase* because that's how we named the file itself (Rails parsing engine will map it this way):
```ruby
# app/controllers/concerns/devise_whitelist.rb
module DeviseWhitelist
  extend ActiveSupport::Concern

  included do
    before_action :configure_permitted_parameters, if: :devise_controller?
  end

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
    devise_parameter_sanitizer.permit(:account_update, keys: [:name])
  end
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
end
```

Now we've organized our code in a much better fashion. The only constant in web development is change, so organizing code with best practices is important!







<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkzODgzNzg0OSwxNzMzOTM3ODc5LDUzNT
MyNDA0NiwtMTczODQ2Nzk0NCw0NTM2MjQxMDMsMTkxNzk2OTE3
OSwtNTIyNDQzMjA0LC01MTk2NDIwODgsLTYyODU3MzU5MCwtOT
Q1NzU3ODUsLTE2NDc1NDYwOTBdfQ==
-->