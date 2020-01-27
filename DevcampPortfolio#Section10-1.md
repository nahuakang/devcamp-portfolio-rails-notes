# DevCamp Ruby on Rails Notes 12
## Section 10: Views

### 1. Customize Rails master Layout File

Do:
```
$ git checkout -b view
```

Our `application.html.erb` layout is by default the one that Rails uses. If in the `ApplicationController` we do the following:
```ruby
# app/controllers/application_controller.rb
...ruby code...
  before_action :set_title

  def set_title
    @page_title = "Devcamp Portfolio | My Portfolio Website"
  end
...ruby code...
```
And then inside `application.html.erb` we do:
```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= @page_title %></title>
...html code...
```
Then we have dynamically set the page title for our page.

But `set_title` is the default for the `ApplicationController` and `@page_title` can be accessed by other controllers (via overwriting in an action). We can do it for other controllers like `BlogsController` or `PortfoliosController` as well:
```ruby
# app/controllers/blogs_controller.rb
class BlogsController < ApplicationController
  before_action :set_blog, only: [:show, :edit, :update, :destroy, :toggle_status]

  def index
    @blogs = Blog.all
    @page_title = "My Portfolio Blog"
  end

  def show
    @page_title = @blog.title # accessed via before_action :set_blog
  end
```

Ok, let's refactor it into a `concern`. Create `app/controllers/concerns/defaul_page_content.rb`:
```ruby
# app/controllers/concerns/defaul_page_content.rb
module DefaultPageContent
  extend ActiveSupport::Concern

  included do
    before_action :set_page_defaults
  end

  def set_page_defaults
    @page_title = "Devcamp Portfolio | My Portfolio Website"
  end
end
```
Change our `application_controller.rb` to:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
  include SetSource
  include CurrentUserConcern
  include DefaultPageContent
end
```
Check Rails server to make sure everything works.

Now, you noticed we change the method from `set_title` to `set_page_defaults` because we can include other instance variables, such as a `@seo_keywords`:
```ruby
# app/controllers/concerns/defaul_page_content.rb
...ruby code...
  def set_page_defaults
    @page_title = "Devcamp Portfolio | My Portfolio Website"
    @seo_keywords = "Nahua Kang portfolio"
  end
...ruby code...
```
This can be then put into our `application.html.erb` as a `meta tag`:
```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= @page_title %></title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <meta name="keywords" content="<%= @seo_keywords %>" />
...html code...
```
Now refresh our blog and click "View Page Source", we should see:
```html
<meta name="keywords" content="Nahua Kang portfolio" />
```

Now, inside of `BlogsController#show`, we can overwrite the default `@seo_keywords` to something else:
```ruby
# app/controllers/blogs_controller.rb
...ruby code...
  def show
    @page_title = @blog.title
    @seo_keywords = @blog.body
  end
...ruby code...
```
In fact, to really do SEO optimization, we'd even create `@blog.keywords` attribute. Using `@blog.body` isn't the way to do it and is just for educational purpose here.

Anyways, everything starts at the master layout `application.html.erb`. Let's do some more to the master layout (use `rake routes | grep [pages or portfolios or blogs]` to see how to use `_path` with `link_to`):
```html
...html code...
  <body>
    <p class="notice"><%= notice %></p>
    <p class="alert"><%= alert %></p>

    <div>
      <div><%= link_to "Home", root_path %></div>
      <div><%= link_to "About", about_path %></div>
      <div><%= link_to "Contact", contact_path %></div>
      <div><%= link_to "Blog", blogs_path %></div>
      <div><%= link_to "Portfolio", portfolios_path %></div>
    </div>
...html code...
```
Now we have a bare-minimal navigation tab on the top (albeit ugly-looking).

Lastly, `<%= yield %>` is the most important bit on the `application.html.erb`. This line slides in all the content on other pages' HTML files. Basically it's the same as the Jinja template's `block content`.


### 2. Create a Secondary Layout File
What do we have to do to have another layout file? Well, definitely inside the `app/views/layouts/` directory ;) 

Let's create `app/views/layouts/blog.html.erb` and copy paste all content from `app/views/layouts/application.html.erb` into it. We'll move things around a bit.

Most importantly, we can change the stylesheets in the `<head>` from:
```ruby
    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```
to:
```ruby
    <%= stylesheet_link_tag 'blogs', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```
