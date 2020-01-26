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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NTQyNDI3OTUsMTAwNDMzMiwtMjExMz
A1OTUxLC03NDU2NzU2NzksLTg3MjcwNTYzNCwxODEwMTA1NDI1
XX0=
-->