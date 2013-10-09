---
layout: post
title: Explore rails console with pry
tags: pry rails console routes helper ruby
---

Before we get started, let's integrate pry with our rails
application. It's pretty simple, just add pry to the development group
of your Gemfile, then `bundle install`.

```ruby
gem 'pry-rails', group: :development
```

Then fire up your `rails console`, when we want to find out what's out
there in Linux shell, we always `ls`, let's do it in pry:

```ruby
[1] pry(main)> ls
Rails::ConsoleMethods#methods: app  controller  helper  new_session  reload!
self.methods: inspect  to_s
locals: _  __  _dir_  _ex_  _file_  _in_  _out_  _pry_  default_command_set
```

### So, what's app?

Let's find out what's that app, just `cd` into it, yeah, you can do
that in pry:

```ruby
[2] pry(main)> cd app
[3] pry(#<ActionDispatch::Integration::Session>):1>
```

Notice how the pry prompt changes? After we cd into app, we'er now in
`ActionDispatch::Integration::Session` directory, no, no,
'context'. Wow, the app method gives us an integration session
instance just like we'er [integration test][] Rails. That means we can
call all the integration test helpers on `app`, such as generating
routes:

```ruby
>> topic_path Topic.first
=> "/topics/1"
# normally, we would call app.topic_path Topic.first, but
# since we'er 'inside app', we can eleminate that.
```

We can also make requests to out application directly in rails
console:

```ruby
>> get "/topics/1"
=> 200
>> response  # to get the response object
=> ...       # output suppressed for simplicity
```

Once again, normally we would call these methods on `app`. Another
trick is, instead of copy paste last commands return value, we can
reference it by `_`, so after we get the topic path, we can just
`get _` to make the requests. By default, pry will print out last
command's return value, you may find it too verbose sometimes, To
suppress the output, add a semicolon after the command: `response;`

### Let's move on to the helper

```
[6] pry(#<ActionDispatch::Integration::Session>):1> cd ../helper
[7] pry(#<ActionView::Base>):1>
```

Now you should feel at home, all the helper methods you love are all
here:

```ruby
>> time_ago_in_words Topic.first.created_at
=> "1 day"
>> number_to_currency(8.88)
=> "$8.88"
# Remember, we'er in the helper context
```

As to more about helper, be sure to check about Nick Quaran's post
[Three quick Rails console tips][]

### What about the controller

```
[10] pry(#<ActionView::Base>):1> cd ../controller
[11] pry(#<ApplicationController>):1> 
```

`controller` just returns a new `ApplicationController` instance,
let's try get the params and flash out:

```ruby
>> params
=> NoMethodError: undefined method `parameters' for nil:NilClass
>> flash
=> RuntimeError: ActionController::Base#flash delegated to request.flash, but request is nil
```

Oops! The problem is the `app`, `helper`, `controller` object is not
hooked together, we have to do it mannually.

```ruby
>> app.get '/'
=> 200
>> controller.request = app.request;
>> helper.controller = controller;
>> controller.flash
=> #<ActionDispatch::Flash::FlashHash:0xb5941b90 @discard=#<Set:{}>, @flashes={}, @now=nil>
>> helper.params
=> { "action" => "index", "controller" => "topics" }
```

### reload! and new_session
`reload!` makes your code changes outside console available to current
session. `new_session` returns a new instance of
`ActionDispatch::Integration::Session`, `app` calls this method
internally, here is the source code:

```ruby
# File railties/lib/rails/console/app.rb, line 8
def app(create=false)
  @app_integration_instance = nil if create
  @app_integration_instance ||= new_session do |sess|
    sess.host! "www.example.com"
  end
end
```

### Resources

* [Railscasts Console Tricks][]
* [Three quick Rails console tips][]


[integration test]: http://guides.rubyonrails.org/testing.html#integration-testing
[Three quick Rails console tips]: http://37signals.com/svn/posts/3176-three-quick-rails-console-tips
[Railscasts Console Tricks]: http://railscasts.com/episodes/48-console-tricks-revised
