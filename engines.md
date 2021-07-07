---
title: Engines
---

- [Getting Started with Engines](https://guides.rubyonrails.org/engines.html) (rails guide)
- [Rails::Engine < Railtie](https://edgeapi.rubyonrails.org/classes/Rails/Engine.html) (Rails::Engine documentation)
- A recent (rails 6) 3 part serie:
  - [Create Your First Rails Engine](https://www.hocnest.com/blog/create-user-rails-engine/)
  - [Test Your First Rails Engine](https://www.hocnest.com/blog/testing-modular-monolith-engines/)
  - [Configure Your First Rails Engine](https://www.hocnest.com/blog/configure-rails-engine/)

Let's say we have a simple app with only users:

```bash
bin/rails generate scaffold user name
```

Then `bin/rails db:migrate`

Now you can launch your `bin/rails server` and go to [localhost:3000/users](http://localhost:3000/users).

Let's create an engine to handle payments for our app:

```bash
bin/rails plugin new payments --mountable
```

We need to clean and fill a bit the `gemspec` of the engine (`payments/payments.gemspec`).

Let's have something like:

```ruby
require_relative "lib/payments/version"

Gem::Specification.new do |spec|
  spec.name = "payments"
  spec.version = Payments::VERSION
  spec.authors = ["Dorian Marié"]
  spec.email = ["dorian.marie@doctolib.com"]
  spec.license = "MIT"
  spec.summary = "handles payments"
  spec.files = Dir["{app,config,db,lib}/**/*", "MIT-LICENSE", "Rakefile", "README.md"]

  spec.add_dependency "rails", "~> 6.1.3", ">= 6.1.3.2"
end
```

We will need to add it to our `config/routes.rb`

```ruby
mount Payments::Engine => "/payments"
```

In the `payments/` directory:

```
bin/rails generate scaffold payment user:references amount:integer
```

Then the migration needs to be copied to the main app:

```
rake payments:install:migrations
```

(this is just an example, amount should probably be in cents and have a currency)

Then `bin/rails db:migrate`

(I needed to do `gem update --system`)

Now you need to link the engine css from the main app, in `app/assets/stylesheets/application.css`:

```
 *= link payments/application.css
```

And you can go to [localhost:3000/payments/payments](https://localhost:3000/payments/payments)
