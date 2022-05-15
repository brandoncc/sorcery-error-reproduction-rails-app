# Sorcery error reproduction app

## Description

Sorcery loads `ActiveRecord` too early, which causes some Rails "defaults"
initializer settings not to properly take effect.

We are working on upgrading a Rails 5 app and the Rails 5.2 update adds
`new_framework_defaults_5_2.rb` with:

```ruby
# Rails.application.config.active_record.cache_versioning = true
```

When that is commented, cache keys look like this:

```ruby
irb(main):001:0> User.last.cache_key
  User Load (0.1ms)  SELECT  "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT ?  [["LIMIT", 1]]
=> "users/1-20220515125125626825"
```

When it is uncommented, it changes the output to this:

```ruby
irb(main):001:0> User.last.cache_key
  User Load (0.1ms)  SELECT  "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT ?  [["LIMIT", 1]]
=> "users/1"
```

When sorcery is added to the project, though, the cache key reverts to the
former version.

If sorcery is commented out in the Gemfile, then the cache key is the correct
value again.

## Reproduction instructions

1. `git clone https://github.com/brandoncc/sorcery-error-reproduction-rails-app.git`
2. `cd sorcery-error-reproduction-rails-app`
3. `bin/rails db:migrate`
4. `bin/rails c`
5. `User.create`
6. `User.last.cache_key`
7. Note the cache key format something like "users/1-20220515125125626825"
8. Open `config/initializers/new_framework_defaults_5_2.rb` and uncomment line 11
9. `bin/rails c`
10. `User.last.cache_key`
11. Note the cache key format now "users/1"
12. Exit the Rails console
13. `bundle add sorcery`
14. `bin/rails c`
15. `User.last.cache_key`
16. Note that the cache key format reverted back to the first format, even though the newer format is still turned on in the initializer
