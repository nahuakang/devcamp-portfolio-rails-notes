# DevCamp Ruby on Rails Notes 15
## Section 12: RubyGems
Check out again:
```
$ git checkout -b rubygem
```
### 1. Install Bootstrap 4 into Rails
We can grab Bootstrap CDN, so why do we install the `bootstrap` gem? Because it's bad practice. Sometimes one of the links will be down on service and we will lose all the styling on our website and we'll be offering bad user experience (even if just for a few hours). Meanwhile, the CDN links are basically URLs linking to files. 

Using Ruby gems, we can encapsulate the Javascript/CSS of Bootstrap and keep in our app repo for *offline mode*. Also, by encapsulating the data, if Bootstrap decides to make a new class name, we will not be affected by such changes.

To remove changes to a file:
```
$ git status
$ git checkout [file-which-we-want-to-remove-changes]
```
Read more about bootstrap-rubygem here: [https://github.com/twbs/bootstrap-rubygem/blob/master/README.md](https://github.com/twbs/bootstrap-rubygem/blob/master/README.md)

Paste the gem into `Gemfile`:
```ruby
...gems...
# Add bootstrap for Udemy lecture 123
gem 'bootstrap', '~> 4.4.1'
...gems...
```
Then inside our terminal hit:
```
$ bundle install
```
This way we get the whole set of the Bootstrap library and put them in the background of the application. If we really want to overwrite some `gem`, we go to `config/initializers` to do so (such as for `devise`).

Import Bootstrap styles in  `app/assets/stylesheets/application.scss`:
```css
// Custom bootstrap variables must be set or imported *before* bootstrap.
@import "bootstrap";
```

Then make sure to rename `app/assets/stylesheets/application.css` to  `app/assets/application.scss`. Otherwise it won't work.

Delete all the layouts' `scss` code we have written so far

 - `blogs.scss`
 - `pages.scss`
 - `portfolios.scss`

 and add:
```css
@import "bootstrap";
```

Then addd Bootstrap dependencies and Bootstrap to your  `app/javascript/packs/application.js`:
```
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

### 2. Build & Publish Your Own Custom Ruby Gem
We'll build a gem that outputs our copyrights' details. Usually we'll just use a `view helper` to achieve this, but here we'll use it as a way to learn how to build our own Ruby gem.

#### A. First Small Step: Create a Module in ApplicationController
Start in `application_controller.rb` to build out a `module`:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseWhitelist
  include SetSource
  include CurrentUserConcern
  include DefaultPageContent

  before_action :set_copyright

  def set_copyright
    @copyright = KangViewTool::Renderer.copyright "Nahua Kang", "All rights reserved"
  end
end

module KangViewTool
  class Renderer
    def self.copyright name, msg
      "&copy; #{Time.now.year} | <b>#{name}</b> #{msg}".html_safe
    end
  end
end
```
Then include in `app/views/layouts/application.html.erb`:
```html
...html code...
  <%= yield %>
  <%= @copyright %>
...html code...
```
Then, on all the HTML pages that inherit from the layout of `application.html.erb`, we'll see
```
© 2020 | **Nahua Kang** All rights reserved
```
The year will be constantly updated by year!

#### B. Second Small Step: Build the RubyGem from Scratch
First we open a new tab on the terminal and move out of our `DevcampPortfolio` directory to a new one, say: 

```
$ cd projects/rails/
$ mkdir gems
$ cd gems
```

Then, we choose the correct version of Ruby we want to use:
```
$ rvm use 2.6.5
```

Then we do bundle according to the module name we want (`kang_view_tool` in this case):
```
$ bundle gem kang_view_tool
$ cd kang_view_tool
$ subl .
```
Git is already initialized for this repo. Take a look at `kang_view_tool.gemspec` to change some of the specs (w/ `TODO:`):
```ruby
require_relative 'lib/kang_view_tool/version'

Gem::Specification.new do |spec|
  spec.name          = "kang_view_tool"
  spec.version       = KangViewTool::VERSION
  spec.authors       = ["nahuakang"]
  spec.email         = ["kangnahua@gmail.com"]

  spec.summary       = %q{Various view specific methods for application I use.}
  spec.description   = %q{Provides generated HTML data for Rails applications.}
  spec.homepage      = "https://nahua.dev"
  spec.license       = "MIT"
  spec.required_ruby_version = Gem::Requirement.new(">= 2.3.0")

  spec.metadata["allowed_push_host"] = "TODO: Set to 'http://mygemserver.com'"

  spec.metadata["homepage_uri"] = spec.homepage
  spec.metadata["source_code_uri"] = "TODO: Put your gem's public repo URL here."
  spec.metadata["changelog_uri"] = "TODO: Put your gem's CHANGELOG.md URL here."

  # Specify which files should be added to the gem when it is released.
  # The `git ls-files -z` loads the files in the RubyGem that have been added into git.
  spec.files         = Dir.chdir(File.expand_path('..', __FILE__)) do
    `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  end
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]
end
```

Then inside `lib/kang_view_tool.rb` add and cut out:
```ruby
require "kang_view_tool/version"
require "kang_view_tool/renderer" # <- ADD THIS

