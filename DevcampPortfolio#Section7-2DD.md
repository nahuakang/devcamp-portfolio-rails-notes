# DevCamp Ruby on Rails Notes 8
## Section 7: Models - Deep Dive
### 1. Working w/ SQL & ActiveRecord
#### A. Create the App
Create a brand new Rails app:
```
$ rails new SqlDive -T --database=postgresql
$ git status
$ git add .
$ git commit -m "init"
$ rails db:create
```
#### B. Generate Book Scaffold
Let's attempt to reverse engineer the project in video lectures 84-86. We will use `scaffold` for `Book` so that we get the corresponding `controller` and `views`.
```ruby
$ rails g scaffold Book title:string sales:integer
Running via Spring preloader in process 63633
      invoke  active_record
      create    db/migrate/20200125133324_create_books.rb
      create    app/models/book.rb
      invoke  resource_route
       route    resources :books
      invoke  scaffold_controller
      create    app/controllers/books_controller.rb
      invoke    erb
      create      app/views/books
      create      app/views/books/index.html.erb
      create      app/views/books/edit.html.erb
      create      app/views/books/show.html.erb
      create      app/views/books/new.html.erb
      create      app/views/books/_form.html.erb
      invoke    helper
      create      app/helpers/books_helper.rb
      invoke    jbuilder
      create      app/views/books/index.json.jbuilder
      create      app/views/books/show.json.jbuilder
      create      app/views/books/_book.json.jbuilder
      invoke  assets
      invoke    scss
      create      app/assets/stylesheets/books.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
```
```ruby
$ rails db:migrate
```

#### C. Generate Model Author and Model Genre
We will use `model` for `Author` and `Genre`. But in case we need the `controller` and `views`, we can hard-code them.
```ruby
$ rails g model Author name:string country:string
Running via Spring preloader in process 64211
      invoke  active_record
      create    db/migrate/20200125135715_create_authors.rb
      create    app/models/author.rb
```
```ruby
$ rails db:migrate
```
```ruby
$ rails g model Genre name:string
Running via Spring preloader in process 64340
      invoke  active_record
      create    db/migrate/20200125140226_create_genres.rb
      create    app/models/genre.rb
```
```ruby
$ rails db:migrate
```
#### D. Generate Migrations to add Author and Genre to Books
```ruby
$ rails g migration add_author_to_books author:references
Running via Spring preloader in process 64543
      invoke  active_record
      create    db/migrate/20200125140838_add_author_to_books.rb
```
```ruby
$ rails db:migrate
```
```ruby
$ rails g migration add_genre_to_books genre:references
Running via Spring preloader in process 64697
      invoke  active_record
      create    db/migrate/20200125141416_add_genre_to_books.rb
```
```ruby
$ rails db:migrate
```

#### E. Edit the Models
```ruby
# app/models/book.rb
class Book < ApplicationRecord
  belongs_to :author
  belongs_to :genre
end
```
```ruby
# app/models/author.rb
class Author < ApplicationRecord
  has_many :books
end
```
```ruby
# app/models/genre.rb
class Genre < ApplicationRecord
  has_many :books
end
```

#### F. Create Seeds
```ruby
# db/seeds.rb
vader = Author.create!(name: 'Vader', country: 'Mustafar')
luke = Author.create!(name: 'Luke', country: 'Tatooine')
leia = Author.create!(name: 'Leia', country: 'Alderaan')

fiction = Genre.create!(name: "Fiction")
non_fiction = Genre.create!(name: "Non-Fiction")
biographies = Genre.create!(name: "Biographies")

Book.create!(title: "The Force", genre_id: non_fiction.id, author_id: luke.id, sales: 500)
Book.create!(title: "The Dark Force: My Journey", genre_id: biographies.id, author_id: vader.id, sales: 1000)
Book.create!(title: "Imagine a Republic", genre_id: fiction.id, author_id: leia.id, sales: 5000)
```
Then we run:
```
$ rails db:setup
```

