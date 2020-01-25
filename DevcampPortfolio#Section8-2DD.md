# DevCamp Ruby on Rails Notes 10
## Section 8: Deep Dive into Bcrypt
### [Pry](https://github.com/pry/pry)
`Pry` is similar to `irb` but better for professional development.

Do
```
$ gem install pry
```

### [Bcrypt](https://github.com/codahale/bcrypt-ruby)

Do
```
$ gem install bcrypt
```

Now we can do 
```ruby
$ pry
[1] pry(main)> require 'bcrypt'
=> true
[2] pry(main)> ssn = BCrypt::Password.create("5555555555")
=> "$2a$12$8B/EZVcayCpW.fsNiLhoQuZ6aMYT.Vo0vw1YwyAruIy7tMlajfYT."
[3] pry(main)> ssn # NOW ENCRYPTED
=> "$2a$12$8B/EZVcayCpW.fsNiLhoQuZ6aMYT.Vo0vw1YwyAruIy7tMlajfYT."
[4] pry(main)> ssn == "5555555555"
=> true
[5] pry(main)> ssn == "5555555556"
=> false
[6] pry(main)> phone = BCrypt::Password.create('555-555-5555', cost: 4) # highest cost: 10
=> "$2a$04$dhQH6KChb0ZzKsMI889eEOjKMjUA/m4LZWA.UU6oKNQ2i0dZbtzby"
[7] pry(main)> rand 100
=> 15
[8] pry(main)> rand 100
=> 31
[9] pry(main)> rand 100
=> 51
[10] pry(main)> salt = BCrypt::Engine.generate_salt
=> "$2a$12$EL/z0qd/vH9ET.SKIykNGO"
```




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcxNjUyMTQ2M119
-->