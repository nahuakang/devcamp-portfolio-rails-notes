# DevCamp Ruby on Rails Notes 7
## Section 7: Models - Data Management
Now we are in the ***M*** of MVC.

Let's pull updates and switch to a new branch
```
$ git checkout master
$ git pull
$ git checkout -b data-feature
```

As of now, if we go into Rails console and type in `Blog.create()`, we will create an empty Object with an `id`, but with `slug: nil`. That's not good and could cause errors, such as `nil:NilClass`.

### 1. Implement Data Validations
We want to protect our app from encountering these errors. One way is by implementing data validations so as to make `title` and `body` required.

Open `app/models/blog.rb`:
```ruby
class Blog < ApplicationRecord
  enum status: { draft: 0, published: 1 }
  extend FriendlyId
  friendly_id :title, use: :slugged

  validates_presence_of :title, :body # <- ADD THIS, we want blogs to have title and body
end
```
Now if we create an empty Blog Object, none of the unqualified `Blog` will be accepted anymore:
```ruby
$ rails c
2.6.5 :001 > Blog.create()
 => #<Blog id: nil, title: nil, body: nil, created_at: nil, updated_at: nil, slug: nil, status: "draft">
2.6.5 :003 > Blog.create!()
Traceback (most recent call last):
        1: from (irb):3
ActiveRecord::RecordInvalid (Validation failed: Title can't be blank, Body can't be blank)
2.6.5 :002 > Blog.last
  Blog Load (0.5ms)  SELECT "blogs".* FROM "blogs" ORDER BY "blogs"."id" DESC LIMIT $1  [["LIMIT", 1]]
 => #<Blog id: 13, title: "ASDF ASDF", body: "asdf asdf asdf asdf", created_at: "2020-01-22 15:08:12", updated_at: "2020-01-22 16:49:39", slug: "asdf-asdf", status: "draft">
```

***NOTE:*** This works also on the browser. If we create a new blog without `title` or `body`, or if we edit a blog and leave the `title` or `body` empty, we will get an error notification, such as `Body can't be blank`.

#### Update with validations for all models
Under `app/models`, we also have `portfolio.rb` and `skill.rb`, so let's update all of them along `db/schema.rb` to see the attributes and pick the ones that should be required:
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  validates_presence_of :title, :body, :main_image, :thumb_image
end
```
```ruby
# app/models/skill.rb
class Skill < ApplicationRecord
  validates_presence_of :title, :percent_utilized
end
```
Update with Git and Github.

### 2. Implement Data Relationships in Rails
Data relationships allow us to have hierarchy between different models.

For instance, in the `db/schema.rb` file, if we want to have some kind of connection between `blogs` and `portfolios`, which we currently don't, we'd have to build out some kind of an id that would associate one with the other.

We'll create a new model from scratch that is connected to the `blogs`, called `topic` (`rails g model` generates a migration file and a model file):
```ruby
$ rails g model Topic title:string
$ rails db:migrate
```
Next, go into `app/models/topic.rb` to immediately add validation:
```ruby
# app/models/topic.rb
class Topic < ApplicationRecord
  validates_presence_of :title
end
```
So far so good. But we want to have a connection betwee nour `blogs` and `topics` tables. So does `topic` belong to `blog` or vice versa? Example below:
```
Blogs:
 - baseball world series, title, topic_id = 1
 - superbowl, title, topic_id = 2
 - spring training, title, topic_id = 1

Topics:
 - topic_id: 1, baseball
 - topic_id: 2, football
```
It's clear that the `topic` owns the `blog` here. Because each `blog` can have a topic ID, but not each topic ID can have many `blog`s. Then let's make a migration:
```ruby
$ rails g migration add_topic_reference_to_blogs topic:references
```
Now we have the migration file `db/migrate/[timestamp]_add_topic_reference_to_blogs.rb`, which shows this reference as `foreign_key`:
```ruby
# db/migrate/[timestamp]_add_topic_reference_to_blogs.rb
class AddTopicReferenceToBlogs < ActiveRecord::Migration[6.0]
  def change
    add_reference :blogs, :topic, null: false, foreign_key: true
  end
