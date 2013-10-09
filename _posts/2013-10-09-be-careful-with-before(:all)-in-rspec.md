---
layout: post
title: Be careful with before(:all) in RSpec
---

After adding more feature specs about a model, I noticed the test
suite began to fail randomly. It seemed somehow the database did not
get cleaned after some scenarios somehow. Here is the code that I use
to debug this, by the way, `pry` is awesome:

```ruby
config.before(:each) do
  binding.pry unless Node.count == 0
end
```

Wow, after a `before(:all)` block in a spec, there are always some
nodes in the database. Following this [post][] by Avdi Grimm, I have
set up database_cleaner with RSpec to clean up the database. Honestly,
I did not know why I have to use it, but almost every tutorial about
RSpec use it. So I started to investigate, but, the README of
database_cleaner is not really very _friendly_.

## So what the hell is transaction, trunction and deletion?

* [Transaction][]: A work-unit performed within a database in a coherent,
  reliable way and independent of other transactions. Transactions act
  in an __all-or-nothing__ fashion, which means the work-unit is
  either performed in its entirety or has no effect at all. This is
  probably the most frequently used cleaning method and the fastest
  because no changes would be directly committed to the database.
  
* [Truncation][]: `TRUNCATE TABLE` is a SQL statement that removes all
  the data in a table without deleting the table structure.

* Deletion: The simplest to understand, just drop the table then
  recreate it. In SQL this means using the `DROP TABLE` + `CREATE
  TABLE` statements. Without doubt, this strategy is the slowest, but
  in some extreme cases, this is the safest fallback.

## RSpec defaults

When you run `rails generate rspec:install`, the following
configuration is included in the spec_helper.rb:

```ruby
config.use_transactional_fixtures = true
```

The name is kind of misleading, what it really means is "run every
example within a transaction." So why do we need database_cleaner?

Data created within a transaction is invisible to other transactions.
This is usually not an issue, the problem comes when using Capybara
with javascript tests, since the test suite and the underlying browser
server are running in different threads, the data created in the test
suite transaction thread is invisible to the browser.

## database_cleaner to the rescue!

We can solve this problem by managing data with database_cleaner,
let's checkout the configuration Avdi suggested:

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = false

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
```

These configurations instruct database_cleaner still use transaction
for normal RSpec test, for javascript tests, use truncation instead.

Another solution is to force Active Record to share the same
connection between all threads, as demonstrated in this [gist][].
Avdi explained why he does not prefer this solution in the same
[post][].

## database_cleaner with before(:all)

This still does not explain why the `before(:all)` block didn't get
cleaned up, after a quick search in rspec [documentation][], I found
out _transactions are not supported in before(:all) and data created
in before(:all) are not rolled back_. So the previous set clean
strategy won't be able to clean it. You have to clean it manually with
a `after(:all)` block:

```ruby
after(:all) do
  DatabaseCleaner.clean_with(:truncation)
end
```

Finally, the test suite all passed.


[post]: http://devblog.avdi.org/2012/08/31/configuring-database_cleaner-with-rails-rspec-capybara-and-selenium/
[Transaction]: http://en.wikipedia.org/wiki/Database_transaction
[Truncation]: http://en.wikipedia.org/wiki/Truncate_(SQL)
[gist]: https://gist.github.com/josevalim/470808
[documentation]: https://www.relishapp.com/rspec/rspec-rails/docs/transactions
