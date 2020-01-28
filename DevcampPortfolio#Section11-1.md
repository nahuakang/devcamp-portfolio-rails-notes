# DevCamp Ruby on Rails Notes 14
## Section 11: Debugging
Debugging is important for professional developers (and for those who hire them). This section shows a few debugging tools (`byebug`, `pry`, and `Error Management`) to fix bugs in our applications.

### 1. Utilize Puts Debugging in Rails
If inside `blogs_controller.rb` we accidentally have:
```ruby
  @blogs = Blog.limit(2)
```
We can add `puts` below the buggy line:
```ruby
  @blogs = Blog.limit(2)
  puts "*" * 500
  puts @blogs.inpsect
  puts "*" * 500
```
We see the output inside our console much better now. Then we know what kind of data is coming through, and we see that there's a limit statement of 2. So we know we should change it back to.

### 2. Using Byebug






<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQzNDAwNzcwNCw3MTYwNzc3MDEsMTAwNz
YyMjE1NSwtMTk0MTI2MjM3Ml19
-->