end
```
NOTE: We'll delete `null: false` for now.
```ruby
# db/migrate/[timestamp]_add_topic_reference_to_blogs.rb
class AddTopicReferenceToBlogs < ActiveRecord::Migration[6.0]
  def change
    add_reference :blogs, :topic, foreign_key: true
  end
end
```
Run migrate:
```ruby
$ rails db:migrate
```
Now we see in `db/schema.rb` there's a new `t.bigint` under `create_table "blogs"`:
```ruby
    t.bigint "topic_id"
```
A final step before we can associate blogs with topics is to fix the models. Our blog now must know ***that it is owned by a topic*** (`belongs_to`):
```ruby
# app/models/blog.rb
class Blog < ApplicationRecord
  enum status: { draft: 0, published: 1 }
  extend FriendlyId
  friendly_id :title, use: :slugged

  validates_presence_of :title, :body

  belongs_to :topic # <- ADD THIS
end
```
The topic must know that ***it has many blogs to it*** (`has_many`):
```ruby
# app/models/topic.rb
class Topic < ApplicationRecord
  validates_presence_of :title

  has_many :blog # <- ADD THIS
end
```
Now we go into the console:
```ruby
$ rails c
2.6.5 :001 > Topic.create!(title: 'Ruby Programming')
   (0.2ms)  BEGIN
  Topic Create (25.4ms)  INSERT INTO "topics" ("title", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["title", "Ruby Programming"], ["created_at", "2020-01-24 15:45:31.113067"], ["updated_at", "2020-01-24 15:45:31.113067"]]
   (5.8ms)  COMMIT
 => #<Topic id: 1, title: "Ruby Programming", created_at: "2020-01-24 15:45:31", updated_at: "2020-01-24 15:45:31">
