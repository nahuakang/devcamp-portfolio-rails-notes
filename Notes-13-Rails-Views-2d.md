# DevCamp Ruby on Rails Notes 13
## Section 10: Views Deep Dive
Create the bare minimal new app:
```
$ rails new ActiveViewDeepDive --database=postgresql
$ rails db:create
$ rails db:migrate
$ git status
$ git add .
$ git commit -m "init"
```

### 1. Create a Controller from Scratch
Create `app/controllers/guides_controller.rb`:
```ruby
# app/controllers/guides_controller.rb
class GuidesController < ApplicationController
  def book
  end
end
```
Whatever action we create, such as `GuidesController#book`, we can map it to a file in `app/views/guides/book.html.erb`. So create the folder `app/views/guides` and then the `book.html.erb` under this directory:
```html
<p>Hey, I'm in the book view</p>
```

So 

 - `guides_controller.rb`: `guides` => `views/guides/`
 - `GuidesController#book` => `views/guides/book.html.erb`


Don't forget `config/routes.rb`, which by default follows the following pattern:
```ruby
Rails.application.routes.draw do
  get 'guides/book' # <- ADD THIS LINE
end
```
Now go to [http://127.0.0.1:3000/guides/book](http://127.0.0.1:3000/guides/book) and it should work.
```ruby
# app/controllers/guides_controller.rb
class GuidesController < ApplicationController
  def book
    @books = ["War and Peace", "Land of Lisp", "The English Patient"]
  end
end
```
Inside `book.html.erb` use some ERB because we have access to the instance variable (`@books`) of the corresponding action `GuidesController#book`:
```html
<p>Hey, I'm in the book view</p>

<% @books.each do |book| %>
  <p><%= book %></p>
<% end %>

<%= @books %> # <- This will display
```
Note `<% %>` only processes the Ruby code while `<%= %>` displays it on the HTML page.

### 2. Create a Scaffold
```
$ rails g scaffold Blog title:string body:text
$ rails db:migrate
```
Let's refactor our `app/views/blogs/index.html.erb`, delete all the `tbody` content to leave out only:
`app/views/blogs/index.html.erb`
```html
<p id="notice"><%= notice %></p>

<h1>Blogs</h1>

<div>
  <%= render @blogs %>
</div>

<br>

<%= link_to 'New Blog', new_blog_path %>
```
Now, remember that Rails does the leg work for us so we can basically create, in the same directory, a partial named `_blog.html.erb` and Rails will understand how to iterate over all the `@blogs` items automatically for us:
`app/views/blogs/_blog.html.erb`
```html
<div>
  <p><%= blog.title %></p>
  <p><%= blog.body %></p>
  <p><%= link_to 'Show', blog %></p>
  <p><%= link_to 'Edit', edit_blog_path(blog) %></p>
  <p><%= link_to 'Destroy', blog, method: :delete, data: { confirm: 'Are you sure?' } %></p>
</div>
<hr>
```
We have a `<hr>` tag here. But we can actually just create a `spacer_template` partial:
`app/views/blogs/_blog.html.erb`
```html
<p id="notice"><%= notice %></p>

<h1>Blogs</h1>

<div>
  <%= render partial: @blogs, spacer_template: 'blog_ruler' %>
</div>

<br>

<%= link_to 'New Blog', new_blog_path %>
```
Now we create the `_blog_ruler.html.erb` partial:
`app/views/blogs/_blog_ruler.html.erb`
```html
<hr>
```
And in `_blog.html.erb` we delete `<hr>`:
```html
<div>
  <p><%= blog.title %></p>
  <p><%= blog.body %></p>
  <p><%= link_to 'Show', blog %></p>
  <p><%= link_to 'Edit', edit_blog_path(blog) %></p>
  <p><%= link_to 'Destroy', blog, method: :delete, data: { confirm: 'Are you sure?' } %></p>
</div>
```
Why do we do this way? Because we don't want the `<hr>` below the last blog post. We only want spacing in between blogs. Additionally it's much cleaner code as well.

Inside `app/views/layouts/application.html.erb` add Bootstrap:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>ActiveViewDeepDive</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    # ADD THIS BELOW
    <%= stylesheet_link_tag "https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

### 2. Caching
Caching is to take a slow-loading website, wrap the components up, and cache them into easy-to-access files for the user on the browser.
`app/views/blogs/index.html.erb`:
```html
...html code...
<% cache do %>
<div>
  <%= render partial: @blogs, spacer_template: 'blog_ruler' %>
</div>
...html code...
<% end %>
```
Then this code:
```html
<div>
  <%= render partial: @blogs, spacer_template: 'blog_ruler' %>
</div>
```
will be saved/cached to the client's browser so next time it loads faster!





<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ1NDQ0NTY3OSw1MTIxNDcyMjksMjA2ND
EyNzc0MiwxODU4MzQ4MjE0LC0xMDg2MTA2MTU5LDEzNTY2Mjgz
NjRdfQ==
-->