This is because when we ran `scaffold`, we have a `blogs.scss` generated under `app/assets/stylesheets` already. 

Then we move the nav bar to below `yield`.

Add this to `blogs.scss` (make sure we delete everything from `application.scss` first so that we have no cross-over between application styles and blog styles):
```scss
body {
	background-color: grey;
}
```
Finally, if we want to use `blog.html.erb` as layout, all we do is add inside `BlogsController`:
```ruby
class BlogsController < ApplicationController
  before_action :set_blog, only: [:show, :edit, :update, :destroy, :toggle_status]
  layout "blog" # <- ADD THIS LINE 
...ruby code...
```

Basically: 

 1. Stylesheet changes happen inside the `app/views/layouts/`
 2. Layout changes happen inside the `app/controllers/`

Let's do the same for Portfolios:

 1. Create `app/views/layouts/portfolio.html.erb`
 2. Copy paste `application.html.erb` to `portfolio.html.erb`
 3. In `portfolio.html.erb`, change `stylesheet_link_tag` to `<%= stylesheet_link_tag 'portfolios', media: 'all', 'data-turbolinks-track': 'reload' %>` (`portfolios.scss` already exists after migration)
 4. Inside `portfolios_controller.rb`, add `layout "portfolio"`

### 3. Implement Partials for Navbar
A `partial` file is a partial piece of code that can be shared by `view` elements/pages. A `partial` file has an underscore at the beginning of the file name. We call a `partial` with `render`.

Look, in our layouts `blog.html.erb` and `portfolio.html.erb`, we have repetitive code for the navbar (just different in where on the page they are located). This is a poor practice. We should put them into a `partial` and share it among these layout pages.


Let's create a new folder called `app/views/shared/`. Then we create `app/views/shared/_nav.html.erb`:
```ruby
# app/views/shared/_nav.html.erb
<div>
  <div><%= link_to "Home", root_path %></div>
  <div><%= link_to "About", about_path %></div>
  <div><%= link_to "Contact", contact_path %></div>
  <div><%= link_to "Blog", blogs_path %></div>
  <div><%= link_to "Portfolio", portfolios_path %></div>
</div>
```
Now, back inside the layouts html files, if we call the `partial`, Rails already assumes that it is under `views/` directory. So we specify only:
```ruby
<%= render "shared/nav" %>
```
**Note that we don't even need to use underscore here.** We just need to specify `shared/nav`. `render` is specific for `partials`. We'll do this for:

 1. `application.html.erb`
 2. `blog.html.erb`
 3. `portfolio.html.erb`

Check in `rails s` that everything works.

### 4. Advanced Partial with Data
Let's implement a `partial` that we can use to pass data into.

#### Data Flow inside Partial
First, take `blogs_controller.rb` as an example. We have `BlogsController#new` action:
```ruby
# app/controllers/blogs_controller.rb
...ruby code...
  def new
    @blog = Blog.new
  end
...ruby code...
```
Here is an instance variable `@blog`, which is then passed into the file `app/views/blogs/new.html.erb`:
```html
# app/views/blogs/new.html.erb
<h1>New Blog</h1>

<%= render 'form', blog: @blog %>

<%= link_to 'Back', blogs_path %>
```
Inside this `new.html.erb`, we render a `partial` which takes in the instance variable from `BlogsController#new`. Now this `blog` which we assigned the value of `@blog` is a local variable passed in via the `partial` file.

Now, if we take a look at `_form.html.erb` then, we know that all the `blog` variables are just the local variable that took in the value of `@blog` instance variable from our `BlogsController#new`:
```html
# app/views/blogs/_form.html.erb
<%= form_with(model: blog, local: true) do |form| %>
  <% if blog.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(blog.errors.count, "error") %> prohibited this blog from being saved:</h2>

      <ul>
        <% blog.errors.full_messages.each do |message| %>
...html code...
```

#### Implement Partial with Data Flow
Since all of our layouts (`application`, `portfolio`, and `blog`) share the same `_nav.html.erb`, does that mean we can't change the styling for each of the layouts? Yes we can. Jsut by making it dynamic inside `_nav.html.erb`:
```html
<div class="<%= location %>"> # <- THIS 'location' IS A LOCAL
  <div><%= link_to "Home", root_path %></div>
  <div><%= link_to "About", about_path %></div>
  <div><%= link_to "Contact", contact_path %></div>
  <div><%= link_to "Blog", blogs_path %></div>
  <div><%= link_to "Portfolio", portfolios_path %></div>
</div>
```
This `location` is basically the same idea as `blog` in the example above.

