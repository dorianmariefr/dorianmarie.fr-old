---
title: Job serialization
---

Let's say I have the following job on the default async backend:

```ruby
class TestingSerializationJob < ApplicationJob
  queue_as :default

  def perform(*args)
    p({ args: args })
    puts caller
  end
end
```

Then I put it in the queue with some parameters:

```
TestingSerializationJob.perform_later 1, "2", user: User.first
```

```
Performing TestingSerializationJob (Job ID: f4466f83-31bd-45fb-b452-3357bb3a37d1) from Async(default) enqueued at 2021-09-25T10:19:11Z with arguments: 1, "2", {:user=>#<GlobalID:0x00007fdc096038c0 @uri=#<URI::GID gid://bank/User/1>>}
```

```
{:args=>[1, "2", {:user=>#<User id: 1, name: "Dorian", created_at: "2021-09-25 08:28:52.551275000 +0000", updated_at: "2021-09-25 09:03:05.626925000 +0000">}]}
```

And the first caller is:

```
/Users/dorianmariefr/.rvm/gems/ruby-3.0.2/bundler/gems/rails-014347620db4/activejob/lib/active_job/execution.rb:58:in `block in _perform_job'
```

[`serialize_argument`](https://github.com/rails/rails/blob/014347620db4b063e63caf3914f1737cd87bf5d5/activejob/lib/active_job/arguments.rb#L94-L119) is called to serialize the arguments, and [`deserialize_argument`](https://github.com/rails/rails/blob/014347620db4b063e63caf3914f1737cd87bf5d5/activejob/lib/active_job/arguments.rb#L121-L140) is called to deserialize the arguments. (I link to the methods but the whole file is relevant)

```ruby
      def serialize_argument(argument)
        case argument
        when *PERMITTED_TYPES
          argument
        when GlobalID::Identification
          convert_to_global_id_hash(argument)
        when Array
          argument.map { |arg| serialize_argument(arg) }
        when ActiveSupport::HashWithIndifferentAccess
          serialize_indifferent_hash(argument)
        when Hash
          symbol_keys = argument.each_key.grep(Symbol).map!(&:to_s)
          aj_hash_key = if Hash.ruby2_keywords_hash?(argument)
            RUBY2_KEYWORDS_KEY
          else
            SYMBOL_KEYS_KEY
          end
          result = serialize_hash(argument)
          result[aj_hash_key] = symbol_keys
          result
        when -> (arg) { arg.respond_to?(:permitted?) }
          serialize_indifferent_hash(argument.to_h)
        else
          Serializers.serialize(argument)
        end
      end
```

So first there are all the permitted types:

```
PERMITTED_TYPES = [ NilClass, String, Integer, Float, BigDecimal, TrueClass, FalseClass ]
```

Then all the GlobalID identifiable object, e.g. Active Record models. It will use the `_aj_globalid` special key in the serialized hash.

Then if it's an array, just serialize each, classic.

Then if it's a hash with indiferent access, it will have the special key `_aj_hash_with_indifferent_access` to `true`.

Then more logic about hashes.

Then uses all the serializers like Date, Time, Range, Module, Object, Symbol, Duration, JSON, etc.

Let's try serializing an Object:

```ruby
> TestingSerializationJob.perform_later Object.new
Failed enqueuing TestingSerializationJob to Async(default): ActiveJob::SerializationError (Unsupported argument type: Object)
/Users/dorianmariefr/.rvm/gems/ruby-3.0.2/bundler/gems/rails-014347620db4/activejob/lib/active_job/serializers.rb:31:in `serialize': Unsupported argument type: Object (ActiveJob::SerializationError)
```

Ok, let's try a Class now:

```ruby
irb(main):011:0> Error performing TestingSerializationJob (Job ID: a3855938-77dd-4e3a-b882-35903e995f87) from Async(default) in 1.35ms: ActiveJob::DeserializationError (Error while trying to deserialize arguments: undefined method `constantize' for nil:NilClass):
/Users/dorianmariefr/.rvm/gems/ruby-3.0.2/bundler/gems/rails-014347620db4/activejob/lib/active_job/serializers/module_serializer.rb:11:in `deserialize'
```

Ok it seems that my `Class.new.name` is `nil` so it can't constantize it :) [A little bug](https://github.com/rails/rails/issues/43306)
