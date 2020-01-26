# DevCamp Ruby on Rails Notes 3
## Section 6: Rails Data Flow
### 1. Create Seed File for Sample Data
Back inside the terminal, let's checkout another branch:
```
$ git checkout -b portfolio-feature
```
Now, inside `DevcampPortfolio/db/seeds.rb`, we can create data:
```ruby
10.times do |blog|
  Blog.create!(
    title: "My Blog Post #{blog}",
    body: "Lorem ipsum dolor sit amet..."
  )
end

puts "10 blog posts created."

5.times do |skill|
  Skill.create!(
    title: "Rails #{skill}",
    percent_utilized: 15,
  )
end

puts "5 skills created"

# Use for image: https://placeholder.com/
9.times do |portfolio_item|
  Portfolio.create!(
    title: "Portfolio title: #{portfolio_item}",
    subtitle: "My great service",
    body: "Lorem ipsum dolor sit amet...",
    main_image: "http://placehold.it/600x400",
    thumb_image: "http://placehold.it/350x200" 
  )
end

puts "9 portfolio items created"
```

**We then do, wiping out all of our test data, creating new databases from scratch, and run our seeds file:**
```
$ rails db:setup
$ rails c
irb(main):001:0> Blog.count
   (7.7ms)  SELECT COUNT(*) FROM "blogs"
=> 10
irb(main):002:0> Skill.count
   (0.3ms)  SELECT COUNT(*) FROM "skills"
=> 5
irb(main):003:0> Portfolio.count
   (0.6ms)  SELECT COUNT(*) FROM "portfolios"
=> 9
irb(main):004:0> Portfolio.last
  Portfolio Load (0.2ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
=> #<Portfolio id: 9, title: "Portfolio title: 8", subtitle: "My great service", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-19 22:09:02", updated_at: "2020-01-19 22:09:02">
irb(main):005:0>
```
***Run this only in development mode!!! Don't do this in production!***

### 2. Portfolio Items
Building the `PortfoliosController`, we will end up with something that looks similar to the `BlogsController`. And that's fine since we want to learn how Rails works.
```
$ rails c
irb(main):002:0> Portfolio.last
  Portfolio Load (0.3ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
=> #<Portfolio id: 9, title: "Portfolio title: 8", subtitle: "My great service", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-19 22:09:02", updated_at: "2020-01-19 22:09:02">
irb(main):003:0>
```
#### A. index action
Now, let's first create an index action, which we use when we want to list all the database items. 
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  def index
    @portfolio_items = Portfolio.all
  end
end
```
Then we make it available to the `views`. For this, we need to create `app/views/portfolios/index.html.erb`, which has some erb lines (notice the `image_tag` which lets us display image thumbnails):
```html
<h1>Portfolio Items</h1>

<% @portfolio_items.each do |portfolio_item| %>
	<p><%= portfolio_item.title %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image %>
<% end %>
```
Obviously, the instance variable `@portfolio_items` from `index action` is available in the corresponding `index.html.erb`. Hit:
```
$ rails s
```
and visit [http://127.0.0.1:3000/portfolios](http://127.0.0.1:3000/portfolios) to check the result.

#### B. new & create actions
First, before creating `new` and `create`, let's take a look at the routes via:
```
$ rake routes | grep portfolio
Prefix		Verb		URI Pattern				Controller#Action
portfolios 	GET    		/portfolios(.:format)	 		portfolios#index
		POST   		/portfolios(.:format)	 		portfolios#create
new_portfolio 	GET    		/portfolios/new(.:format)		portfolios#new
edit_portfolio 	GET    		/portfolios/:id/edit(.:format)		portfolios#edit
portfolio 	GET    		/portfolios/:id(.:format)		portfolios#show
		PATCH  		/portfolios/:id(.:format)		portfolios#update
		PUT    		/portfolios/:id(.:format)		portfolios#update
		DELETE 		/portfolios/:id(.:format)		portfolios#destroy
```
`Prefix` is just our route methods, which we'll get into later. `Verb` is *HTTP request method*. `URI Pattern`  is what we type into the browser after the domain name. 

Take a close look, the `portfolios#new` action is just offering the form which the user fills, whereas the `portfolios#create` action is the one that actually talks to the database to generate a new portfolio item. Now, back in controller, do:
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  ...
  def new
  end
end
```
Note that if we don't have this `new` action, we might get `"No route matches [POST] "/portfolios/new"` error.