Then we can change our layouts files with different local variables:
```ruby
# app/views/layouts/application.html.erb
<%= render "shared/nav", location: 'top' %>
```
```ruby
# app/views/layouts/portfolio.html.erb
<%= render "shared/nav", location: 'top' %>
```
```ruby
# app/views/layouts/blog.html.erb
<%= render "shared/nav", location: 'bottom' %>
```

Now we update our `app/assets/stylesheets` files, including

 1. `application.css`
 2. `portfolios.scss`
 4. `blogs.scss`

```css
# portfolios.scss
.top a {
  color: red;
}
```
```css
# blogs.scss
.bottom a {
	color: white;
}
```
```css
# application.css
.top a {
	color: brown;
}
```
Now everything should work.

Let's create a file `app/views/portfolios/_form.html.erb` and copy all the `form` lines from `app/views/portfolios/new.html.erb`. Then we'll just `render` the `form` for:

 1. `app/views/portfolios/new.html.erb`
 2. `app/views/portfolios/edit.html.erb`


So from `new.html.erb`:
```html
<h1>Create a new portfolio item</h1>

<%= render 'form', portfolio_item: @portfolio_item %>
```
And from `edit.html.erb`:
```html
<h1>Edit This Portfolio Item</h1>

<%= render 'form', portfolio_item: @portfolio_item %>
```
In `_form.html.erb`, make sure `form_with(mode: @portfolio_item, local: true)` becomes `form_with(mode: portfolio_item, local:true)` since we're passing in the local `portfolio_item`:
```html
<%= form_with(model: portfolio_item, local: true) do |form| %>

  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

...html code...

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

### 5. Guide to View Helpers

View helpers are similar to partials. We can call it from our `view` files but a helper will be written in Ruby and a `partial` is written in HTML. Inside `app/helpers/application_helper.rb`, we can put:
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def sample_helper
    "<p>My helper</p>".html_safe # strips off the <p> tags
  end
end
```
Now inside `home.html.erb`, append:
```html
<h1>Hi, <%= current_user.first_name %></h1>
<p>Find me in app/views/pages/home.html.erb</p>

<h2>Blog Posts</h2>
<%= @posts.inspect %>

<h2>Skills</h2>
<%= @skills.inspect %>

<hr>

<%= sample_helper %> # <- DO THIS
```
We will see the `My helper` on the homepage.