#### G. Edit Index Views
Edit it to how we like it to be:
```html
...html code...
<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Author</th>
      <th>Region</th>
      <th>Genre</th>
      <th>Sales</th>
      <th colspan="5"></th>
    </tr>
  </thead>

  <tbody>
    <% @books.each do |book| %>
      <tr>
        <td><%= book.title %></td>
        <td><%= book.author.name %></td>
        <td><%= book.author.country %></td>
        <td><%= book.genre.name %></td>
        <td><%= book.sales %></td>
        <td><%= link_to 'Show', book %></td>
        <td><%= link_to 'Edit', edit_book_path(book) %></td>
        <td><%= link_to 'Destroy', book, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>
...html code...
```

#### H. find_by_sql
Let's play with what we created in the console a bit, to understand SQL:
```ruby
$ rails c
Running via Spring preloader in process 65730
Loading development environment (Rails 6.0.2.1)
 :001 > Book.all
  Book Load (0.4ms)  SELECT "books".* FROM "books" LIMIT $1  [["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Book id: 1, title: "The Force", sales: 500, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 2, genre_id: 2>, #<Book id: 2, title: "The Dark Force: My Journey", sales: 1000, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 1, genre_id: 3>, #<Book id: 3, title: "Imagine a Republic", sales: 5000, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 3, genre_id: 1>]>
 :002 > Book.all.count
   (1.4ms)  SELECT COUNT(*) FROM "books"
 => 3
  :003 > Book.find_by_sql("SELECT books.* FROM books")
  Book Load (4.6ms)  SELECT books.* FROM books
 => [#<Book id: 1, title: "The Force", sales: 500, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 2, genre_id: 2>, #<Book id: 2, title: "The Dark Force: My Journey", sales: 1000, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 1, genre_id: 3>, #<Book id: 3, title: "Imagine a Republic", sales: 5000, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 3, genre_id: 1>]
```
Note that we can do:
```
Book.find_by_sql("SELECT books.* FROM books")
```
and it's the same as:
```
Book.all
```

#### I. find_by_[name]
We can do specific searches in uncool ways, for instance, by trying to find the book with the title `The Force` using `Book.where(title: 'The Force')`:
```ruby
 :004 > Book.where(title: 'The Force')
  Book Load (0.4ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 LIMIT $2  [["title", "The Force"], ["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Book id: 1, title: "The Force", sales: 500, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 2, genre_id: 2>]>
```
So how to make it better? Will `Book.where(title: 'The Force').author` work? No.

