# DevCamp Ruby on Rails Notes 5
## Section 6: Rails Data Flow
### Curious cases:

**Question: What are *bang methods*?** [https://stackoverflow.com/questions/612189/why-are-exclamation-marks-used-in-ruby-methods](https://stackoverflow.com/questions/612189/why-are-exclamation-marks-used-in-ruby-methods)

**Question: What are enums?** [https://naturaily.com/blog/ruby-on-rails-enum](https://naturaily.com/blog/ruby-on-rails-enum)

### 10. Custom Actions for Updating Status
Essentially, we want the ability to have a button on the `blogs page` to toggle between being in `draft mode` or `publish mode`.

1. We need to update the `data model`, update the schema, and run migrations. This way we have the draft and publish modes.
2. Whenever data can change stages from publish to draft, or an invoice going through an approval flow. We could use `enum` to change state quickly in Rails.

#### Use Rails' `enum` to manage data stages
**What the heck is an `enum`?** [https://naturaily.com/blog/ruby-on-rails-enum](https://naturaily.com/blog/ruby-on-rails-enum)

First let's take a look at the `db/schema.rb`. Focus on the blogs table:
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
We want a `status` attribute. It needs to be an integer with a default value. Do:
```ruby
$ rails g migration add_post_status_to_blogs status:integer
Running via Spring preloader in process 22348
      invoke  active_record
      create    db/migrate/20200122144318_add_post_status_to_blogs.rb
```
Then, we can give `status` a default value inside the `db/migrate/[timestamp]_add_post_status_to_blogs.rb`:
```ruby
class AddPostStatusToBlogs < ActiveRecord::Migration[6.0]
  def change
    add_column :blogs, :status, :integer, default: 0
  end
end
```
`enum`s work with integers.
Now, do:
```ruby
$ rails db:migrate
```

Now, inside `db/schema.rb`, we see the `status` with default:
```ruby
  create_table "blogs", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "slug"
    t.integer "status", default: 0
    t.index ["slug"], name: "index_blogs_on_slug", unique: true
  end
```

All this is, is just a regular integer attribute. Nothing special about `status` yet. To make this work, we need to access `app/models/blog.rb` and create an `enum`:
```ruby
# app/models/blog.rb
class Blog < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: :slugged
end
```

We can pass to `enum` a `status` with key/value pair `{draft: 0, published: 1}`:
```ruby
# app/models/blog.rb
class Blog < ApplicationRecord
  enum status: { draft: 0, published: 1 }
  extend FriendlyId
  friendly_id :title, use: :slugged
end
```

So this `app/models/blog.rb` will be the only place where we discuss the 1 and 0 of the column `status`. Otherwise we'll be able to just call `draft` and `published`:
```ruby
$ rails c
Running via Spring preloader in process 22831
Loading development environment (Rails 6.0.2.1)
2.6.5 :001 > Blog.create!( title:"ASDF ASDF", body: "asdf asdf asdf asdf")
   (0.2ms)  BEGIN
  Blog Exists? (14.3ms)  SELECT 1 AS one FROM "blogs" WHERE "blogs"."id" IS NOT NULL AND "blogs"."slug" = $1 LIMIT $2  [["slug", "asdf-asdf"], ["LIMIT", 1]]
  Blog Create (29.0ms)  INSERT INTO "blogs" ("title", "body", "created_at", "updated_at", "slug") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["title", "ASDF ASDF"], ["body", "asdf asdf asdf asdf"], ["created_at", "2020-01-22 15:08:12.024709"], ["updated_at", "2020-01-22 15:08:12.024709"], ["slug", "asdf-asdf"]]
   (1.9ms)  COMMIT
 => #<Blog id: 13, title: "ASDF ASDF", body: "asdf asdf asdf asdf", created_at: "2020-01-22 15:08:12", updated_at: "2020-01-22 15:08:12", slug: "asdf-asdf", status: "draft">
```
Now we see the newly created blog`ASDF ASDF` has not only a `slug`, but a `status` of `draft`.

Again, we can change its `status` to `published` by using `Blog.last.published!`, then the `status` is set to 1. **Literally, we're treating `published` as a method here.**
```ruby
2.6.5 :002 > Blog.last.published!
  Blog Load (6.9ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" DESC LIMIT $1  [["LIMIT", 1]]
   (0.2ms)  BEGIN
  Blog Update (28.9ms)  UPDATE "blogs" SET "status" = $1, "updated_at" = $2 WHERE "blogs"."id" = $3  [["status", 1], ["updated_at", "2020-01-22 15:09:43.291996"], ["id", 13]]
   (6.2ms)  COMMIT
 => true
```
**This is an easy way to manage data at different stages.** Make change on the fly gets easy.

Running `Blog.published` (which is a ***database scope***) will run through the data table and query all items that are published:
```ruby
2.6.5 :004 > Blog.published
  Blog Load (0.6ms)  SELECT "blogs".* FROM "blogs" WHERE "blogs"."status" = $1 LIMIT $2  [["status", 1], ["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Blog id: 13, title: "ASDF ASDF", body: "asdf asdf asdf asdf", created_at: "2020-01-22 15:08:12", updated_at: "2020-01-22 15:09:43", slug: "asdf-asdf", status: "published">]>
 2.6.5 :005 > Blog.first.published!
  Blog Load (0.6ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" ASC LIMIT $1  [["LIMIT", 1]]
   (0.1ms)  BEGIN
  Blog Update (6.3ms)  UPDATE "blogs" SET "status" = $1, "updated_at" = $2 WHERE "blogs"."id" = $3  [["status", 1], ["updated_at", "2020-01-22 15:15:52.557219"], ["id", 1]]
   (5.6ms)  COMMIT
 => true
2.6.5 :006 > Blog.published
  Blog Load (0.9ms)  SELECT "blogs".* FROM "blogs" WHERE "blogs"."status" = $1 LIMIT $2  [["status", 1], ["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Blog id: 13, title: "ASDF ASDF", body: "asdf asdf asdf asdf", created_at: "2020-01-22 15:08:12", updated_at: "2020-01-22 15:09:43", slug: "asdf-asdf", status: "published">, #<Blog id: 1, title: "My Blog Post 0", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", created_at: "2020-01-19 22:09:02", updated_at: "2020-01-22 15:15:52", slug: "my-blog-post-0", status: "published">]>
 2.6.5 :007 > Blog.published.count
   (22.8ms)  SELECT COUNT(*) FROM "blogs" WHERE "blogs"."status" = $1  [["status", 1]]
 => 2
```
Eventually, we will only show visitors the `published` blogs by running `Blog.published` to do so for visitors, and `Blog.all` for site admins.

### 11. Implement Custom Action in Rails (Toggling Button)

Opens console:
```ruby
$ rails c
2.6.5 :001 > Blog.last.draft?
  Blog Load (2.8ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" DESC LIMIT $1  [["LIMIT", 1]]
 => false
2.6.5 :002 > Blog.last.published?
  Blog Load (0.9ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" DESC LIMIT $1  [["LIMIT", 1]]
 => true
```
Have an item selected, then we can run `draft?` and `published?` with a Boolean answer `true/false`. In the `blogs_controller.rb`, we can then use these to filter blog pieces.

Back to `app/views/blogs/index.html.erb`, let's first change `colspan`:
```html
<th colspan="4"></th>
```
Add a column:
```html
<td><%= blog.status %></td>
```
Like this:
```html
<!-- app/view/blogs/index.html.erb -->
<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Body</th>
      <th colspan="4"></th>
    </tr>
  </thead>

  <tbody>
    <% @blogs.each do |blog| %>
      <tr>
        <td><%= blog.title %></td>
        <td><%= blog.body %></td>
        <td><%= blog.status %></td>
        <td><%= link_to 'Show', blog %></td>
        <td><%= link_to 'Edit', edit_blog_path(blog) %></td>
        <td><%= link_to 'Delete Post', blog, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Temporarily change `<td><%= blog.status %></td>` to `<td><%= link_to blog.status, root_path %></td>` just to see if the link would go to `root`. Yes it does. In fact, `root_path` is the placeholder for our customized route.

Now, inside `config.routes.rb`, let's add to `resources :blog`:
```ruby
  resources :blogs do
    member do
      post :toggle_status
    end
  end
```
Then we test it out with `rake routes`:
```
$ rake routes | grep blog
toggle_status_blog 	POST   /blogs/:id/toggle_status(.:format)	blogs#toggle_status
blogs 			GET    /blogs(.:format)				blogs#index
	               	POST   /blogs(.:format)				blogs#create
new_blog 		GET    /blogs/new(.:format)			blogs#new
edit_blog 		GET    /blogs/:id/edit(.:format)		blogs#edit
blog			GET    /blogs/:id(.:format)			blogs#show
			PATCH  /blogs/:id(.:format)			blogs#update
			PUT    /blogs/:id(.:format)			blogs#update
			DELETE /blogs/:id(.:format)			blogs#destroy
```
We see a new route `toggle_status_blog` mapped to the action `blogs#toggle_status` with URI pattern: `/blogs/:id/toggle_status`.

So we can now use `toggle_status_blog_path` in `app/views/blogs/index.html.erb`:
```html
<td><%= link_to blog.status, toggle_status_blog_path %></td>
```
Try it with server, we'll get the error: `ActionController::UrlGenerationError in Blogs#index` if we try to access the link. This is because we do not have a `toggle_status` action inside `blogs_controller.rb`:
```ruby
# app/controllers/blogs_controller.rb
  def toggle_status
  end
``` 

Still fails. Because we need an `:id`. Since we are in `index.html.erb`, which has access to action `index`'s `@blogs` instance variable, we can just pass in each `blog` into the `toggle_status_blog_path` method like:
```html
<td><%= link_to blog.status, toggle_status_blog_path(blog) %></td>
```

Cool, now let's add `byebug` to `toggle_status`:
```ruby
# app/controllers/blogs_controller.rb
  def toggle_status
    byebug
  end
```
Now, accessing the link we still get a `No route matches [GET] "/blogs/my-great-blog-title/toggle_status"`, because our `blogs#toggle_status` is only a `POST` without `GET`. We can just add a redirect:
```ruby
# app/controllers/blogs_controller.rb
  def toggle_status
    byebug
    redirect_to blogs_url
  end
```

And turns out we need a `GET` not `POST` in `routes.rb`:
```ruby
# config/routes.rb
  resources :blogs do
    member do
      get :toggle_status # change from post to get
    end
  end
```
Ok, try out `byebug` and then remove it from `toggle_status` method. First, put `toggle_status` into the `before_action` list so that it has access to the `@blog` instance variable:
```ruby
class BlogsController < ApplicationController
  before_action :set_blog, only: [:show, :edit, :update, :destroy, :toggle_status]
  ...
  def toggle_status
    if @blog.draft?
      @blog.published!
    elsif @blog.published?
      @blog.draft!
    end
    
    redirect_to blogs_url, notice: 'Post status has been updated.'
  end
  ...
end
```




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNjU3OTc1NjJdfQ==
-->