**NOTE** the `.html_safe` sanitizes HTML from the string (otherwise we'd see: `<p>My helper</p>`). If you have to use user input, don't use `.html_safe`, otherwise put it in.

If we don't like writing too much ERB in our `html.erb` files, we might be able to just use a `view helper`. Such as our navigation tabs. We also don't want to put them in a `partial` because we want to leave out the conditional logic outside the `views` directly. Moving conditional logic into a `partial` does not really solve the issue here.

Let's do this.

Inside `application.html.erb`, `blog.html.erb`, and `portfolio.html.erb`, we have the same line of code:
```ruby
    <% if current_user.is_a?(User) %>
      <%= link_to "Logout", destroy_user_session_path, method: :delete %>
    <% else %>
      <%= link_to "Register", new_user_registration_path %>
      <%= link_to "Login", new_user_session_path %>
    <% end %>
```

Let's put it all into `application_helper.rb`, where all application pages can access:
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def login_helper
    if current_user.is_a?(User)
      link_to "Logout", destroy_user_session_path, method: :delete
    else
      (link_to "Register", new_user_registration_path) +
      "<br>".html_safe +
      (link_to "Login", new_user_session_path)
    end
  end
end
```
Now inside the three layouts, we'll just do:
```ruby
...html code...
	<%= login_helper %>
...html code...
```
And we get our register/login/logout functions on the page again.

**When to use a `view helper` vs. a `partial`? If the majority of the logic is based on Ruby code, then use `view helper`. If it's majority HTML code, use a `partial`.**

### 6. Use Rails `content_tag` Helper to Auto Generate HTML Code
What is a `content helper`? They are methods we can use inside our `views` or `helper methods`.

Let's go to `home.html.erb`:
```html
<h1>Hi, <%= current_user.first_name %></h1>
<p>Find me in app/views/pages/home.html.erb</p>

<h2>Blog Posts</h2>
<%= @posts.inspect %>

<h2>Skills</h2>
<%= @skills.inspect %>

<hr>

<%= content_tag :p, class: "my-special-class" do %> # <- DO THIS
Hi I am in a paragraph tag.
<% end %>
```
Once you inspect it on your homepage, you see it's in the source code as:
```html
<p class="my-special-class">
Hi I am in a paragraph tag.
</p>
```

So why do we use `content_tag`? Very helpful inside `helper methods`:
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def login_helper
	...ruby code...
  end

  # def sample_helper
  #   "<p>My helper</p>".html_safe
  # end

  def sample_helper
    content_tag(:div, "My content", class: "my-class")
  end   
end
```
Now we can call this sample_helper again inside `home.html.erb`:
```html
<h1>Hi, <%= current_user.first_name %></h1>
<p>Find me in app/views/pages/home.html.erb</p>

<h2>Blog Posts</h2>
<%= @posts.inspect %>

<h2>Skills</h2>
<%= @skills.inspect %>

<hr>

<%= sample_helper %>
```
And we get at the bottom the following HTML code when inspecting the element:
```html
<div class="my-class">My content</div>
```
`content_tag` is cleaner and more dynamic than the hard code we did before with `sample_helper` (better than typing `.html_safe` as well). 

Now, remember our `session` tracking little system on all layouts, i.e.  `application.html.erb`, `blog.html.erb`, and `portfolio.html.erb`:
```ruby
    <% if session[:source] %>
      Thanks for visiting me from <%= session[:source] %>
    <% end %>
```
Let's refactor it with `application_helper.rb` using `content_tag`:
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def login_helper
    ...ruby code...
  end

  def source_helper
    if session[:source]
      greeting = "Thanks for visiting me from #{session[:source]}"
      content_tag(:p , greeting, class: "source-greeting")
    end
  end
end
```
Then we replace the ERB in `application.html.erb`, `blog.html.erb`, and `portfolio.html.erb` with:
```html
...html code...
    <%= source_helper %>
...html code...
```

Now do [http://127.0.0.1:3000/?q=facebook](http://127.0.0.1:3000/?q=facebook) in *incognito* to check that it works indeed.

We can pass parameters to the `source_helper` method, too, like a standard Ruby method!

### 7. Rendering Data Collections via Partials

We'll go to `app/views/blogs/index.html.erb`. Let's create `_blog.html.erb` partial inside `app/views/blogs`:
```html
# app/views/blogs/_blog.html.erb
<tr>
  <td><%= blog.title %></td>
  <td><%= blog.body %></td>
  <td><%= link_to blog.status, toggle_status_blog_path(blog) %></td>
  <td><%= link_to 'Show', blog %></td>
  <td><%= link_to 'Edit', edit_blog_path(blog) %></td>
  <td><%= link_to 'Delete Post', blog, method: :delete, data: { confirm: 'Are you sure?' } %></td>
</tr>
```
And leave inside `app/views/blogs/blog.html.erb`:
```html
...html code...
  <tbody>
    <%= render @blogs %>
  </tbody>
...html code...
```
***Holy shit it still works!* This is convention over configuration. One convention is that if we choose to pass in the instance variable `render @blogs`, in the background Rails would first look for a `partial` named singular `_blog` and if it finds it, Rails will render each of them for us without us doing the `.each` method!**

Let's do this for our `app/views/portfolios/index.html.erb`. We create a `_portfolio_item.html.erb` partial first under `app/views/portfolios/`, then we do:
`app/views/portfolios/index.html.erb`
```html
<h1>Portfolio Items</h1>

<%= link_to "Create New Item", new_portfolio_path %>

<%= render @portfolio_items %>

```
And `app/views/portfolios/_portfolio_item.html.erb`:
```html
<p><%= link_to portfolio_item.title, portfolio_show_path(portfolio_item) %></p>

<p><%= portfolio_item.subtitle %></p>

<p><%= portfolio_item.body %></p>

<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
<%= link_to "Edit", edit_portfolio_path(portfolio_item) %>
<%= link_to "Delete Portfolio", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } %>
```
Should work, right? Huh no. It does not. We did exactly the same as with `blogs`, but we are missing one little step. Remember `controllers`? `blogs` is the controller name and the table name! **For `portfolios`, `portfolio_items` isn't the Rails-generated name! We need to give Rails some hints about the changes we choose in order for it to work!** 

Again, our `blogs` template worked with `render @blogs` with `partials`. The naming matches perfectly. It's the Rails way of doing things (***strict naming convention***).

Our `portfolios` don't work. We're getting the following errors:
```
Missing partial portfolios/_portfolio with {:locale=>[:en], :formats=>[:html], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :jbuilder]}
```
Why is it looking for `portfolios/_portfolio` when we named it `_portfolio_item`??? Well, it's a hint for Rails' parsing engine that is looking for the name that follows the naming pattern! Rails doesn't know we are switching the name to `portfolio_item`. 

**SOLUTION.** Now let's fix it:
`app/views/portfolios/index.html.erb`
```html
<h1>Portfolio Items</h1>

