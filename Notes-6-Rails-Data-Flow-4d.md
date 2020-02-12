# DevCamp Ruby on Rails Notes 6
## Section 6: Rails Data Flow - Deep Dive
### Interesting bits
***Rails Naming Conventions:*** [https://gist.github.com/iangreenleaf/b206d09c587e8fc6399e](https://gist.github.com/iangreenleaf/b206d09c587e8fc6399e)

### 1. Create RoutingApp
```ruby
$ rails new RoutingApp -T --database=postgresql
$ cd RoutingApp
$ rails db:create
$ rails g scaffold Blog title:string body:text
$ rails db:migrate
```

### 2. Generate Scaffold Blog
```ruby
$ rake routes
Prefix 		Verb   		URI Pattern	 		Controller#Action
blogs 		GET		/blogs(.:format)		blogs#index
		POST   		/blogs(.:format) 		blogs#create
new_blog 	GET    		/blogs/new(.:format)		blogs#new
edit_blog 	GET    		/blogs/:id/edit(.:format) 	blogs#edit
blog 		GET    		/blogs/:id(.:format) 		blogs#show
		PATCH  		/blogs/:id(.:format)		blogs#update
		PUT		/blogs/:id(.:format) 		blogs#update
		DELETE 		/blogs/:id(.:format)		blogs#destroy
```
Again, `Prefix` is the method we can use inside our code to give us full route path. `Verb` is about clarifying if we're getting information from server or sending information to the server. `URI Pattern` is the path for URL. `Controller#Action` we already know well now.


### 3. Generate Controller Pages
```ruby
$ rails g controller Pages about contact
$ rake routes | grep pages
pages_about 	GET    	/pages/about(.:format)	 pages#about
pages_contact 	GET    	/pages/contact(.:format) pages#contact
```

### 4. Customize Routes
Notice the problem of visiting `about` and `contact` via `/pages/about` and `pages/contact`. Let's change that.

Current `routes.rb`:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'pages/about'
  get 'pages/contact'
  resources :blogs
end
```

Change by `get 'URL_WE_WANT', to: 'CONTROLLER#ACTION'`
```ruby
Rails.application.routes.draw do
  get 'about', to: 'pages#about'
  get 'contact', to: 'pages#contact'
  resources :blogs
end
```
Run 
```ruby
$ rake routes | grep pages
about 	GET    /about(.:format)		pages#about
contact GET    /contact(.:format)	pages#contact
```

### 5. Customize Routes Further
We can literally put in any URL we want:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact'
  resources :blogs
end
```

Run:
```ruby
$ rake routes
$ rails s
```
Try: [http://127.0.0.1:3000/leadgen/advertising/landingpage/lead](http://127.0.0.1:3000/leadgen/advertising/landingpage/lead). It works.

But beware of the Prefix: `leadgen_advertising_landingpage_lead`.

Open `about.html.erb`, we'd have to type a long method to link to the `contact` page:
```html
<h1>Pages#about</h1>
<p>Find me in app/views/pages/about.html.erb</p>

<%= link_to "Lead Page",  leadgen_advertising_landingpage_lead_path %>
```

### 6. Customize Prefix Method
So let's customize the Prefix method in `routes.rb`:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'
  resources :blogs
end
```

Run:
```ruby
$ rake routes
lead 	GET 	/leadgen/advertising/landingpage/lead(.:format) pages#contact
```
Change:
```html
<h1>Pages#about</h1>
<p>Find me in app/views/pages/about.html.erb</p>

<%= link_to "Lead Page",  lead_path %>
```
And the code is much cleaner now.

### 7. Create Homepage
Open up the `pages_controller.rb`:
```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def about
  end

  def contact
  end
end
```
Add `home` action:
```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def about
  end

  def contact
  end

  def home
  end
end
```

Create home.html.erb in `app/views/pages/home.html.erb`:
```html
<h1>Homepage</h1>
```

Create the route in `config/routes.rb`:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs

  root to: 'pages#home' # <- Create this
end
```
Whenever we use `root`, we get a `root_path` that points to our homepage.

### 8. Create Nested Routes

Let's create another controller:
```ruby
$ rails g controller Dashboard main user blog
```

Now in `config/routes.rb`, we have three new items:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'dashboard/main' # <- NEW
  get 'dashboard/user' # <- NEW
  get 'dashboard/blog' # <- NEW
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'
  resources :blogs
  root to: 'pages#home'
end
```

***WHAT IF we want it to look like:*** 
`localhost:3000/admin/dashboard/main`?

#### A. NAMESPACING in ROUTES.RB
Well, we need to name space these routes then:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do 		# <- NAMESPACING
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs

  root to: 'pages#home'
end
```

#### B. MOVE CONTROLLER TO NAMESPACED FOLDER
***JUST THIS NAMESPACING alone won't do.*** We'd get `uninitialized constant Admin`. Rails is looking for an `Admin` with capitalized `A`.

Take a look at `rake routes`:
```ruby
$ rake routes | grep admin
	admin_dashboard_main GET 	/admin/dashboard/main(.:format) 	admin/dashboard#main
	admin_dashboard_user GET    	/admin/dashboard/user(.:format)		admin/dashboard#user
	admin_dashboard_blog GET    	/admin/dashboard/blog(.:format)		admin/dashboard#blog
```
`Controller#Action` here becomes nested, too:
```
admin/dashboard#main
admin/dashboard#user
admin/dashboard#blog
```
We have the correct routes, Prefix names are good, but the `Controller#Action` is problematic. 

**SOLUTION:** Fix it by first creating a folder named `admin` under `app/controllers`, then drag `dashboard_controllers.rb` into `app/controllers/admin/`.

#### C. NOTIFY CONTROLLER'S CLASS
Then, to perform nesting, the class name has to be informed:
```ruby
# app/controllers/admin/dashboard_controller.rb
class DashboardController < ApplicationController
```
**MUST BE CHANGED TO:**
```ruby
# app/controllers/admin/dashboard_controller.rb
class Admin::DashboardController < ApplicationController
```

#### D. MOVE VIEWS TO NAMESPACED FOLDER
FINALLY: update the views since otherwise we'd miss `views`. We need to move the `app/views/dashboard/` must be moved to `app/views/admin/dashboard/`. 

NOW try: [http://127.0.0.1:3000/admin/dashboard/main](http://127.0.0.1:3000/admin/dashboard/main). It works.

For moderately large projects, we will do ***nesting***. So we need to remember this flow for ***namespacing***.

### 9. How to build resources from scratch?
#### A. ADD RESOURCES TO ROUTES.RB
If we just add `resources :posts` to `config/routes.rb`
```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs
  resources :posts # <- ADD THIS RESOURCES

  root to: 'pages#home'
end
```
And run `rake routes`, we see we get all the routes defined:
```ruby
$ rake routes | grep posts
posts 		GET    	/posts(.:format)		posts#index
		POST   	/posts(.:format)		posts#create
new_post 	GET    	/posts/new(.:format)		posts#new
edit_post 	GET    	/posts/:id/edit(.:format)	posts#edit
post 		GET    	/posts/:id(.:format)		posts#show
		PATCH  	/posts/:id(.:format)		posts#update
		PUT    	/posts/:id(.:format)		posts#update
		DELETE 	/posts/:id(.:format)		posts#destroy
```

#### B. ADD CONTROLLER TO CONTROLLERS
Add `posts_controller.rb` to 	`app/controllers/`:
```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
  end
end
```
***The naming must be exactly the same!***

#### C.  ADD THE TEMPLATES
Add new directory: `app/views/posts/`. Since we only implemented `index` action in the controller, let's create `app/views/posts/index.html.erb`:
```html
<h1>Here are the posts</h1>
```
Visit [http://127.0.0.1:3000/posts](http://127.0.0.1:3000/posts) and it works.

### 10. Globbing
For anything after `posts/`

Let's add on to `resources :posts`:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs
  resources :posts

  get 'posts/*missing', to: 'posts#missing' # <- ADD THIS

  root to: 'pages#home'
end
```

In `posts_controller.rb`, add:
```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
  end

  def missing # <- ADD THIS
  end
end
```

Then create the `missing.html.erb`:
```ruby
<h1>These are not the posts you were looking for...</h1>
```

Now, going to [http://127.0.0.1:3000/posts/gibberish/abc](http://127.0.0.1:3000/posts/gibberish/abc) will show the `missing.html.erb` page. Going to [http://127.0.0.1:3000/posts/gibberish](http://127.0.0.1:3000/posts/gibberish) will be caught by `posts#show` action and throws an `The action 'show' could not be found for PostsController` error.

***Very important is the position of the `glob`.*** If our `resources :posts` has other methods, like `new`:
```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
  end

  def missing
  end

  def new
  end
end
```
And we create the `show.html.erb`:
```html
<h1>Make a new post here.</h1>
```

If we have:
```ruby
  get 'posts/*missing', to: 'posts#missing' # <- ADD THIS
  resources :posts
```
We won't see `show` anymore!!! ***Ruby routes start at the top and goes its way down. If it finds a route that matches, it ignores everything else.*** We might be able to catch gibberish routes with the `glob`, but we will also not be able to see any of the other actions we want.

### 11. Completely Custom & Dynamic Route
Often we want to create dynamic routes listing in the URL just the item's titles, etc. Open `config.routes.rb`:
```ruby
Rails.application.routes.draw do
  namespace :admin do
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs
  resources :posts

  get 'posts/*missing', to: 'posts#missing'

  get 'query/:else', to: 'pages#something' # <- ADD THIS, :else can be :id as well

  root to: 'pages#home'
end
```
Rails will treat `:something` as a dynamic value and pull in whatever the user types in. Now, in `pages#something`, the action does not necessarily need to match the `:something`. We can use any name for the dynamic part or the URL and for the action.

Add the `something` to `pages_controller.rb`:
```ruby
class PagesController < ApplicationController
  def about
  end

  def contact
  end

  def home
  end

  def something # <- ADD THIS
    @else = params[:else] # ADD THIS <- :else must match routes.rb 'query/:else'
  end
end
```
**Note that whatever we put down in `routes.rb` (i.e. `:else`) has to match whatever we write in `params`!**

Create `app/views/pages/something.html.erb`:
```html
<h1><%= @else %></h1>
```

Now we can get whatever we put into the route. Data flow is:
**router -> controller -> view**.

We can further extend this:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs
  resources :posts

  get 'posts/*missing', to: 'posts#missing'

  get 'query/:else/:another_one', to: 'pages#something' # :else can be :id as well

  root to: 'pages#home'
end
```
In `pages_controller`:
```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def about
  end

  def contact
  end

  def home
  end

  def something
    @else = params[:else]
    @another_one = params[:another_one] # <- ADD THIS
  end
end
```
In `something.html.erb`:
```html
<h1><%= @else %></h1>
<h2><%= @another_one %></h2>
```
Note since we have both `:else` and `:another_one`, URLs like `localhost:3000/query/hey` won't work anymore. To keep it, do:
```ruby
Rails.application.routes.draw do
  namespace :admin do
    get 'dashboard/main'
    get 'dashboard/user'
    get 'dashboard/blog'  
  end
  
  get 'about', to: 'pages#about'
  get 'leadgen/advertising/landingpage/lead', to: 'pages#contact', as: 'lead'

  resources :blogs
  resources :posts

  get 'posts/*missing', to: 'posts#missing'

  get 'query/:else/:another_one', to: 'pages#something' # :else can be :id as well
  get 'query/:else', to: 'pages#something' # ADD THIS TO KEEP BOTH

  root to: 'pages#home'
end
```
Now it'd all work.



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA0MzI0MTQ3N119
-->