2.6.5 :002 > Topic.create!(title: 'Software Engineering')
   (0.2ms)  BEGIN
  Topic Create (0.4ms)  INSERT INTO "topics" ("title", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["title", "Software Engineering"], ["created_at", "2020-01-24 15:45:44.187637"], ["updated_at", "2020-01-24 15:45:44.187637"]]
   (6.1ms)  COMMIT
 => #<Topic id: 2, title: "Software Engineering", created_at: "2020-01-24 15:45:44", updated_at: "2020-01-24 15:45:44">
 2.6.5 :003 > Topic.all
  Topic Load (0.6ms)  SELECT "topics".* FROM "topics" LIMIT $1  [["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Topic id: 1, title: "Ruby Programming", created_at: "2020-01-24 15:45:31", updated_at: "2020-01-24 15:45:31">, #<Topic id: 2, title: "Software Engineering", created_at: "2020-01-24 15:45:44", updated_at: "2020-01-24 15:45:44">]>
```
Now we can connect our topics with blogs (still in the console) by using `topic_id: Topic.last.id`:
```ruby
2.6.5 :003 > Blog.create!(title: 'Some ruby cool stuff', body: 'blah blah blah blah', topic_id: Topic.last.id)
  Topic Load (0.3ms)  SELECT "topics".* FROM "topics" ORDER BY "topics"."id" DESC LIMIT $1  [["LIMIT", 1]]
   (0.2ms)  BEGIN
  Blog Exists? (0.5ms)  SELECT 1 AS one FROM "blogs" WHERE "blogs"."id" IS NOT NULL AND "blogs"."slug" = $1 LIMIT $2  [["slug", "some-ruby-cool-stuff"], ["LIMIT", 1]]
  Topic Load (0.2ms)  SELECT "topics".* FROM "topics" WHERE "topics"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  Blog Create (37.0ms)  INSERT INTO "blogs" ("title", "body", "created_at", "updated_at", "slug", "topic_id") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"  [["title", "Some ruby cool stuff"], ["body", "blah blah blah blah"], ["created_at", "2020-01-24 15:49:58.308044"], ["updated_at", "2020-01-24 15:49:58.308044"], ["slug", "some-ruby-cool-stuff"], ["topic_id", 2]]
   (0.7ms)  COMMIT
 => #<Blog id: 14, title: "Some ruby cool stuff", body: "blah blah blah blah", created_at: "2020-01-24 15:49:58", updated_at: "2020-01-24 15:49:58", slug: "some-ruby-cool-stuff", status: "draft", topic_id: 2>
```

### 3. Implement Custom Scopes
A custom scope is a custom database query.

First let's update our `db/seeds.rb` to bring in `topics`:
```ruby
# db/seeds.rb
... ...
3.times do |topic| # <- ADD THIS BLOC
  Topic.create!(
      title: "Topic #{topic}"
    )
end

puts "3 Topics created"

10.times do |blog|
  Blog.create!(
    title: "My Blog Post #{blog}",
    body: "Lorem ipsum dolor sit amet......",
    topic_id: Topic.last.id # <- ADD THIS
  )
end
... ...
1.times do |portfolio_item| # <- ADD THIS BLOC
  Portfolio.create!(
    title: "Portfolio title: #{portfolio_item}",
    subtitle: "Vue JS",
    body: "Lorem ipsum dolor sit amet......",
    main_image: "http://placehold.it/600x400",
    thumb_image: "http://placehold.it/350x200" 
  )
end

puts "1 portfolio item created"
```
After the changes, run
```
$ rails db:setup
Database 'DevcampPortfolio_development' already exists
Database 'DevcampPortfolio_test' already exists
3 Topics created
10 blog posts created.
5 skills created
8 portfolio items created
1 portfolio item created
```
Now open up the console to play with the seeds. Note the syntax of `Portfolio.where(subtitle: "Ruby on Rails")` returns all portfolios for Ruby on Rails:
```ruby
$ rails c
2.6.5 :001 > Portfolio.first.subtitle
  Portfolio Load (0.3ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" ASC LIMIT $1  [["LIMIT", 1]]
 => "Ruby on Rails"
2.6.5 :002 > Portfolio.last.subtitle
  Portfolio Load (0.7ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
 => "Vue JS"
2.6.5 :003 > Portfolio.where(subtitle: 'Ruby on Rails')
  Portfolio Load (0.3ms)  SELECT "portfolios".* FROM "portfolios" WHERE "portfolios"."subtitle" = $1 LIMIT $2  [["subtitle", "Ruby on Rails"], ["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Portfolio id: 1, title: "Portfolio title: 0", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 2, title: "Portfolio title: 1", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 3, title: "Portfolio title: 2", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 4, title: "Portfolio title: 3", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 5, title: "Portfolio title: 4", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 6, title: "Portfolio title: 5", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 7, title: "Portfolio title: 6", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">, #<Portfolio id: 8, title: "Portfolio title: 7", subtitle: "Ruby on Rails", body: "Lorem ipsum dolor sit amet, consectetur adipiscing...", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-24 22:07:50", updated_at: "2020-01-24 22:07:50">]>
2.6.5 :004 > Portfolio.where(subtitle: 'Ruby on Rails').count 
(0.5ms)  SELECT COUNT(*) FROM "portfolios" WHERE "portfolios"."subtitle" = $1  [["subtitle", "Ruby on Rails"]]
 => 8
```

Inside our `app/controllers/portfolios_controller.rb`, we can change the `index` function from `@portfolio_items = Portfolio.all` to:
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  def index
    @portfolio_items = Portfolio.where(subtitle: 'Ruby on Rails')
    # or @portfolio_items = Portfolio.where(subtitle: 'Vue JS')
  end
  ...
end
```
This is fine. ***BUT THIS IS NOT THE BEST PRACTICE.*** Instead, it is best to build ***custom scopes***. In fact, `enum` is like a custom scope.

Let's open up `portfolio.rb` and first define a ***class method*** (`self.method_name`):
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  validates_presence_of :title, :body, :main_image, :thumb_image

  def self.vuejs # <- DO THIS CLASS METHOD
    where(subtitle: 'Vue JS')
  end

end
```
Then we call it on the controller:
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  def index
    @portfolio_items = Portfolio.vuejs
  end
  ...
end
```

Alternatively, we can do `scope`:
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  validates_presence_of :title, :body, :main_image, :thumb_image

  # DO THIS BELOW
  scope :ruby_on_rails_portfolio_items, -> { where(subtitle: 'Ruby on Rails') }

end
```
Then we do:
```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  def index
    @portfolio_items = Portfolio.ruby_on_rails_portfolio_items
  end
  ...
end
```
***We are thus forced to keep all the logic for the `model` inside the `model.file`.***

***The controller should only manage data flow and not have data logic in it.***

Finally, let's change the `portfolios_controller.rb`:
```ruby
# app/models/portfolio.rb
class PortfoliosController < ApplicationController
  def index
    @portfolio_items = Portfolio.all # <- CHANGE BACK
  end

  def vuejs # <- ADD THIS
    @vuejs_portfolio_items = Portfolio.vuejs
  end
... ...
end
```
Then we create the `views` for `vuejs`:
```html
<h1>Vue JS Portfolio Items</h1>

<%= link_to "Create New Item", new_portfolio_path %>

<% @vuejs_portfolio_items.each do |portfolio_item| %>
	<p><%= link_to portfolio_item.title, portfolio_show_path(portfolio_item) %></p>
	<p><%= portfolio_item.subtitle %></p>
	<p><%= portfolio_item.body %></p>
	<%= image_tag portfolio_item.thumb_image unless portfolio_item.thumb_image.nil? %>
	<%= link_to "Edit", edit_portfolio_path(portfolio_item) %>
	<%= link_to "Delete Portfolio", portfolio_path(portfolio_item), method: :delete, data: { confirm: "Are you sure?" } %>
<% end %>
```
Then we update the route:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :portfolios, except: [:show]
  get 'vuejs-items', to: 'portfolios#vuejs'
  ... ...
end
```
Then we go to [http://127.0.0.1:3000/vuejs-items](http://127.0.0.1:3000/vuejs-items) and it should work. ***This is not the best way to do it because we have hard-coded item. This is just for demonstration purposes.***


### 4. Set Up Data Defaults
Check out `schema.rb`, when we dealt with `enum`, we already set default values before:
```ruby
...
  create_table "blogs", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "slug"
    t.integer "status", default: 0 # <- Set Default Value
...
```
With `enum`, we know we wont' change the default value anymore. But with other scenarios, we might want to change the default value. ***For these scenarios, we can set the default value not during migration, but inside the model files directly.***

If we do something like:
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  validates_presence_of :title, :body, :main_image, :thumb_image

  def self.vuejs
    where(subtitle: 'Vue JS')
  end

  # -> means a lambda
  scope :ruby_on_rails_portfolio_items, -> { where(subtitle: 'Ruby on Rails') }

  after_initialize :set_defaults # <- ADD THIS

  def set_defaults
    self.main_image ||= "http://placehold.it/600x400"
    self.thumb_image ||= "http://placehold.it/350x200"
  end
end
```
`after_initialize :set_defaults` means that after a `Portfolio` item is initialized (when the `new` action is called like `Portfolio.new` or, not so relevant here, after the `create` action), we want to set its default value. `:set_defaults` *is the name of the method that we need to create.* Now we also implement this `set_defaults` function right below.

Now let's fire up the server, and examine the default images (main and thumb), being shown when we create a new `Portfolio` item:
```
$ rails s
```
Remember two ways to set defaults:

 1. In the migrations (check end result in `schema.rb`
 2. Set in the model

**NOTE:** Using `||=` is a shortcut for saying:
```ruby
if self.main_image == nil
  self.main_image = "http://placehold.it/600x400"
```
If `self.main_image` exists, then the `||=` does not run.

### 5. Integrate Concerns
There is a lot of debate regarding `concerns/` but it could be useful at times. `concerns` can be easily abused by Rails programmers. **`concerns/` should really have to do with data.** Don't do `helper` modules inside `concerns/`. Put helper modules inside `lib/`.

You can find `concerns/` under `app/models/concerns/`. Whenever we have a functionality that does not fully belong inside a model file (such as `app/models/portfolio.rb`) or it should be shared across multiple models, then `concerns/` *might* fit.

Let's do:
```ruby
$ rails g migration add_badge_to_skills badge:text
Running via Spring preloader in process 57353
      invoke  active_record
      create    db/migrate/20200125093759_add_badge_to_skills.rb
$ rails db:migrate
== 20200125093759 AddBadgeToSkills: migrating =================================
-- add_column(:skills, :badge, :text)
   -> 0.0048s
== 20200125093759 AddBadgeToSkills: migrated (0.0049s) ========================
```
Now our `skills` can have images associated with them. Let's do so in our `skill.rb` file:
```ruby
# app/models/skill.rb
class Skill < ApplicationRecord
  validates_presence_of :title, :percent_utilized

  # <- ADD ALL BELOW
  after_initialize :set_defaults

  def set_defaults
    self.badge ||= "http://placehold.it/250x250"
  end

end
```
But we are not happy with `self.badge ||= "http://placehold.it/250x250"` as it might get buggy. So let's create a `concern` about it so the `concern` will generate this text and we get a method for it.

Create a file under `app/models/concerns/` and name it `placeholder.rb`:
```ruby
# app/models/concerns/placeholder.rb
module Placeholder
  extend ActiveSupport::Concern

  # (height:, width:) keeps the order; it's otherwise the same as (height, width)
  def self.image_generator(height:, width:)
    "http://placehold.it/#{height}x#{width}"
  end
end
```
Then we update our models:
```ruby
# app/models/skill.rb
class Skill < ApplicationRecord
  include Placeholder # <- ADD THIS
  validates_presence_of :title, :percent_utilized

  after_initialize :set_defaults

  def set_defaults # <- ADD THIS
    self.badge ||= Placeholder.image_generator(height: 250, width: 250)
  end
end
```
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  include Placeholder
  validates_presence_of :title, :body, :main_image, :thumb_image

  def self.vuejs
    where(subtitle: 'Vue JS')
  end

  # -> means a lambda
  scope :ruby_on_rails_portfolio_items, -> { where(subtitle: 'Ruby on Rails') }

  after_initialize :set_defaults

  def set_defaults
    self.main_image ||= Placeholder.image_generator(height: 600, width: 400)
    self.thumb_image ||= Placeholder.image_generator(height: 350, width: 200)
  end
end
```

Go into console:
```ruby
$ rails c
Running via Spring preloader in process 58055
Loading development environment (Rails 6.0.2.1)
2.6.5 :001 > Skill.create!(title: "Some Skill", percent_utilized: 80)
   (0.1ms)  BEGIN
  Skill Create (1.3ms)  INSERT INTO "skills" ("title", "percent_utilized", "created_at", "updated_at", "badge") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["title", "Some Skill"], ["percent_utilized", 80], ["created_at", "2020-01-25 10:01:36.770872"], ["updated_at", "2020-01-25 10:01:36.770872"], ["badge", "http://placehold.it/250x250"]]
   (0.5ms)  COMMIT
 => #<Skill id: 6, title: "Some Skill", percent_utilized: 80, created_at: "2020-01-25 10:01:36", updated_at: "2020-01-25 10:01:36", badge: "http://placehold.it/250x250">
```
We see the `badge` at the end. It works.


### 6. Build Parent/Child Relationship
```ruby
$ rails g model Technology name:string portfolio:references
Running via Spring preloader in process 59405
      invoke  active_record
      create    db/migrate/20200125111419_create_technologies.rb
      create    app/models/technology.rb
$ rails db:migrate
```

This will give us:
```ruby
# app/models/technology.rb
class Technology < ApplicationRecord
  belongs_to :portfolio
end
```
But we also need to update `Portfolio`:
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  has_many :technologies # <- ADD THIS
  ... code ...
end
```

Inside the console:
```ruby
$ rails c
2.6.5 :001 > Technology.create!(name: "Ruby on Rails", portfolio_id: Portfolio.last.id)
  Portfolio Load (0.4ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
   (0.2ms)  BEGIN
  Portfolio Load (0.2ms)  SELECT "portfolios".* FROM "portfolios" WHERE "portfolios"."id" = $1 LIMIT $2  [["id", 12], ["LIMIT", 1]]
  Technology Create (1.1ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Ruby on Rails"], ["portfolio_id", 12], ["created_at", "2020-01-25 11:23:07.065107"], ["updated_at", "2020-01-25 11:23:07.065107"]]
   (11.0ms)  COMMIT
 => #<Technology id: 1, name: "Ruby on Rails", portfolio_id: 12, created_at: "2020-01-25 11:23:07", updated_at: "2020-01-25 11:23:07">
2.6.5 :002 > Technology.last.portfolio
  Technology Load (0.8ms)  SELECT "technologies".* FROM "technologies" ORDER BY "technologies"."id" DESC LIMIT $1  [["LIMIT", 1]]
  Portfolio Load (0.2ms)  SELECT "portfolios".* FROM "portfolios" WHERE "portfolios"."id" = $1 LIMIT $2  [["id", 12], ["LIMIT", 1]]
 => #<Portfolio id: 12, title: "Placeholder Portfolio", subtitle: "placeholder portfolio", body: "blah blah blah blah blah", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-25 09:57:47", updated_at: "2020-01-25 09:57:47">
2.6.5 :003 > Portfolio.last.technologies
  Portfolio Load (0.8ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
  Technology Load (0.7ms)  SELECT "technologies".* FROM "technologies" WHERE "technologies"."portfolio_id" = $1 LIMIT $2  [["portfolio_id", 12], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Technology id: 1, name: "Ruby on Rails", portfolio_id: 12, created_at: "2020-01-25 11:23:07", updated_at: "2020-01-25 11:23:07">]>
```

Let's also add some seeds:
```ruby
# db/seeds.rb
... code ...
3.times do |technlogy|
  Technology.create!(
    name: "Technology #{technology}",
    portfolio_id: Portfolio.last.id
  )
end

puts "3 technologies created"
```

Back inside the console:
```ruby
2.6.5 :004 > p = Portfolio.last
  Portfolio Load (0.9ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
 => #<Portfolio id: 12, title: "Placeholder Portfolio", subtitle: "placeholder portfolio", body: "blah blah blah blah blah", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-25 09:57:47", updated_at: "2020-01-25 09:57:47">
2.6.5 :005 > p.technologies
  Technology Load (0.6ms)  SELECT "technologies".* FROM "technologies" WHERE "technologies"."portfolio_id" = $1 LIMIT $2  [["portfolio_id", 12], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Technology id: 1, name: "Ruby on Rails", portfolio_id: 12, created_at: "2020-01-25 11:23:07", updated_at: "2020-01-25 11:23:07">]>
2.6.5 :006 > p.technologies.create!(name: "Ruby")
   (0.5ms)  BEGIN
  Technology Create (0.4ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Ruby"], ["portfolio_id", 12], ["created_at", "2020-01-25 11:28:42.102012"], ["updated_at", "2020-01-25 11:28:42.102012"]]
   (5.7ms)  COMMIT
 => #<Technology id: 2, name: "Ruby", portfolio_id: 12, created_at: "2020-01-25 11:28:42", updated_at: "2020-01-25 11:28:42">
2.6.5 :007 > p.technologies
  Technology Load (0.9ms)  SELECT "technologies".* FROM "technologies" WHERE "technologies"."portfolio_id" = $1 LIMIT $2  [["portfolio_id", 12], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Technology id: 1, name: "Ruby on Rails", portfolio_id: 12, created_at: "2020-01-25 11:23:07", updated_at: "2020-01-25 11:23:07">, #<Technology id: 2, name: "Ruby", portfolio_id: 12, created_at: "2020-01-25 11:28:42", updated_at: "2020-01-25 11:28:42">]>
2.6.5 :008 > Technology.all
  Technology Load (0.5ms)  SELECT "technologies".* FROM "technologies" LIMIT $1  [["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Technology id: 1, name: "Ruby on Rails", portfolio_id: 12, created_at: "2020-01-25 11:23:07", updated_at: "2020-01-25 11:23:07">, #<Technology id: 2, name: "Ruby", portfolio_id: 12, created_at: "2020-01-25 11:28:42", updated_at: "2020-01-25 11:28:42">]>
```
***Portfolio owns technologies. We see literally that Rails allows us to create technologies via portfolio.***


So the content in `seeds.rb` can be changed to:
```ruby
# db/seeds.rb
... code ...
3.times do |technology|
  Portfolio.last.technologies.create!(
    name: "Technology #{technology}"
  )
end

puts "3 technologies created"
```

Then we run:
```
$ rails db:setup
```

### 7.1 Rails Complex Forms: Configuring Nested Attributes in Models
Let's do this:
```ruby
# app/models/portfolio.rb
class Portfolio < ApplicationRecord
  has_many :technologies
  accepts_nested_attributes_for :technologies,
                                reject_if: lambda { |attrs| attrs['name'].blank? }
  ... code ...
end
```
Go into the console to test out with this line: `Portfolio.create!(title: "Web app", subtitle: "web app for show", body: "blah blah blah", technologies_attributes: [{name: 'Ruby'}, {name: 'Ruby on Rails'}, {name: 'Vue JS'}, {name: 'React'}])`
```ruby
$ rails c
Running via Spring preloader in process 60866
Loading development environment (Rails 6.0.2.1)
2.6.5 :001 > Portfolio.create!(title: "Web app", subtitle: "web app for show", body: "blah blah blah", technologies_attributes: [{name: 'Ruby'}, {name: 'Ruby on Rails'}, {name: 'Vue JS'}, {name: 'React'}])
   (0.2ms)  BEGIN
  Portfolio Create (6.2ms)  INSERT INTO "portfolios" ("title", "subtitle", "body", "main_image", "thumb_image", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5, $6, $7) RETURNING "id"  [["title", "Web app"], ["subtitle", "web app for show"], ["body", "blah blah blah"], ["main_image", "http://placehold.it/600x400"], ["thumb_image", "http://placehold.it/350x200"], ["created_at", "2020-01-25 12:45:32.908233"], ["updated_at", "2020-01-25 12:45:32.908233"]]
  Technology Create (8.0ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Ruby"], ["portfolio_id", 10], ["created_at", "2020-01-25 12:45:32.919168"], ["updated_at", "2020-01-25 12:45:32.919168"]]
  Technology Create (0.4ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Ruby on Rails"], ["portfolio_id", 10], ["created_at", "2020-01-25 12:45:32.928204"], ["updated_at", "2020-01-25 12:45:32.928204"]]
  Technology Create (0.4ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Vue JS"], ["portfolio_id", 10], ["created_at", "2020-01-25 12:45:32.929586"], ["updated_at", "2020-01-25 12:45:32.929586"]]
  Technology Create (0.4ms)  INSERT INTO "technologies" ("name", "portfolio_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "React"], ["portfolio_id", 10], ["created_at", "2020-01-25 12:45:32.930853"], ["updated_at", "2020-01-25 12:45:32.930853"]]
   (6.6ms)  COMMIT
 => #<Portfolio id: 10, title: "Web app", subtitle: "web app for show", body: "blah blah blah", main_image: "http://placehold.it/600x400", thumb_image: "http://placehold.it/350x200", created_at: "2020-01-25 12:45:32", updated_at: "2020-01-25 12:45:32">
2.6.5 :002 > Portfolio.last.technologies.count
  Portfolio Load (0.3ms)  SELECT "portfolios".* FROM "portfolios" ORDER BY "portfolios"."id" DESC LIMIT $1  [["LIMIT", 1]]
   (0.8ms)  SELECT COUNT(*) FROM "technologies" WHERE "technologies"."portfolio_id" = $1  [["portfolio_id", 10]]
 => 4
```

### 7.2 Rails Complex Forms: Configuring Nested Attributes in Form
Inside the controller, change the following:
```ruby
# app/models/portfolios_controller.rb
... code ...
  def new
    @portfolio_item = Portfolio.new
    3.times { @portfolio_item.technologies.build } # <- EDIT THIS
  end

  def create
    @portfolio_item = Portfolio.new(params.require(:portfolio).permit(:title, :subtitle, :body, technologies_attributes: [:name])) # <- EDIT THIS

    respond_to do |format|
      if @portfolio_item.save
        format.html { redirect_to portfolios_path, notice: 'Your portfolio is live.' }
      else
        format.html { render :new }
      end
    end
  end
... code ...
```

Then we fix our `app/views/portfolios/new.html.erb` form:
```html
...html code...
  <ul>
    <%= form.fields_for :technologies do |technology_form| %>
    <li>
      <%= technology_form.label :name %>
      <%= technology_form.text_field :name %>
    </li>
    <% end %>
  </ul>
...html code...
```
Then we fix our `app/views/portfolios/show.html.erb` form:
```html
...html code...
<h2>Technologies Used:</h2>

<% @portfolio_item.technologies.each do |t| %>
  <p><%= t.name %></p>
<% end %>
...html code...
```



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNzE3ODEzM119
-->