<%= link_to "Create New Item", new_portfolio_path %>

<%= render partial: 'portfolio_item', collection: @portfolio_items %>
```
The `collections` is telling Rails that we want to render this `@portfolio_items` collection type looking for the `_portfolio_item` partial.

### 8. Helpful ActionView Helper Methods
Check out here [https://guides.rubyonrails.org/layouts_and_rendering.html](https://guides.rubyonrails.org/layouts_and_rendering.html) for more information.

#### Method: distance_of_time_in_words
To the `blogs/index.html.erb`, let's add a date to the table:
```html
...html code...
  <thead>
    <tr>
      <th>Title</th>
      <th>Date</th> # <- ADD THIS HERE
      <th>Body</th>
      <th colspan="4"></th>
    </tr>
  </thead>
...html code...
```
And inside `blogs/_blog.html.erb`, we add:
```html
...html code...
<tr>
  <td><%= blog.title %></td>
  <td><%= blog.created_at %></td> # <- ADD THIS HERE
  <td><%= blog.body %></td>
...html code...
```
Now refreshing to the blogs page, we have the dates. Let's implement a feature for how long ago the blog is posted by way of a `view helper` called `distance_of_time_in_words()` which takes in two arguments, time of the blog's creation and the current time:
```html
...html code...
<tr>
  <td><%= blog.title %></td>
  <td>Published <%= distance_of_time_in_words(blog.created_at, Time.now) %> ago</td>
  <td><%= blog.body %></td>
...html code...
```
Now instead of the hard-coded time, we get `2 days ago`, `less than a minute ago`, etc.

#### Method: number_to_phone
Go to `contact.html.erb`, say if we want to display our phone number:
```html
<h1>Pages#contact</h1>
<p><%= number_to_phone "5555555555" %></p>
```
We get the phone displayed as `555-555-5555`.

#### Method: number_to_currency
Go to `contact.html.erb`, say if we want to display currency:
```html
<h1>Pages#contact</h1>
<p><%= number_to_phone "5555555555" %></p>
<p><%= number_to_currency 150 %></p>
```
We get: `$150.00`.

#### Method: number_to_percentage
```html
<h1>Pages#contact</h1>
<p><%= number_to_phone "5555555555" %></p>
<p><%= number_to_currency 150 %></p>
<p><%= number_to_percentage 80.4 %></p>
```
We get: `80.400%`.

#### Method: number_with_delimiter
```html
<h1>Pages#contact</h1>
<p><%= number_to_phone "5555555555" %></p>
<p><%= number_to_currency 150 %></p>
<p><%= number_to_percentage 80.4 %></p>
<p><%= number_with_delimiter 12323238979878971 %></p>
```
We get: `12,323,238,979,878,971`.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkzMTk4NTY1NCwyMDc5ODU5ODQyLDEzOT
AwMjE2OTAsLTE3NjM1MzExNDEsNDk3NDY3NjM3LC0xMTYwMjAw
ODE0LC0xOTIwNzk0MTI2LC00NzgwMDI4NTMsNTgyNTU2NTkzLD
E5NTk1MzAzODEsLTk4MTc4MTQ2OSwtNDg5MzI4MjQ3LDEwMDQz
MzIsLTIxMTMwNTk1MSwtNzQ1Njc1Njc5LC04NzI3MDU2MzQsMT
gxMDEwNTQyNV19
-->