# DevCamp Ruby on Rails Notes 11
## Section 9: Controllers
### 0. Interesting Bits: HashWithIndifferentAccess
Understanding [HashWithIndifferentAccess in Rails](https://stackoverflow.com/questions/11381408/ruby-symbol-as-key-but-cant-get-value-from-hash/11384424#11384424):

> You are probably used to working within Rails where a very specific variant of Hash, called HashWithIndifferentAccess, is used for things like params. This particular class works like a standard ruby Hash, except when you access keys you are allowed to use either Symbols or Strings. The standard Ruby Hash, and generally speaking, Hash implementations in other languages, expect that to access an element, the key used for later access should be an object of the same class and value as the key used to store the object. HashWithIndifferentAccess is a Rails convenience class provided via the Active Support libraries. You are free to use them yourself, but they have first be brought in by requiring them.

Examples:
```ruby
$ irb
2.6.5 :001 > require 'active_support/core_ext'
 => true
2.6.5 :002 > h1 = {'name' => 'nahua', 'country' => 'germany'}
 => {"name"=>"nahua", "country"=>"germany"}
2.6.5 :003 > h1['name']
 => "nahua"
2.6.5 :004 > h1[:name]
 => nil
2.6.5 :005 > h2 = h1.with_indifferent_access
 => {"name"=>"nahua", "country"=>"germany"}
2.6.5 :006 > h2[:name]
 => "nahua"
2.6.5 :007 > h2['name']
 => "nahua"
2.6.5 :008 > h3 = {name: 'nahua', country: 'germany'}
 => {:name=>"nahua", :country=>"germany"}
2.6.5 :009 > h3[:name]
 => "nahua"
2.6.5 :010 > h3["name"]
 => nil
2.6.5 :011 >
```

### 1. Data Flow Review & Params
Create a new branch
```
$ git checkout master
$ git branch
$ git checkout -b controller
```
Go into server:
```ruby
$ rails s
```
Hit [http://127.0.0.1:3000/portfolio/4](http://127.0.0.1:3000/portfolio/4) and check the output in the console:
```ruby
Started GET "/portfolio/4" for 127.0.0.1 at 2020-01-26 10:38:06 +0100
Processing by PortfoliosController#show as HTML
  Parameters: {"id"=>"4"}
  Portfolio Load (0.4ms)  SELECT "portfolios".* FROM "portfolios" WHERE "portfolios"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
  ↳ app/controllers/portfolios_controller.rb:44:in `show'
  Rendering portfolios/show.html.erb within layouts/application
  Technology Load (0.5ms)  SELECT "technologies".* FROM "technologies" WHERE "technologies"."portfolio_id" = $1  [["portfolio_id", 4]]
  ↳ app/views/portfolios/show.html.erb:11
  Rendered portfolios/show.html.erb within layouts/application (Duration: 26.6ms | Allocations: 4263)
[Webpacker] Everything's up-to-date. Nothing to do
  User Load (0.4ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 ORDER BY "users"."id" ASC LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/views/layouts/application.html.erb:16
Completed 200 OK in 46ms (Views: 38.4ms | ActiveRecord: 2.5ms | Allocations: 10112)
```
Once Rails gets the `"/portfolio/4" `, it is sent to be processed by `PortfoliosController#show` as `HTML` (the only option we have). If it were an API call, we'd see `JSON` or `XML` instead of `HTML`.

Take a look at  the hash `Parameters: {"id"=>"4"}`. This is `params` in Rails and how we use `@portfolio_item = Portfolio.find(params[:id])` to select `:id`, which will be `4`.

Rails router finds a route that matches the URL we put into the browser, and it stores the URL pattern into a hash of `params`. From there on, the controller comes to the stage and takes over the action, such as with `PortfolioController#show` action.

If we now add to `app/views/portfolios/show.html.erb`:
```html
...html code...
<hr>
<%= params.inspect %>
<hr>
<%= params.inspect %>
<hr>
<%= params[:action] %>
<hr>
<%= params[:controller] %>
<hr>
<%= params[:id] %>
<hr>
<%= Portfolio.find(params[:id]) %>
<hr>
<%= Portfolio.find(params[:id]).title %>
```

We will see the following on that specific portfolio page (`<hr>` ignored):
```
<ActionController::Parameters {"controller"=>"portfolios", "action"=>"show", "id"=>"4"} permitted: false>
show
portfolios
4
#<Portfolio:0x00007fe25407a380>
Portfolio title: 3
```
There is this underlying data that we have access to as we move from page to page in Rails app.

So as we can see, `params` is the link between the `routes` and the `controller` for data flow.

### 2. Use Rails Session to Share Data Between Pages
We want to create dynamic routes to share our portfolio on LinkedIn or Twitter and show the visitors that we know where they come from.

Examples of such a link we have in mind:
```
http://127.0.0.1:3000/portfolio/4?q=linkedin
http://127.0.0.1:3000/portfolio/4?q=twitter
http://127.0.0.1:3000/portfolio/4?q=facebook
```
And we want the social media banner follow where they come from, even after `?q=<source>` is gone from the URL.

Let's go back to the parent controller `application_controller.rb`, so everything done in this controller can be accessed by all other controllers in our project:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist

  before_action :set_source			# <- ADD THIS

  def set_source					# <- ADD THIS   
    session[:source] = params[:q] if params[:q]
  end
end
```
Let's go back into our `app/views/portfolios/show.html.erb` to append:
```html
...html code...
<hr>
<%= session.inspect %>
```
Refresh, we'll see an extremely long (*seriously extreme*) text started with:
```
#<ActionDispatch::Request::Session:0x00007fe252d7c6e8 @by=#<ActionDispatch::Session::CookieStore:0x00007fe252ac43b0 @app=#<ActionDispatch::ContentSecurityPolicy::Middleware:0x00007fe252ac43d8 @app=#<Rack::Head:0x00007fe252ac4400...
```

Sessions are also underlying our application as to how the application communicates with the browser. ***All it does is passing data between the server and the browser.*** 

Btw, we see `_csrf_token` in the text. We can see it has `"QUERY_STRING"=>""` among the session text. All is stored in `cookie` format on the browser.

**Open up incognito mode (because it gives us a new session)**, type in `http://127.0.0.1:3000/portfolio/4?q=twitter`. We'll see:
```
"QUERY_STRING"=>"q=twitter", ...
"REQUEST_URI"=>"/portfolio/4?q=twitter", ...
"ORIGINAL_FULLPATH"=>"/portfolio/4?q=twitter", ...
@filtered_parameters={"q"=>"twitter", "controller"=>"portfolios", "action"=>"show", "id"=>"4"}, ...
```

Now inside `app/views/portfolios/show.html.erb` let's delete session and do `params.inspect`:
```html
...html code...
<hr>
<%= params.inspect %>
```
We'll see:
```ruby
<ActionController::Parameters {"q"=>"twitter", "controller"=>"portfolios", "action"=>"show", "id"=>"4"} permitted: false>
```
By doing `?q=twitter` we told Rails that we'll pass in some other data so please grab them. And then let's set `session[:source]` to `params[:q]` so we'll have access to this on all our application pages.

Indeed it's in session: 
```
@delegate={"session_id"=>"f2ab48f6408cdfe6322f08bfcbb0069a", "source"=>"twitter"}
```

Now let's go into `app/views/application.html.erb`:
```html
...html code...
    <%= yield %>

    <% if session[:source] %>
      Thanks for visiting me from <%= session[:source] %>
    <% end %>
...html code...
```
And it's there on the page [http://127.0.0.1:3000/portfolio/4?q=twitter](http://127.0.0.1:3000/portfolio/4?q=twitter) and afterwards [http://127.0.0.1:3000/portfolios](http://127.0.0.1:3000/portfolios), we'll see:
```
Thanks for visiting me from twitter
```

Imagine an e-commerce site where you want to keep track of shopping cart items. You can utilize `session` to do so. But `session` data, a note of warning, is not secure. **Don't put anything important into `session`. NO CREDIT CARD INFORMATION! REPEAT! DO NOT PUT ANYTHING IMPORTANT INTO SESSION** 

### 3. Refactor Session Tracker into Controller Concern
The reason that `set_source` method worked on all of our application pages is because it's done "`before_action`". All the controllers in our application inherits from the mother of all controllers, `ApplicationController`, and gets `set_source`'ed as well.

Now let's refactor and create a `set_source.rb` module inside `app/controllers/concerns/` (and mimic `devise_whitelist.rb`):
```ruby
# app/controllers/concerns/set_source.rb
module SetSource
  extend ActiveSupport::Concern

  included do
    before_action :set_source
  end

  def set_source
    session[:source] = params[:q] if params[:q]
  end
end
```
Inside `application_controller.rb`, we'll do:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
  include SetSource
end
```
It works again!

### 4. Work with Strong Params in Rails
We've been working with strong params since we built our first `scaffold`. Just check out the last line of `BlogsController`:
```ruby
# app/controllers/blogs_controller.rb
...ruby code...
    # Never trust parameters from the scary internet, only allow the white list through.
    def blog_params
      params.require(:blog).permit(:title, :body)
    end
...ruby code...
```
`.permit` says what we permit inside our `forms`. A hacker who passes other elements to the `form` might throw malicious code on our server. So whitelisting is a good idea.

Let's take a look at `config/application.rb`:
```ruby
# config/application.rb
...ruby code
module DevcampPortfolio
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.

    # Don't generate system test files.
    config.generators.system_tests = nil
  end
end
```
Now, if you put in `config.action_controller.permit_all_parameters = true`, then it'll overwrite all the other `.permit` we've done. Rarely do we do this (maybe in the admin site only for special reasons)!

Anyways, let's refactor our `portfolios_controller.rb` to have `blogs_controller.rb`'s setup.
```ruby
# app/controllers/portfolios_controller.rb
  def create
    @portfolio_item = Portfolio.new(portfolio_params) 	# <- CALL HERE
...ruby code...
  def update
    @portfolio_item = Portfolio.find(params[:id])

    respond_to do |format|
      if @portfolio_item.update(portfolio_params) 		# <- CALL HERE
...ruby code...
  private 												# <- DECLARE METHOD HERE
    def portfolio_params
      params.require(:portfolio).permit(:title,
                                        :subtitle,
                                        :body,
                                        technologies_attributes: [:name]
                                        )
    end
...ruby code...
```
Check everything works with server on. Now it's DRY and we only make changes at one spot in this code.

### 5. Deep Dive: Guest User Feature with Session
Right now, we have some not-so-good code inside:

 1. `app/views/pages/home.html.erb` (insecure code `current_user.first_name if current_user`, which is defensive)
 2. `app/views/application.html.erb` (When not logged in, displays `Hi, `)

In contrast, this below is not defensive:
```ruby
# app/views/application.html.erb
    <% if current_user %>
      <%= link_to "Logout", destroy_user_session_path, method: :delete %>
    <% else %>
      <%= link_to "Register", new_user_registration_path %>
      <%= link_to "Login", new_user_session_path %>
    <% end %>
```

So let's fix the issues. Before starting, try install:
```
$ gem install ostruct
```

How does `ostruct` work? See below (`require 'ostruct' returns false but still works`):
```ruby
$ pry
[1] pry(main)> require 'ostruct'
=> false
[2] pry(main)> guest = OpenStruct.new(name: "Nahua Kang", first_name: "Nahua", last_name: "Kang", email: "asdf@asdf.com")
=> #<OpenStruct name="Jordan Hudgens", first_name="Jordan", last_name="Hudgens", email="asdf@asdf.com">
[3] pry(main)> guest.name
=> "Jordan Hudgens"
[4] pry(main)> guest.email
=> "asdf@asdf.com"
```
So we use `ostruct` to mimic a real user.

First, we want to overwrite `devise`'s `current_user` method so that we can use it even when no user is logged in:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
  include SetSource

  # overwrite current_user method provided by devise
  def current_user
    # super: we want the exact behavior provided by devise
    # super || ostruct is literally like false || true => (always) true
    # basically, if super is true, ostruct is never used; otherwise, ostruct is used
    super || OpenStruct.new(name: "Guest User", first_name: "Guest", last_name: "User", email: "guest@example.com")
  end
end
```
Run `rails s` to check everything works. Ok, then we can get rid of our defensive code in `app/views/pages/home.html.erb`:
```html
<h1>Hi, <%= current_user.first_name %></h1>
<p>Find me in app/views/pages/home.html.erb</p>
...html code...
```
Hit refresh, still works. Update application file to make sure `current_user` does not always work if we are using `ostruct` user. We can differentiate class type between a real user (`User`) and a fake guest user (`OpenStruct`).
```ruby
$ pry
[1] pry(main)> require 'ostruct'
=> false
[2] pry(main)> guest = OpenStruct.new(name: "Jordan Hudgens", first_name: "Jordan", last_name: "Hudgens", email: "asdf@asdf.com")
=> #<OpenStruct name="Jordan Hudgens", first_name="Jordan", last_name="Hudgens", email="asdf@asdf.com">
[3] pry(main)> guest.class
=> OpenStruct
[4] pry(main)> guest.is_a?(OpenStruct)
=> true
```
So make the tiny change again here in `application.html.erb`:
```ruby
# app/views/layouts/application.html.erb
...html code...
    <% if current_user.is_a?(User) %>
      <%= link_to "Logout", destroy_user_session_path, method: :delete %>
    <% else %>
      <%= link_to "Register", new_user_registration_path %>
      <%= link_to "Login", new_user_session_path %>
    <% end %>
...html code...
```
Now it should all work fine. If we have not logged in, it will show guest user but display login and register, if we are logged in, it will display user name and logout.

Finally, let's build our concern for `ApplicationController`. We'll call this concern `CurrentUserConcern` for a file inside `app/controllers/concerns/current_user_concern.rb`:
```ruby
# app/controllers/concerns/current_user_concern.rb
module CurrentUserConcern
  extend ActiveSupport::Concern

    # overwrite current_user method provided by devise
  def current_user
    # super: we want the exact behavior provided by devise
    # super || ostruct is literally like false || true => (always) true
    # basically, if super is true, ostruct is never used; otherwise, ostruct is used
    super || OpenStruct.new(name: "Guest User", first_name: "Guest", last_name: "User", email: "guest@example.com")
  end
end
```
And refactor `application_controller.rb`:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
  include SetSource
  include CurrentUserConcern # <- DO THIS
end
```
Finally, redo `app/controllers/concerns/current_user_concern.rb`:
```ruby
# app/controllers/concerns/current_user_concern.rb
module CurrentUserConcern
  extend ActiveSupport::Concern

  def current_user
    super || guest_user
  end

  def guest_user
    OpenStruct.new(name: "Guest User",
                  first_name: "Guest",
                  last_name: "User",
                  email: "guest@example.com"
                  )
  end
end
```

Don't forget to `git push` and merge on Github before checking out master on your terminal and hit  `git pull`.








<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg5NDYxOTcxOSwxNTc1MjIwODIyLDE3MD
M4MDczNyw2NzU1Nzc0NTEsMTI1Mjc3NDY3MSwtMTIyODc0NDY0
MCwzOTU4MjMyMzksMTEwMzg2MDc5NywtOTYxNzEzMzk1LDIwND
k2OTk5MzcsLTU2NDYyOTM5MCw4ODYzMzczMTEsLTUzMDg1MjAx
NCwtMTQ2MjM3NjI5NywxMTkzMDA3NDI4LC0xMjgwNTIxNDEzLD
E2NjcwOTQ3MTddfQ==
-->