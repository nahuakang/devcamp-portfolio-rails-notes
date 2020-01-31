# DevCamp Ruby on Rails Notes 16
## Section 13: Authorization in Rails
**Authorization is not the same as authentication!** It's about permission structure, i.e. what kind of activities a user is authorized to do. For example, you don't want any user to login and start pushing blogs out.

### 1. Install Petergate Gem
Take a look at the [Petergate Gem](https://github.com/elorest/petergate).

Here's a take on how to generate gem installs: [https://github.com/elorest/petergate/blob/master/lib/generators/petergate/install_generator.rb](https://github.com/elorest/petergate/blob/master/lib/generators/petergate/install_generator.rb).

Do
```
$ git checkout -b authorization
```

Add the following line into `Gemfile` so that we specify the version to install:
```ruby
gem 'petergate', '~> 2.0', '>= 2.0.1'
```
Then run:
```
$ bundle install
```

### 2. Pre-Configure Petergate Gem: Refactor
Go through the README on Petergate gem Github page, specifically:

"If you're using  [devise](https://github.com/plataformatec/devise)  you're in luck, otherwise you'll have to add following methods to your project:
```
current_user
after_sign_in_path_for(current_user)
authenticate_user!
```
Run the generators
```
rails g petergate:install
rake db:migrate
```
This will add a migration and insert petergate into your User model."

Let's do it:
```
$ rails g petergate:install
      insert  app/models/user.rb
      create  db/migrate/20200130153905_add_roles_to_users.rb
```
New lines added by PeterGate below:
```ruby
# app/models/user.rb
class User < ApplicationRecord
  ############################################################################################
  ## PeterGate Roles                                                                        ##
  ## The :user role is added by default and shouldn't be included in this list.             ##
  ## The :root_admin can access any page regardless of access settings. Use with caution!   ##
  ## The multiple option can be set to true if you need users to have multiple roles.       ##
  petergate(roles: [:admin, :editor], multiple: false)                                      ##
  ############################################################################################ 
```
Now run
```
$ rails db:migrate
```
Double check in `schema.rb` to see if `users` have `roles`.

### 3. Implement Authorization to Rails App

Time to refactor our `current_user_concern.rb` so that there's no buggy experience with PeterGate:
```ruby
# app/models/controllers/current_user_concern.rb
module CurrentUserConcern
  extend ActiveSupport::Concern

    # overwrite current_user method provided by devise
  def current_user
    # super: we want the exact behavior provided by devise
    # super || ostruct is literally like false || true => (always) true
    # basically, if super is true, ostruct is never used; otherwise, ostruct is used
    super || guest_user
  end

  def guest_user
	# <- DELETE ALL OF THIS    
  end
end
```

Create `app/models/guest_user.rb`:
```ruby
# app/models/guest_user.rb
class GuestUser < User
  # Create some getters and setters
  attr_accessor :name, :first_name, :last_name, :email
end
```
Back in the `current_user_concern.rb`:
```ruby
# app/models/controllers/current_user_concern.rb
module CurrentUserConcern
  extend ActiveSupport::Concern

    # overwrite current_user method provided by devise
  def current_user
    # super: we want the exact behavior provided by devise
    # super || ostruct is literally like false || true => (always) true
    # basically, if super is true, ostruct is never used; otherwise, ostruct is used
    super || guest_user
  end

  def guest_user
    guest = GuestUser.new
    guest.name = "Guest User"
    guest.first_name = "Guest"
    guest.last_name = "User"
    guest.email = "guest@example.com"
    guest # return guest
  end
end
```
Now go to server and check if we log out a user, the "Guest" appears or not.

Back in `application_helper.rb`, we have an issue now:
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def login_helper
    if current_user.is_a?(User) # <- THIS IS AN ISSUE AS GuestUser inherits User
      link_to "Logout", destroy_user_session_path, method: :delete
    else
      (link_to "Register", new_user_registration_path) +
      "<br>".html_safe +
      (link_to "Login", new_user_session_path)
    end
  end
  ...ruby code...
end
```
Fix it by:
```ruby
module ApplicationHelper
  def login_helper
    if current_user.is_a?(GuestUser) # <- Fix here down
      (link_to "Register", new_user_registration_path) +
      "<br>".html_safe +
      (link_to "Login", new_user_session_path)
    else
      link_to "Logout", destroy_user_session_path, method: :delete
    end
  end
  ...ruby code...
end
```

### 3. Configure Petergate Gem

Note `user` role is by default included by PeterGate. So let's just use one `site_admin` for now:
```ruby
# app/models/user.rb
class User < ApplicationRecord
  ############################################################################################
  ## PeterGate Roles                                                                        ##
  ## The :user role is added by default and shouldn't be included in this list.             ##
  ## The :root_admin can access any page regardless of access settings. Use with caution!   ##
  ## The multiple option can be set to true if you need users to have multiple roles.       ##
  petergate(roles: [:site_admin], multiple: false)                                      ##
  ############################################################################################ 
 
```
Check PeterGate for the Controllers section and do the following. Go to the `BlogController`:
```ruby
# app/controllers/blogs_controller.rb
class BlogsController < ApplicationController
  before_action :set_blog, only: [:show, :edit, :update, :destroy, :toggle_status]
  layout "blog"
  # ADD THIS BELOW
  access all: [:show, :index], user: {except: [:destroy, :new, :create, :update, :edit]}, site_admin: :all
  ...ruby code...
end
```
Ok, inside the console let's edit one user to be site admin:
```ruby
$ rails c
2.6.5 :016 > User.first.update!(name: "Test Admin", roles: "site_admin")
  User Load (0.8ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
   (0.2ms)  BEGIN
  User Update (25.9ms)  UPDATE "users" SET "roles" = $1, "name" = $2, "updated_at" = $3 WHERE "users"."id" = $4  [["roles", "site_admin"], ["name", "Test Admin"], ["updated_at", "2020-01-30 16:20:03.294789"], ["id", 1]]
   (1.7ms)  COMMIT
 => true
```
Now back on the server and let's login with our test user that's a site-admin, we'll be able to see `http://127.0.0.1:3000/blogs/new`, but if we login with a normal user, we will see `Permission Denied`.

Next, let's try hide content on the `index.html.erb` for blogs (details on PeterGate already):
```html
...html code...
<%= link_to 'New Blog', new_blog_path if logged_in?(:site_admin) %>
```
This way, only site admin can see the link to create new blogs.

Do the same for our partial `_blog.html.erb`, specifically the edit link and the delete post link:
```html
...html code...
  <td><%= link_to 'Edit', edit_blog_path(blog) if logged_in?(:site_admin) %></td>
  <td><%= link_to 'Delete Post', blog, method: :delete, data: { confirm: 'Are you sure?' } if logged_in?(:site_admin) %></td>
</tr>
```
And also `app/views/blogs/show.html.erb`:
```html
...html code...
<%= link_to 'Edit', edit_blog_path(@blog) if logged_in?(:site_admin) %> |
<%= link_to 'Back', blogs_path %>
```

Now normal users won't see the edit and delete options on blogs.

Finally, let's not forget to do the same for portfolios:
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  before_action :set_portfolio_item, only: [:edit, :update, :show, :destroy]
  layout "portfolio"
  # ADD THIS BELOW
  access all: [:show, :index, :vuejs], user: {except: [:destroy, :new, :create, :update, :edit]}, site_admin: :all
  ...ruby code...
end
```
Now if the normal user tries to access: `http://127.0.0.1:3000/portfolios/new`, there will be a notice `Permission Denied`. 

And we will clean up:

 - `_portfolio_item.html.erb`
 - `app/views/portfolios/index.html.erb`
 - `vuejs.html.erb`

`app/views/portfolios/index.html.erb`
```html
...html code...
<%= link_to "Create New Item", new_portfolio_path if logged_in?(:site_admin) %>
...html code...
```

`vuejs.html.erb`
```html
...html code...
<%= link_to "Create New Item", new_portfolio_path if logged_in?(:site_admin) %>
...html code...
	<%= link_to "Edit", edit_portfolio_path(portfolio_item) if logged_in?(:site_admin) %>
	<%= link_to "Delete Portfolio", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } if logged_in?(:site_admin) %>
...html code...
```
`app/views/portfolios/_portfolio_item.html.erb`
```html
...html code...
<%= link_to "Edit", edit_portfolio_path(portfolio_item) if logged_in?(:site_admin) %>
<%= link_to "Delete Portfolio", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } if logged_in?(:site_admin) %>
...html code...
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA4Nzk3NTg5NSwtMTgxMDkxMzMwMCwxNz
k2NTk4MTU2LDE2MjU5NzI3MTIsLTEyNjMwNzA4NzAsLTQxODU0
MzI1NF19
-->