Remember to create the `app/views/portfolios/new.html.erb` file. Create a \<h1> tag, and then a form. Since we won't dig deeper into forms right now, we can just copy the scaffold-created one from `app/views/blogs/_form.html.erb`, which is a `partial`. Get rid of all lines on `errors`. Fix the naming:
```html
<h1>Create a new portfolio item</h1>

<%= form_with(model: @portfolio_item, local: true) do |form| %>

  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div class="field">
    <%= form.label :subtitle %>
    <%= form.text_field :subtitle %>
  </div>

  <div class="field">
    <%= form.label :body %>
    <%= form.text_area :body %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```
**Now, we created this `@portfolio_item` out of thin air for the `new.html.erb` file, so now we must include it in the corresponding action/method**:
```ruby
class PortfoliosController < ApplicationController
  ...
  def new
    @portfolio_item = Portfolio.new
  end
end
```
But remember, `new` ***only renders the form!*** It does not really create the portfolio item. In older versions of Rails, an error "**the action 'create' could not be found for PortfoliosController**". In our Rails, it is throwing a `routing error: No route matches [POST] "/portfolios/new"`.

The other half of the game is similar to the `create` action in `app/controllers/blogs_controller.rb`, which connects us to the database. Let's copy the code from `blogs_controller.rb` to our `portfolios_controller.rb`. 
```ruby
# app/controllers/portfolios_controller.rb#create
  def create
    @portfolio_item = Portfolio.new(params.require(:portfolio).permit(:title, :subtitle, :body))

    respond_to do |format|
      if @portfolio_item.save
        format.html { redirect_to portfolios_path, notice: 'Your portfolio is live.' }
      else
        format.html { render :new }
      end
    end
  end
```
Note: We don't know what `blog_params` is about so we'll copy the `private` method `def blog_params`'s line: `params.require(:blog).permit(:title, :body)` and feed into our create, just telling the form what we allow it to access, changing `:blog` to `:portfolio` and permit `:title`, `:subtitle` and `:body`. This is all just to make sure others cannot submit something via the form without certain information.

Delete the `format.json` lines, and we can change the `redirect_to` from `@portfolio_item` to showing the entire portfolio, a.k.a.` portfolios_path` (corresponding to the `rake routes` results where `Prefix` is `portfolios`).

If we try to submit a portfolio again, we see an error that we do not provide a `thumb_image` for the `app/views/portfolios/index.html.erb` file. Let's comment that off or do a conditional erb statement, and then everything works (choose any of the three lines with `image_tag` below but not all):
```html
<!--app/views/portfolios/index.html.erb-->
<h1>Portfolio Items</h1>

<% @portfolio_items.each do |portfolio_item| %>
	<p><%= portfolio_item.title %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image if !portfolio_item.thumb_image.nil? %>
	<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
	<%#= image_tag portfolio_item.thumb_image %> <!--or just comment off-->
<% end %>
```

### 3. Implement the Ability to Edit Database Records
Now, we need an `edit` action and `update` action.

#### A. edit action
First, let's create `app/views/portfolios/edit.html.erb` and copy everything from `app/views/portfolios/new.html.erb` to it, with a few changes:
```html
<h1>Edit This Portfolio Item</h1>
<%= form_with(model: @portfolio_item, local: true) do |form| %>

  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div class="field">
    <%= form.label :subtitle %>
    <%= form.text_field :subtitle %>
  </div>

  <div class="field">
    <%= form.label :body %>
    <%= form.text_area :body %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```
Then we create the `edit` action inside `app/controllers/portfolios_controller.rb`. 

Note that `edit` will be a bit different from `create`. For edit, the Prefix is `edit_portfolio`, its URI pattern is `/portfolios/:id/edit(.:format)` and controller#action is `portfolios#edit`. The URI requires `:id`.

**We'll find this `:id` by `params`, and all `params` does is examine the URI `/portfolios/:id/edit(.:format)` whenever a user enters it.**
```ruby
# app/controllers/portfolios_controller.rb
  def edit
    @portfolio_item = Portfolio.find(params[:id])
  end
```
Let's fire up the console to take a look. Note that if we only do `Portfolio.find()`, it will find with the id given (in this case `Portfolio.find(5)`).
```
$ rails c
... ...
irb(main):001:0> Portfolio.last
  Portfolio Load (1.2ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
=> #<Portfolio id: 11, title: "b", subtitle: "b", body: "b", main_image: nil, thumb_image: nil, created_at: "2020-01-20 14:27:32", updated_at: "2020-01-20 14:27:32">

irb(main):002:0> Portfolio.find(5)
  Portfolio Load (0.2ms)  SELECT "portfolios".* FROM "portfolios" WHERE "portfolios"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
=> #<Portfolio id: 5, title: "Portfolio title: 4", subtitle: "My great service", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-19 22:09:02", updated_at: "2020-01-19 22:09:02">
```

