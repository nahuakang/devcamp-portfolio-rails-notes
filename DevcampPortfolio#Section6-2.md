# DevCamp Ruby on Rails Notes 4
## Section 6: Rails Data Flow
### Question: Why Callbacks use Symbols in Rails?
[https://stackoverflow.com/a/26786737](https://stackoverflow.com/a/26786737)

### Question: Routes, any difference between to: or =>?
[https://stackoverflow.com/a/22494102](https://stackoverflow.com/a/22494102)

### 7. Custom Routes for Pages
Go to `DevcampPortfolio/config/routes.rb`:
```ruby
Rails.application.routes.draw do
  resources :portfolios
  get 'pages/home'
  get 'pages/about'
  get 'pages/contact'
  resources :blogs
end
```
We can change it to set the root path below as well:
```ruby
Rails.application.routes.draw do
  resources :portfolios

  get 'pages/about'
  get 'pages/contact'

  resources :blogs
  
  root to: 'pages#home' # sets the root path
end
```
**IMPORTANT! Whenever you change the `routes.rb` file, you must restart the server!** Restart the server and then go to [http://127.0.0.1:3000/](http://127.0.0.1:3000/).

**Now, what if I want to go to about but not to `http://127.0.0.1:3000/pages/about` but just `http://127.0.0.1:3000/about`?**
Do `get 'about', to: 'pages#about'`.

```ruby
Rails.application.routes.draw do
  resources :portfolios

  get 'about', to: 'pages#about'
  get 'pages/contact'

  resources :blogs
  
  root to: 'pages#home' # sets the root path
end
```
**EXPLANATION:** If we pass in `routes.rb` just:
```ruby
get 'pages/contact'
```
Rails is smart enough to know that the left side of the slash is controller `pages` and the right side of it is the action `contact`. But if not, it assumes the `to: 'controller#action'` as what to do.

So let's change all:
```ruby
Rails.application.routes.draw do
  resources :portfolios

  get 'about', to: 'pages#about'
  get 'contact', to: 'pages#contact'

  resources :blogs
  
  root to: 'pages#home'
end
```

If you run `rake routes`, you will see the change:
```
  about 	GET 	/about(.:format) 		pages#about
contact 	GET 	/contact(.:format) 		pages#contact
   root		GET		/						pages#home
```
We have TOTAL CONTROL over what to pass in for specific `controller#action`.

### 8. Custom Routes for Resources Routes

**How would we go about changing the URI pattern of `portfolios#show` from `portfolios/:id` to `portfolio/:id`?**

Use `except: [:show]`:
```ruby
# DevcampPortfolio/config/routes.rb
Rails.application.routes.draw do
  resources :portfolios, except: [:show]
  get 'portfolio/:id', to: 'portfolios#show'
  ...more routes
end
```
Now, check routes again:
```ruby
$ rake routes | grep portfolio
portfolios 		GET    /portfolios(.:format)			portfolios#index
				POST   /portfolios(.:format)			portfolios#create 
new_portfolio 	GET    /portfolios/new(.:format)		portfolios#new
edit_portfolio 	GET    /portfolios/:id/edit(.:format)	portfolios#edit
portfolio 		PATCH  /portfolios/:id(.:format)		portfolios#update
				PUT    /portfolios/:id(.:format)		portfolios#update
				DELETE /portfolios/:id(.:format)		portfolios#destroy
				GET    /portfolio/:id(.:format)			portfolios#show
```
If we hit up the server and try to go to [http://127.0.0.1:3000/portfolio/1](http://127.0.0.1:3000/portfolio/1), we get an `# ActiveRecord::RecordNotFound in PortfoliosController#show` error.

**This is caused by the fact that the Prefix `portfolio_path` method does not work for the custom route!** And it means that we need to create our own method name with `as: 'portfolio_show'` (**This `portfolio_show` will become `portfolios#show`'s Prefix**):
```ruby
# DevcampPortfolio/config/routes.rb
Rails.application.routes.draw do
  resources :portfolios, except: [:show]
  get 'portfolio/:id', to: 'portfolios#show', as: 'portfolio_show'
  ...more routes
end
```
Check `rake routes` again:
```ruby
$ rake routes | grep portfolio
portfolios 		GET    /portfolios(.:format)			portfolios#index
				POST   /portfolios(.:format)			portfolios#create 
new_portfolio 	GET    /portfolios/new(.:format)		portfolios#new
edit_portfolio 	GET    /portfolios/:id/edit(.:format)	portfolios#edit
portfolio 		PATCH  /portfolios/:id(.:format)		portfolios#update
				PUT    /portfolios/:id(.:format)		portfolios#update
				DELETE /portfolios/:id(.:format)		portfolios#destroy
portfolio_show	GET    /portfolio/:id(.:format)			portfolios#show
```
**Finally, update the views!** Go to index.html.erb and change any `portfolios#show`'s `portfolio_path` to `portfolio_show_path`:

### 9. Friendly Routes
What are friendly routes? URLs with words rather than pure `:id`s. `slug` is an example of a friendly route. *It's also more SEO friendly.*

First we want to integrate the `friendly_id` gem. Repo is here: [https://github.com/norman/friendly_id](https://github.com/norman/friendly_id).

Inside the `Gemfile`, do the following:
```ruby
# Add friendly_id for Udemy lecture 66
gem 'friendly_id', '~> 5.2.4' # Note: You MUST use 5.0.0 or greater for Rails 4.0+
```
Then run: 
```ruby
$ bundle install
```
Run
```ruby
$ rails generate friendly_id
      create  db/migrate/20200121092153_create_friendly_id_slugs.rb
      create  config/initializers/friendly_id.rb
```

And then look into `db/migrate/[timestamp]_create_friendly_id_slugs.rb`. We see that `friendly_id` uses its own data table.
```ruby
# db/migrate/[timestamp]_create_friendly_id_slugs.rb
...
class CreateFriendlyIdSlugs < MIGRATION_CLASS
  def change
    create_table :friendly_id_slugs do |t|
      t.string   :slug,           :null => false
      t.integer  :sluggable_id,   :null => false
      t.string   :sluggable_type, :limit => 50
      t.string   :scope
      t.datetime :created_at
    end
    add_index :friendly_id_slugs, [:sluggable_type, :sluggable_id]
    add_index :friendly_id_slugs, [:slug, :sluggable_type], length: { slug: 140, sluggable_type: 50 }
    add_index :friendly_id_slugs, [:slug, :sluggable_type, :scope], length: { slug: 70, sluggable_type: 50, scope: 70 }, unique: true
  end
end
```
Besides that, we also have `config/initializers/friendly_id.rb`, where we can do a lot of things (except not with the reserved words). We will just use the default action:
```ruby
FriendlyId.defaults do |config|
  config.use :reserved
  # reserved_words won't be used as part of the slug
  config.reserved_words = %w(new edit index session login logout users admin
    stylesheets assets javascripts images)
end
```
In any case, we need to run a migration to install the data table:
```
$ rails db:migrate
```

#### Migration to existing databases
Whenever we want to add or delete something (a column) from a database, we run a migration:
```
$ rails g migration [NAMING OF MIGRATION] [DATA:DATATYPE:UNIQUE]
```
**This `[NAMING OF MIGRATION]` is very important!** If we say `add_slug_to_blogs`, Rails will know that we want to add something to a database. If we specify the last bit to be the data table (`blogs`), Rails knows which data table to do migrations on. **We don't have to do it, but it's *extremely* helpful.** Do it with underscore is fine:

```
$ rails g migration add_slug_to_blogs slug:string:uniq
Running via Spring preloader in process 39155
      invoke  active_record
      create    db/migrate/20200121093311_add_slug_to_blogs.rb
```
Or with ***CamelCase***:
```
$ rails g migration AddSlugToBlogs slug:string:uniq
```
`uniq` ensures that no two blog posts have the same URL in the end.

Back in `db/migrate/[timestamp]_add_slug_to_blogs.rb`, we see that the migration already knows to add changes to `:blogs`:
```ruby
# db/migrate/[timestamp]_add_slug_to_blogs.rb
class AddSlugToBlogs < ActiveRecord::Migration[6.0]
  def change
    add_column :blogs, :slug, :string
    add_index :blogs, :slug, unique: true
  end
end
```
Not only do we add a column to `blogs` called `slug`, but to make it unique, we also add an `index` to the column. This we will cover in details in later chapters.

Finally, migrate it:
```
$ rails db:migrate
```

**THE SEQUENCE IS:**
```
$ rails generate migration [NAME OF MIGRATION] [DATA:DATATYPE]
$ rails db:migrate
```
Very similar to Python Django: `makemigrations`, `migrate`.

We can double-check that it is migrated inside the `schema.rb` file on that second last line:
```ruby
# db/schema.rb
...
  create_table "blogs", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "slug"
    t.index ["slug"], name: "index_blogs_on_slug", unique: true
  end
...
```
Now, we are not done yet. Remember to always follow the instruction from the repo: [https://github.com/norman/friendly_id](https://github.com/norman/friendly_id)

Go to our `app/models/blog.rb` file to make some changes (note instead of `:name` as in the repo, we'll use `:title`, which is the title of the blog that will be used for generating the slug):
```ruby
# app/models/blog.rb
class Blog < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: :slugged
end
```

Check it inside `rails console`:
```
$ rails c
irb(main):001:0> Blog.create!(title:"My great blog title", body:"Lorem ipsum blah blah blah")
   (142.3ms)  BEGIN
  Blog Exists? (5.1ms)  SELECT 1 AS one FROM "blogs" WHERE "blogs"."id" IS NOT NULL AND "blogs"."slug" = $1 LIMIT $2  [["slug", "my-great-blog-title"], ["LIMIT", 1]]
  Blog Create (0.5ms)  INSERT INTO "blogs" ("title", "body", "created_at", "updated_at", "slug") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["title", "My great blog title"], ["body", "Lorem ipsum blah blah blah"], ["created_at", "2020-01-21 09:49:26.328250"], ["updated_at", "2020-01-21 09:49:26.328250"], ["slug", "my-great-blog-title"]]
   (12.5ms)  COMMIT
=> #<Blog id: 12, title: "My great blog title", body: "Lorem ipsum blah blah blah", created_at: "2020-01-21 09:49:26", updated_at: "2020-01-21 09:49:26", slug: "my-great-blog-title">
```
Now there's a slug!

Finally, we need to change the `blogs_controller.rb` and how it finds the blog. Instead of 
```ruby
def set_blog
  @blog = Blog.find(params[:id])
end
```
WE WANT:
```ruby
def set_blog
  @blog = Blog.friendly.find(params[:id])
end
```
This is because the previous code was looking for `params[:id]` and by using `friendly.`, it does the work to not look for the `params[:id]` but instead the corresponding `slug`.

Finally, we need to get slugs for each existing blog (also inside the repo README):
```ruby
$ rails console
irb(main):001:0> Blog.find_each(&:save)
irb(main):002:0> Blog.first
  Blog Load (0.6ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> #<Blog id: 1, title: "My Blog Post 0", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", created_at: "2020-01-19 22:09:02", updated_at: "2020-01-21 09:54:24", slug: "my-blog-post-0">
```
This way, `friendly` will create a `slug` for each post. Example URL looks like this: [http://127.0.0.1:3000/blogs/my-great-blog-title](http://127.0.0.1:3000/blogs/my-great-blog-title).

When a user types down `http://127.0.0.1:3000/blogs/my-great-blog-title`, the router passes the `params` to the `blogs_controller` (like a quarterback passing the ball to the receiver). Specifically, we are passing the `params` into the method `Blog.friendly.find()`:
```ruby
    def set_blog
      @blog = Blog.friendly.find(params[:id])
    end
```

Concretely, it is 	`my-great-blog-title` as the `params`, wrapped in the `friendly` method, that we then store to be used in our actions, such as `show`, and then be passed into `views`.

**Data flow: `router -> controller -> model -> controller -> view`**


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDkzNDE1N119
-->