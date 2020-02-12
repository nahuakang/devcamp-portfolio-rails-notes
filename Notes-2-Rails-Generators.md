# DevCamp Ruby on Rails Notes 2
## Section 5: Rails Generators
### 0. Note on Secrets
In `Rails 6`, there is no more `app/config/secrets.yml` but instead `app/config/master.key`, which is automatically inside the `app/.gitignore` file.

### 1. Controller Generator: PAGES
Always work on branches. Do:
```
$ git checkout -b controller-generator
```
We want to have the concept of pages, such as homepage, about, contact, etc. So let's do:
```
$ rails g controller Pages home about contact
```
Note that a controller generator does not talk to the database, so we don't pass in data types like `string` or `text`. This will create the following:

 - `app/controllers/pages_controller.rb` (incl. actions)
 - route: `get 'pages/home'`
 - route: `get 'pages/about'`
 - route: `get 'pages/contact'`
 - other relevant views for the actions and routes

We can go to `http://127.0.0.1:3000/pages/home` and there will be a bare-minimal page. This minimal creation from `controller` contrasts the `scaffold`'s richness.

#### MVC: Model, View, Controller
An architecture for data flow and communication.

All the routes in `app/config/routes.rb` are mapped to actions inside `app/controllers/pages_controller.rb`. 

A user hits: `http://127.0.0.1:3000/pages/home`. The router does `get 'pages/home'`, which triggers the action `pages#home`, basically the `home` method inside `pages_controller.rb`. Any instance variable inside methods of  `pages_controller.rb` will be made available to the `action`'s corresponding `views`.

### 2. Model Generator: SKILLS
Checkout another git branch:
```
$ git checkout -b model-generator
```
Run Rails to generate the model we want to represent skills (keep it singular):
```
$ rails g model Skill title:string percent_utilized:integer
```
This creates the following:
```
Running via Spring preloader in process 19101
    invoke  active_record    # <= this is ORM
    create  db/migrate/20200119203155_create_skills.rb
    create  app/models/skill.rb
```
Lighter-weight than our controller. We don't get views, routes, or anything. `schema.rb` file will not be updated until we run the following:
```
$ rails db:migrate
```
Now, if you type in to get Rails console, you can play with the model you just created:
```
$ rails c
```
Inside the console, we can run `ModelName.create` (for *production*) or `ModelName.create!` (throws an error if something goes wrong), put in the attributes `title` and `percent_utilized`:
```
irb(main):001:0> Skill.create!(title: "Rails", percent_utilized: 75)
```
Now we created a new record for `Skill`. Bring up all the skills by:
```
irb(main):001:0> Skill.all 
```

Now, inside `app/controllers/pages_controller.rb`, call the Skill Model `Skill.all`:
```ruby
class PagesController < ApplicationController
  def home
    @posts = Blog.all
    @skills = Skill.all
  end
```
Then, include it inside `app/views/pages/home.html.erb`:
```html
...
<h2>Skills</h2>
<%= @skills.inspect %>
```
And we'll be able to see it reflected!

### 3. Resource Generator
Checkout again on git:
```
$ git checkout -b resource-generator
```

Then, do the Rails command for resource generator:
```
$ rails g resource Portfolio title:string subtitle:string body:text main_image:text thumb_image:text
```
It created a whole bunch of files for us. We see `db/migrate` file (`active_record`), `controller` files, corresponding `views` directory (no views), `helper`, `assets` as well as `route`:
```
$ Running via Spring preloader in process 19654
      invoke  active_record
      create    db/migrate/20200119205110_create_portfolios.rb
      create    app/models/portfolio.rb
      invoke  controller
      create    app/controllers/portfolios_controller.rb
      invoke    erb
      create      app/views/portfolios
      invoke    helper
      create      app/helpers/portfolios_helper.rb
      invoke    assets
      invoke      scss
      create        app/assets/stylesheets/portfolios.scss
      invoke  resource_route
       route    resources :portfolios
```

**`resource generator`s are very skinny `scaffold`s.** We have everything except the `form`s, `view`s, or pre-filled items like in `Blog`. We might use `resource generator` more often than `scaffold` in day-to-day Rails programming.

Don't forget to migrate:
```
$ rails db:migrate
```
And the changes are reflected now in `db/schema.rb`.

### 4. Deep Dive into Generators (incl. Customization)
For this, we'll create a new application:
```
$ rails new GeneratorApp -T --database=postgresql
```
Run db create line:
```
$ rails db:create
```
We know `scaffold`:
```
$ rails g scaffold Post title:string body:text
$ rails db:migrate
```
Then, run server:
```
$ rails s
```
Then, we can go to:
```
http://127.0.0.1:3000/posts
```
And we have weird styles that come with `app/assets/stylesheets/scaffolds.scss`. So we can customize it.

#### 4.1 Customize Generators
Inside `app/config/application.rb`, add the following lines:
```ruby
...
module GeneratorApp
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0

    # ADD THE BLOCK BELOW:
    config.generators do |g|
      g.orm                   :active_record
      g.template_engine       :erb # can change to slim or haml
      g.test_framework        :test_unit, fixture:false
      g.stylesheets           false
      g.javascripts           false
    end

    # Don't generate system test files.
    config.generators.system_tests = nil
  end
end
```

Now, we can generate scaffold again and see:
```
$ rails g scaffold Blog title:string
$ rails db:migrate
```
This time, we won't see a `scaffold.scss` created, nor `javascript` files. 

#### 4.2 Overwrite Things with Generators
Also, you can customize what gets generated for your `html.erb` file, which is in `GeneratorApp/lib` directory, where we overwrite the styles.

First, create `GeneratorApp/lib/templates/` folder. Then we'll create `erb/scaffold/`: `GeneratorApp/lib/templates/erb/scaffold/`.

Then a file: `GeneratorApp/lib/templates/erb/scaffold/index.html.erb`. We can overwrite anything in this file. Whatever you put in here, a new `scaffold`'s `index` page will have what you wrote here.

So let's go to Rails' repo: [Rails: rails/generators/erb/scaffold/templates](https://github.com/rails/rails/tree/master/railties/lib/rails/generators/erb/scaffold/templates), copy paste the `index.html.erb.tt`, which then we customize. All this change will happen for future `scaffold` generated items.

Change the default index to:
```html
<h1><%= plural_table_name.titleize %></h1>
<hr>
<p id="notice"><%%= notice %></p>
<div>
  <%% @<%= plural_table_name %>.each do |<%= singular_table_name %>| %>
    <div>
<% attributes.reject(&:password_digest?).each do |attribute| -%>
      <p><%%= <%= singular_table_name %>.<%= attribute.column_name %> %></p>
<% end -%>
      <p><%%= link_to 'Show', <%= model_resource_name %> %></p>
      <p><%%= link_to 'Edit', edit_<%= singular_route_name %>_path(<%= singular_table_name %>) %></p>
      <p><%%= link_to 'Delete', <%= model_resource_name %>, method: :delete, data: { confirm: 'Are you sure?' } %></p>
    </div>
  <%% end %>
</div>
<br>
<%%= link_to 'Create a new <%= singular_table_name.titleize %>', new_<%= singular_route_name %>_path %>
```


Create another scaffold:

```
$ rails g scaffold Category title:string description:text
$ rails db:migrate
```

Now go to [http://127.0.0.1:3000/categories](http://127.0.0.1:3000/categories). It will be different.

You can even create your generators completely from scratch.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIyMzA5NDUxMl19
-->