So, automatically, whenever we go to, say,  [http://127.0.0.1:3000/portfolios/1/edit](http://127.0.0.1:3000/portfolios/1/edit), we see not only the form but the data being populated already!

#### B. update action
But if we click on `Update Portfolio`, we get the `The action 'update' could not be found for PortfoliosController` error. 

So now is time we actually implement the `update` action as well. Let's just first copy the same method from `app/controllers/blogs_controller.rb`. We'll fix it in places such as `redirect_to portfolios_path`. 

Notice we will use again the line:
```
@portfolio_item = Portfolio.find(params[:id]) 
params.require(:portfolio).permit(:title, :subtitle, :body)
```
So to follow DRY, we will eventually need a method to generate these lines.

But for now, the end result looks like this:
```ruby
# app/controllers/portfolios_controller.rb
  def update
    @portfolio_item = Portfolio.find(params[:id])

    respond_to do |format|
      if @portfolio_item.update(params.require(:portfolio).permit(:title, :subtitle, :body))
        format.html { redirect_to portfolios_path, notice: 'The record was successfully updated.' }
      else
        format.html { render :edit }
      end
    end
  end
```

### 4. Use link_to for Creating Dynamic Links
First, we already know what our routes are with:
```
$ rake routes | grep portfolio
...
new_portfolio	GET		/portfolios/new(.:format)	portfolios#new
...
```

#### A. Add the link to new action
Ok, to create a new link that directly gets us to the `new` action, where we can create a new portfolio, we will do the following to `app/views/portfolios/index.html.erb`:
```html
<h1>...</h1>
<a href="portfolios/new">Create New Item</a>
...rest of the HTML code
```
Now, if you click `Create New Item` with the server run, you'll be redirected.

But there's a Rails way to do this with `link_to`, where we call the method `link_to`, offers a text description and then the path `new_portfolio_path`, which is basically the Prefix `new_portfolio` with `_path` added:
```ruby
<h1>...</h1>
<a href="portfolios/new">Create New Item</a>
<%= link_to "Create New Item", new_portfolio_path %>
...rest of the HTML code
```

***So why do the Prefixes not show `_path` by default?*** Well, they can show either `_path` or `_url`. You can do the following and print them out to examine:
```ruby
<h1>...</h1>
<a href="portfolios/new">Create New Item</a>
<%= link_to "Create New Item", new_portfolio_path %>
<p>
	<%= new_portfolio_path %> # => /portfolios/new
</p>

<p>
	<%= new_portfolio_url %> # => http://127.0.0.1:3000/portfolios/new
</p>
...rest of the HTML code
```
Now you get the difference. Rails is offering us the flexible option to choose! 80% of the time we might use the `_path` option than `_url` version.

But `_url` is useful for scenarios where we need `subdomains`:
```ruby
<%= link_to "Create New Item", new_portfolio_url, subdomain: 'my_subdomain' %>
```
Another reason is whenever we send an email, we need to send the `_url` method and not the relative path `_path` method.

#### B. Add the link to edit action for each id
The Prefix for edit is `edit_portfolio` with URI pattern `/portfolios/:id/edit(.:format)`. So we can pass the `edit_portfolio_path` method, but we need to specify the `:id` of the portfolio item, like `edit_portfolio_item(portfolio_item.id)`:
```ruby
<% @portfolio_items.each do |portfolio_item| %>
	<p><%= portfolio_item.title %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
	<%= link_to "Edit", edit_portfolio_path(portfolio_item.id) %>
<% end %>
```
**Very important, `link_to` creates an \<a> tag!** So it's basically the same as the below:
```ruby
<a href="/portfolios/<%= portfolio_item.id %>/edit">Edit</a>
```
`link_to`  is just cleaner code. Wow! Isn't that amazing :)

### 5. Create show link
We want each portfolio's title to be a link directed to the show page. So we need to create a `show` method, then it needs to pass to a `show` file that can render.

So let's create `show` in `app/controllers/portfolios_controller.rb`:
```ruby
  def show
    @portfolio_item = Portfolio.find(params[:id])
  end
```

Next, we create a `app/views/portfolios/show.html.erb`, just put in a minimal HTML:
```html
<h1>Show</h1>
```
And click [http://127.0.0.1:3000/portfolios/2](http://127.0.0.1:3000/portfolios/2), we see it works!

Let's enrich the `show.html.erb` file (Always refer to `schema.rb` for accessing database variables!):
```html
<%= image_tag @portfolio_item.main_image %>

<h1><%= @portfolio_item.title %></h1>

<em><%= @portfolio_item.subtitle %></em>

<p><%= @portfolio_item.body %></p>

```
Refresh the link again, yes it works. Now, let's add a link to the title of each portfolio item on index.

Remember from `rake routes | grep portfolio` that the Prefix for `portfolios#show` is just `portfolio`? **Then the `link_to` will only need `portfolio_path` inside the `index.html.erb`, of course also by specifying the `:id`**:
```html
...HTML
<% @portfolio_items.each do |portfolio_item| %>
	<p><%= link_to portfolio_item.title, portfolio_path(portfolio_item.id) %></p>
	...more HTML
<% end %>
```
Also, Rails is smart enough to know that if we use `portfolio_path`, we probably will feed the method an id, so we can actually just do `portfolio_path(portfolio_item)` and `edit_portfolio_path(portfolio_item)`:
```html
<% @portfolio_items.each do |portfolio_item| %>
	<p><%= link_to portfolio_item.title, portfolio_path(portfolio_item) %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
	<%= link_to "Edit", edit_portfolio_path(portfolio_item) %>
<% end %>
```

### 6. Implement destroy action (Delete)
Deleting is called `destroy`ing in Rails. Actually, `delete()` and `destroy()` are different:

 - `delete()`: [https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-delete](https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-delete)
 - `destroy()`: [https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-destroy](https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-destroy)

`Delete` is pure kind of going into database and I don't care about callbacks. `Destroy` is more careful in removing items. I'd get an error if I delete something that's related to another part of the database.

Let's create `destroy` action, which has no `views` templates since it works with the database, and copy the destroy code from `blogs_controller.rb`. We specify what `destroy` should destroy (`Portfolio.find(params[:id])`), then destroy it, and delete the line of `format.json` since we don't need to create API here yet, and finally specify `redirect_to portfolios_url`:
```ruby
# app/controllers/portfolios_controller.rb
  def destroy
    # Perform the lookup
    @portfolio_item = Portfolio.find(params[:id])

    # Destroy and delete the record
    @portfolio_item.destroy

    # Redirect
    respond_to do |format|
      format.html { redirect_to portfolios_url, notice: 'Record was removed.' }
    end
  end
```
Finally, let's make it happen on `views` under `index.html.erb`:
```html
<% @portfolio_items.each do |portfolio_item| %>
	<p><%= link_to portfolio_item.title, portfolio_path(portfolio_item) %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
	<%= link_to "Edit", edit_portfolio_path(portfolio_item) %>
	<%= link_to "Delete Portfolio", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } %>
<% end %>
```
**IMPORTANT!** But we're using `portfolio_path(portfolio_item)` both in the code above for the first `<p>` element and the last `erb` element. ***How does Rails know which leads to which action???*** 

Well, remember the routes here:
```
$ rake routes | grep portfolio
Prefix			Verb		URI Pattern				Controller#Action
portfolios 		GET    		/portfolios(.:format)	 		portfolios#index
			POST   		/portfolios(.:format)	 		portfolios#create
new_portfolio 		GET    		/portfolios/new(.:format)		portfolios#new
edit_portfolio 		GET    		/portfolios/:id/edit(.:format)		portfolios#edit
portfolio 		GET    		/portfolios/:id(.:format)		portfolios#show
			PATCH  		/portfolios/:id(.:format)		portfolios#update
			PUT    		/portfolios/:id(.:format)		portfolios#update
			DELETE 		/portfolios/:id(.:format)		portfolios#destroy
```
Prefix `portfolio` stands for all the verbs: `GET, PATCH, PUT, DELETE`. The way we tell how it works is in the `method` that we specify:
```ruby
<%= link_to "Delete Portfolio Item", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } %>
```
**If we don't specify `method: delete`, it'll be done as the `portfolio#show` action. Default is `method: :get`. Other verbs need specification!**
<!--stackedit_data:
eyJoaXN0b3J5IjpbODY5NDU3MjRdfQ==
-->
