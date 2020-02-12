# DevCamp Ruby on Rails Notes 1
## Section 3: App Creation & Project Planning
### 1. Creating DevcampPortfolio
Whenever we start a new application, especially if the database is not the default `sqlite3`, we want to do `rails db:create && rails db:migrate`, or as below:
```
$ rails new DevcampPortfolio -T --database=postgresql  
$ rails db:create  
$ rails db:migrate  
```
Note that `rails` is just a coating for `rake`. We can always do `rake db:create && rake db:migrate` instead.

### 2. Scaffolding
**Scaffold** is the ability to generate multiple items at the same time with many attributes like "title" and "body":  
```
$ rails g scaffold Blog title:string body:text  
$ rails db:migrate  
$ rails s			# Go to http://127.0.0.1:3000/blogs  
```

After scaffolding, remember to  delete `app/assets/stylesheets/scaffold.scss` to avoid custom CSS being overwritten by `scaffold.scss`.

### 3. Quick look into Blogs Controller
Scaffold created the `app/controllers/blogs_controller.rb` and all the code inside. Note that we have on top of the controller file:
```ruby
class BlogsController < ApplicationController
  before_action :set_blog, only: [:show, :edit, :update, :destroy]
  ...
  def index
    ...
  def show
  ...
  def new
  ...
```
`before_action` is a method that includes a method for other methods. So basically, this line is telling Rails that before action, make sure the methods (or ***actions***) `show`, `edit`, `update`, and `destroy` all include a method called `set_blog`, which we can find at the bottom of `blogs_controller.rb`:
```ruby
...
  private
    # Use callbacks to share common setup or constraints between actions.
    def set_blog
      @blog = Blog.find(params[:id])
    end
```

The action `new`, for example, 
```ruby
  def new
    @blog = Blog.new
  end
```
maps to `app/views/new.html.erb`. It does not create a new blog, but gives us the ability to instantiate a `Blog object`, giving us a form that includes `title` and `body`. Then, the action `create` will take the form parameters (`blog_params`).
```ruby
  def create
    @blog = Blog.new(blog_params)
```
So when you hit the button `create blog`, `blog_params` will be sent so that HTML will be formatted and we will be redirected to the `show` action (`@blog`).

Just as `new` is linked to `create`, `edit` is linked to `update`. `edit` makes the form available and `update` is where we make the changes for a blog post happen.

Note that `blog_params` is a method that you can find at the bottom of `blogs_controller.rb`:
```ruby
    # Never trust parameters from the scary internet, only allow the white list through.
    def blog_params
      params.require(:blog).permit(:title, :body)
    end
```

`destroy` method is for deleting a record on our blog. In `app/views/index.html.erb` we see the line related to the action `destroy`:
```html
...
<td><%= link_to 'Destroy', blog, method: :delete, data: { confirm: 'Are you sure?' } %></td>
...
```

**NOTE: When you place an instance variable (`@blog`) in a controller method, the data inside the instance variable is made available to its corresponding view!!!**

### 4. Routes inside routes.rb & the resources keyword
If we examine `app/config/routes.rb`, we see:
```ruby
Rails.application.routes.draw do
  resources :blogs
end
```
How do we get all the routes out of `resources :blogs`? Try the following command:
```
$ rake routes
```
Well, `resources` is a special word, like `show`, `update`, `destroy`. So `resources` contains all the routes shown in the command `rake routes`:
```
Prefix 		Verb		URI Pattern		 	Controller#Action
blogs 		GET    		/blogs(.:format) 		blogs#index
		POST   		/blogs(.:format)  		blogs#create
new_blog 	GET		/blogs/new(.:format)		blogs#new
edit_blog 	GET    		/blogs/:id/edit(.:format)	blogs#edit
blog 		GET    		/blogs/:id(.:format)		blogs#show
		PATCH  		/blogs/:id(.:format)		blogs#update
		PUT    		/blogs/:id(.:format)		blogs#update
		DELETE 		/blogs/:id(.:format)		blogs#destroy
```

The `:id` in the URI pattern is saying that it's expecting an ID, like `/blogs/3/edit`.

### 5. File System in Rails
Ruby on Rails is ***convention over configuration***. So naming is very important, especially between files in `app/views` and `app/controllers`. For the default process to take place, **the controller action name needs to be the same as the view file name**.

We will rarely do anything in `app/bin`.

`app/config` has an `config/environment/` directory for `development`, `test`, and `production`. It has the `config/initializers/` for setting things, like `assets.rb` and `inflections.rb` (the reason for why we created `Blog` scaffold but it's called a `blogs_controller`). There's also `session_store.rb` for managing cookies.

There's `config/application.rb`, which is basically the `main` file for our project. It's the master file.

There's `config/secrets.yml` where we can put API keys or anything that we should access securely.

Finally, there's `database.yml`. This is where problems between `sqlite3` and `postgresql` issues happen.

Inside `app/db`, `db/seeds.rb` is for putting in fake data to play with.

### 6. Deep Dive: Application Generator
Get the help menu with:
```
$ rails -h
```
We see usage as `rails new APP_PATH [options]`. Read through to understand how we can write application generator commands.

For instance, `-m` gives us the ability to generate application with a template. `-d` or `--database=DATABASE` is what we did with `postgresql`.

Using `rails new APP -B` is very fast as it creates a base application without doing `bundle install` already (skip bundle).

`Puma` is a powerful web server shipped automatically with `Rails 5`. Can do `-P` to skip it. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc1MjM1MzMyN119
-->