```ruby
 :005 > Book.where(title: 'The Force').author
Traceback (most recent call last):
        1: from (irb):5
  Book Load (1.0ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 LIMIT $2  [["title", "The Force"], ["LIMIT", 11]]
NoMethodError (undefined method `author' for #<Book::ActiveRecord_Relation:0x00007fbb68f23340>)
```

This is because `Book.where(title: 'The Force')` is not a single book but ***a collection of (one) book(s)!*** The class of this query is `ActiveRecord_Relation`.

To make it work, we'd have to do `Book.where(title: 'The Force').first.author`:
```ruby
 :006 > Book.where(title: 'The Force').first.author
  Book Load (1.0ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 ORDER BY "books"."id" ASC LIMIT $2  [["title", "The Force"], ["LIMIT", 1]]
  Author Load (0.3ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
 => #<Author id: 2, name: "Luke", country: "Tatooine", created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42">
```

Take a look at the difference between the two queries, one is `Book::ActiveRecord_Relation`, the other is `Book`:
```ruby
 :007 > Book.where(title: 'The Force').class
 => Book::ActiveRecord_Relation
 :008 > Book.where(title: 'The Force').first.class
  Book Load (0.9ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 ORDER BY "books"."id" ASC LIMIT $2  [["title", "The Force"], ["LIMIT", 1]]
 => Book(id: integer, title: string, sales: integer, created_at: datetime, updated_at: datetime, author_id: integer, genre_id: integer)
```

There is a more dynamic and fun way to grab the book with the title `The Force`:
```ruby
 :009 > Book.find_by_title('The Force')
  Book Load (0.8ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 LIMIT $2  [["title", "The Force"], ["LIMIT", 1]]
 => #<Book id: 1, title: "The Force", sales: 500, created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42", author_id: 2, genre_id: 2>
  :010 > Book.find_by_title('The Force').class
  Book Load (0.8ms)  SELECT "books".* FROM "books" WHERE "books"."title" = $1 LIMIT $2  [["title", "The Force"], ["LIMIT", 1]]
 => Book(id: integer, title: string, sales: integer, created_at: datetime, updated_at: datetime, author_id: integer, genre_id: integer)
```
Indeed, using `Book.find_by_title('The Force')` will return a single `Book`.

#### J.  How does Book.find_by_title method work?
We can't find `find_by_title` method at all in our code repo because Rails creates such dynamic methods by examining our `schema.rb` file.

We can do many search queries, including for instance:
```ruby
 :011 > Author.find_by_country('Tatooine')
  Author Load (0.5ms)  SELECT "authors".* FROM "authors" WHERE "authors"."country" = $1 LIMIT $2  [["country", "Tatooine"], ["LIMIT", 1]]
 => #<Author id: 2, name: "Luke", country: "Tatooine", created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42">
```

However, if you use `find_by_country` and you have multiple authors from `Tatooine`, you will get back ***only a single author***:
```ruby
 :012 > Author.create!(name: 'Biggs', country: 'Tatooine')
   (0.2ms)  BEGIN
  Author Create (6.8ms)  INSERT INTO "authors" ("name", "country", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["name", "Biggs"], ["country", "Tatooine"], ["created_at", "2020-01-25 14:55:28.389135"], ["updated_at", "2020-01-25 14:55:28.389135"]]
   (12.2ms)  COMMIT
 => #<Author id: 4, name: "Biggs", country: "Tatooine", created_at: "2020-01-25 14:55:28", updated_at: "2020-01-25 14:55:28">
  :013 > Author.find_by_country('Tatooine')
  Author Load (0.5ms)  SELECT "authors".* FROM "authors" WHERE "authors"."country" = $1 LIMIT $2  [["country", "Tatooine"], ["LIMIT", 1]]
 => #<Author id: 2, name: "Luke", country: "Tatooine", created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42">
```
This is when we want to do `Author.where(country: 'Tatooine)`:
```ruby
 :014 > Author.where(country: 'Tatooine')
  Author Load (0.4ms)  SELECT "authors".* FROM "authors" WHERE "authors"."country" = $1 LIMIT $2  [["country", "Tatooine"], ["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Author id: 2, name: "Luke", country: "Tatooine", created_at: "2020-01-25 14:29:42", updated_at: "2020-01-25 14:29:42">, #<Author id: 4, name: "Biggs", country: "Tatooine", created_at: "2020-01-25 14:55:28", updated_at: "2020-01-25 14:55:28">]>
```

#### K. See if author exists?
Biggs has no books, so let's see how it works:
```ruby
 :015 > Author.find_by_name("Biggs")
  Author Load (0.3ms)  SELECT "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Biggs"], ["LIMIT", 1]]
 => #<Author id: 4, name: "Biggs", country: "Tatooine", created_at: "2020-01-25 14:58:47", updated_at: "2020-01-25 14:58:47">
 :016 > biggs = Author.find_by_name("Biggs")
  Author Load (1.0ms)  SELECT "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Biggs"], ["LIMIT", 1]]
 => #<Author id: 4, name: "Biggs", country: "Tatooine", created_at: "2020-01-25 14:58:47", updated_at: "2020-01-25 14:58:47">
 :017 > biggs.books
  Book Load (0.4ms)  SELECT "books".* FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 4], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy []>
  :018 > biggs.books.any?
  Book Exists? (1.2ms)  SELECT 1 AS one FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 4], ["LIMIT", 1]]
 => false
```
Calling `biggs.books` returns an empty `ActiveRecord`. Calling `biggs.books.any?` returns `false`.

#### L. Tally an Author's Sales
Update seeds:
```ruby
# db/seeds.rb
vader = Author.create!(name: 'Vader', country: 'Mustafar')
luke = Author.create!(name: 'Luke', country: 'Tatooine')
leia = Author.create!(name: 'Leia', country: 'Alderaan')
biggs = Author.create!(name: 'Biggs', country: 'Tatooine')

fiction = Genre.create!(name: "Fiction")
non_fiction = Genre.create!(name: "Non-Fiction")
biographies = Genre.create!(name: "Biographies")

Book.create!(title: "The Force", genre_id: non_fiction.id, author_id: luke.id, sales: 500)
Book.create!(title: "The Dark Force: My Journey", genre_id: biographies.id, author_id: vader.id, sales: 1000)
Book.create!(title: "How to Create a Deathstar", genre_id: non_fiction.id, author_id: vader.id, sales: 2000)
Book.create!(title: "Imagine a Republic", genre_id: fiction.id, author_id: leia.id, sales: 5000)
```

Let's go into the console again and run `vader.books.sum(:sales)`:
```ruby
$ rails c
 :001 > vader = Author.find_by_name('Vader')
  Author Load (0.3ms)  SELECT "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Vader"], ["LIMIT", 1]]
 => #<Author id: 1, name: "Vader", country: "Mustafar", created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22">
  :002 > vader.books
  Book Load (1.1ms)  SELECT "books".* FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 1], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Book id: 2, title: "The Dark Force: My Journey", sales: 1000, created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22", author_id: 1, genre_id: 3>, #<Book id: 3, title: "How to Create a Deathstar", sales: 2000, created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22", author_id: 1, genre_id: 2>]>
  :003 > vader.books.sum(:sales)
   (1.3ms)  SELECT SUM("books"."sales") FROM "books" WHERE "books"."author_id" = $1  [["author_id", 1]]
 => 3000
```
The SQL code is: `SELECT SUM("books"."sales") FROM "books" WHERE "books"."author_id" = $1  [["author_id", 1]]`.

How about averages?
```ruby
 :004 > Book.average(:sales)
   (10.6ms)  SELECT AVG("books"."sales") FROM "books"
 => 0.2125e4
 :005 > Book.average(:sales).to_f
   (0.5ms)  SELECT AVG("books"."sales") FROM "books"
 => 2125.0
```

How about the most popular book?
```ruby
 :006 > Book.maximum(:sales)
   (0.7ms)  SELECT MAX("books"."sales") FROM "books"
 => 5000
  :007 > Book.order('sales DESC').first
  Book Load (0.8ms)  SELECT "books".* FROM "books" ORDER BY sales DESC LIMIT $1  [["LIMIT", 1]]
 => #<Book id: 4, title: "Imagine a Republic", sales: 5000, created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22", author_id: 3, genre_id: 1>
```

#### M. Run multiple queries and connect tables
To answer the question of which genres has Vader written in, we must get genres to have many authors, and get authors to have many genres.

Okay, a tricky thing here is that we will not run migrations to give `genres` table a `author_id`, nor will we give `authors` table a `genre_id`. This is because while for `books`, each book will have 1 `genre` and 1 `author`, we can't say a `genre` just has one `author` or an `author` has one `genre`.

So we use a `through` so that Rails allows us to go through tables. Even though `author` does not have a `genre_id` nor `genre` has an `author_id` the `books` table has `genre_id` and `author_id`.

So all we do now is:
```ruby
# app/models/genre.rb
class Genre < ApplicationRecord
  has_many :books
  has_many :authors, through: :books
end
```

```ruby
# app/models/author.rb
class Author < ApplicationRecord
  has_many :books
  has_many :genres, through: :books
end
```
This way, we link the models `Author` and `Genre` together through `Book`.

We can literally run `Author.first.genres`, i.e. `vader.genres`:
```ruby
$ rails c
 :001 > vader = Author.find_by_name('Vader')
  Author Load (0.4ms)  SELECT "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Vader"], ["LIMIT", 1]]
 => #<Author id: 1, name: "Vader", country: "Mustafar", created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22">
 :002 > vader.books
  Book Load (0.8ms)  SELECT "books".* FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 1], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Book id: 2, title: "The Dark Force: My Journey", sales: 1000, created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22", author_id: 1, genre_id: 3>, #<Book id: 3, title: "How to Create a Deathstar", sales: 2000, created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22", author_id: 1, genre_id: 2>]>
 :003 > vader.genres
  Genre Load (0.4ms)  SELECT "genres".* FROM "genres" INNER JOIN "books" ON "genres"."id" = "books"."genre_id" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 1], ["LIMIT", 11]]
 => #<ActiveRecord::Associations::CollectionProxy [#<Genre id: 2, name: "Non-Fiction", created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22">, #<Genre id: 3, name: "Biographies", created_at: "2020-01-25 15:04:22", updated_at: "2020-01-25 15:04:22">]>
```

Imagine building an author's profile page and list all the genres this author writes in!

#### N. Using pluck
What if I don't want to query for `ActiveRecord::Relation` but just a simple list? We use `pluck`:
```ruby
 :005 > Genre.pluck(:name)
   (0.6ms)  SELECT "genres"."name" FROM "genres"
 => ["Fiction", "Non-Fiction", "Biographies"]
  :006 > Genre.pluck('name')
   (0.5ms)  SELECT "genres"."name" FROM "genres"
 => ["Fiction", "Non-Fiction", "Biographies"]
  :007 > Author.pluck(:name)
   (0.6ms)  SELECT "authors"."name" FROM "authors"
 => ["Vader", "Luke", "Leia", "Biggs"]
 :008 > Book.pluck(:title)
   (1.2ms)  SELECT "books"."title" FROM "books"
 => ["The Force", "The Dark Force: My Journey", "How to Create a Deathstar", "Imagine a Republic"]
  :010 > vader.books.pluck(:title)
   (0.4ms)  SELECT "books"."title" FROM "books" WHERE "books"."author_id" = $1  [["author_id", 1]]
 => ["The Dark Force: My Journey", "How to Create a Deathstar"]
  :011 > Author.ids
   (0.5ms)  SELECT "authors"."id" FROM "authors"
 => [1, 2, 3, 4]
```

#### O. Use includes
Look at the console output below. Everytime we go to the `view` action, we are running each of these SQL queries.
```ruby
$ rails s
=> Booting Puma
=> Rails 6.0.2.1 application starting in development
=> Run `rails server --help` for more startup options
Puma starting in single mode...
* Version 4.3.1 (ruby 2.6.5-p114), codename: Mysterious Traveller
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://127.0.0.1:3000
* Listening on tcp://[::1]:3000
Use Ctrl-C to stop
Started GET "/books" for 127.0.0.1 at 2020-01-25 16:28:31 +0100
   (6.5ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by BooksController#index as HTML
  Rendering books/index.html.erb within layouts/application
  Book Load (0.8ms)  SELECT "books".* FROM "books"
  ↳ app/views/books/index.html.erb:18
  Author Load (0.9ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:21
  Genre Load (0.3ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:23
  Author Load (0.2ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:21
  Genre Load (0.2ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:23
  CACHE Author Load (0.0ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:21
  CACHE Genre Load (0.0ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:23
  Author Load (0.2ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:21
  Genre Load (0.2ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/views/books/index.html.erb:23
  Rendered books/index.html.erb within layouts/application (Duration: 76.3ms | Allocations: 23049)
[Webpacker] Everything's up-to-date. Nothing to do
Completed 200 OK in 137ms (Views: 101.1ms | ActiveRecord: 20.6ms | Allocations: 32199)
```

That takes a bit more time. So we can use, inside `app/controllers/books_controller.rb`, `@books = Book.includes(:author, :genre)` instead of `@books = Book.all`. This way, we can store the data without querying everything again:
```ruby
# app/controllers/books_controller.rb
...ruby code...
  def index
    @books = Book.includes(:author, :genre) # <- DO THIS INSTEAD
  end
...ruby code...
```
We get the same results but with fewer queries:
```ruby
^[Started GET "/books" for 127.0.0.1 at 2020-01-25 16:33:46 +0100
Processing by BooksController#index as HTML
  Rendering books/index.html.erb within layouts/application
  Book Load (0.3ms)  SELECT "books".* FROM "books"
  ↳ app/views/books/index.html.erb:18
  Author Load (1.1ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" IN ($1, $2, $3)  [["id", 2], ["id", 1], ["id", 3]]
  ↳ app/views/books/index.html.erb:18
  Genre Load (0.3ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" IN ($1, $2, $3)  [["id", 2], ["id", 3], ["id", 1]]
  ↳ app/views/books/index.html.erb:18
  Rendered books/index.html.erb within layouts/application (Duration: 40.5ms | Allocations: 17728)
[Webpacker] Everything's up-to-date. Nothing to do
Completed 200 OK in 56ms (Views: 42.0ms | ActiveRecord: 10.8ms | Allocations: 22659)
```
Much less console output than before :)

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDE4NTY5NDE3LC02NTUyNzgxMDJdfQ==
-->