module KangViewTool # <- CUT OUT TO kang_view_tool/renderer.rb
  class Error < StandardError; end
  # Your code goes here...
end
```
Create `kang_view_tool/renderer.rb` and add:
```ruby
module KangViewTool
  class Error < StandardError; end
  class Renderer
    def self.copyright name, msg
      "&copy; #{Time.now.year} | <b>#{name}</b> #{msg}".html_safe
    end
  end
end
```
Now we need to create a repo:
```
$ git status
$ git add .
$ git commit -m "kang_view_tool init"
```
Go to Github to create the repo `kang_view_tool` and then in the terminal:
```
$ git remote add origin git@github.com:nahuakang/kang_view_tool.git
$ git push -u origin master
$ echo "*.gem" >> .gitignore
$ git status
$ git add .
$ git commit -m "updated gitignore file and readme"
```

Now, we are going to run the following to generate the gem:
```
$ gem build kang_view_tool.gemspec
  Successfully built RubyGem
  Name: kang_view_tool
  Version: 0.1.0
  File: kang_view_tool-0.1.0.gem
```
**Now, `gems` not on ruby-gems but on Github can be used already in our Rails app.**

To use it, inside the `Gemfile` do:
```ruby
# Use our own gem on Github
gem 'kang_view_tool', git: 'https://github.com/nahuakang/kang_view_tool'
```
Back in our terminal, if we run `bundle install`, `bundler` will grab the new gem:
```
$ bundle install
$ rails s
```
Check homepage, yes, we still have the copyright line! We are now pulling from a Ruby gem code on Github!

Now, refactor by deleting everything related to this gem out of `application_controller.rb` and inside `app/helpers/application_helper/rb`, do:
```ruby
# app/helpers/application_helper/rb
...ruby code...
  def copyright_generator
    @copyright = KangViewTool::Renderer.copyright "Nahua Kang", "All rights reserved"
  end
...ruby code...
```
Back inside the layout `application.html.erb`, remove
```html
<%= @copyright %>
```
and instead put in:
```html
<%= copyright_generator %>
```

The reason we move it out of `ApplicationController` is that we want helpers handling data to be in the Controller, and the rest to be in the `view helpers`.

#### C. Publish the Ruby Gem for the world
Register, confirm email, and edit profile in [https://rubygems.org](https://rubygems.org/). Sign in with RubyGems on terminal.

Back in the gem's directory, do:
```
$ bundle exec rake release
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0ODAzMTU0NzYsLTI5NDMxODYyMiwtMT
A1NDg3Njc0MCwxMDc3NzQ1NDIzLDc3NDAyNTUzOSw4MTk4Mzc4
ODksMTEwNTc3MzcxOCwxMDg4NDYzNDQwLC0xNTgwNTI4MTE1LD
E1Nzk4Mzc3MTBdfQ==
-->