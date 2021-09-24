---
title: Batches
---

`find_each` and `find_in_batches` both use `in_batches` and have similar documentation.

[activerecord/lib/active_record/relation/batches.rb](https://github.com/rails/rails/blob/main/activerecord/lib/active_record/relation/batches.rb)

Let's run the following script:

```ruby
gem 'activerecord', git: 'https://github.com/rails/rails'
gem 'sqlite3'

require 'active_record'
require 'logger'

ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: ':memory:')
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
  create_table :users
end

class User < ActiveRecord::Base
end

BATCH_SIZE = 5
USERS_COUNT = 13

User.insert_all((1..USERS_COUNT).map { |id| { id: id } })

User.find_each(batch_size: BATCH_SIZE) do |user|
  puts "find_each #{user.inspect}"
end

User.find_in_batches(batch_size: BATCH_SIZE) do |users|
  puts "find_in_batches #{users.inspect}"
end

User.in_batches(of: BATCH_SIZE) do |users|
  puts "in_batches #{users.inspect}"
end
```

And we get the following output:

```sql
-- create_table(:users)
D, [2021-09-24T16:34:43.383818 #56959] DEBUG -- :    (1.0ms)  SELECT sqlite_version(*)
D, [2021-09-24T16:34:43.384739 #56959] DEBUG -- :    (0.3ms)  CREATE TABLE "users" ("id" integer PRIMARY KEY AUTOINCREMENT NOT NULL)
   -> 0.0172s
D, [2021-09-24T16:34:43.409260 #56959] DEBUG -- :    (0.2ms)  CREATE TABLE "ar_internal_metadata" ("key" varchar NOT NULL PRIMARY KEY, "value" varchar, "created_at" datetime(6) NOT NULL, "updated_at" datetime(6) NOT NULL)
D, [2021-09-24T16:34:43.419863 #56959] DEBUG -- :   ActiveRecord::InternalMetadata Load (1.0ms)  SELECT "ar_internal_metadata".* FROM "ar_internal_metadata" WHERE "ar_internal_metadata"."key" = ? LIMIT ?  [["key", "environment"], ["LIMIT", 1]]
D, [2021-09-24T16:34:43.423701 #56959] DEBUG -- :   TRANSACTION (0.1ms)  begin transaction
D, [2021-09-24T16:34:43.424058 #56959] DEBUG -- :   ActiveRecord::InternalMetadata Create (0.1ms)  INSERT INTO "ar_internal_metadata" ("key", "value", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["key", "environment"], ["value", "default_env"], ["created_at", "2021-09-24 14:34:43.423302"], ["updated_at", "2021-09-24 14:34:43.423302"]]
D, [2021-09-24T16:34:43.424287 #56959] DEBUG -- :   TRANSACTION (0.0ms)  commit transaction
D, [2021-09-24T16:34:43.424791 #56959] DEBUG -- :    (0.1ms)  SELECT sqlite_version(*)
D, [2021-09-24T16:34:43.426155 #56959] DEBUG -- :   User Bulk Insert (0.1ms)  INSERT INTO "users" ("id") VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), (11), (12), (13) ON CONFLICT  DO NOTHING
D, [2021-09-24T16:34:43.426914 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 5]]
find_each #<User id: 1>
find_each #<User id: 2>
find_each #<User id: 3>
find_each #<User id: 4>
find_each #<User id: 5>
D, [2021-09-24T16:34:43.428267 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 5], ["LIMIT", 5]]
find_each #<User id: 6>
find_each #<User id: 7>
find_each #<User id: 8>
find_each #<User id: 9>
find_each #<User id: 10>
D, [2021-09-24T16:34:43.429017 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 10], ["LIMIT", 5]]
find_each #<User id: 11>
find_each #<User id: 12>
find_each #<User id: 13>
D, [2021-09-24T16:34:43.430156 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 5]]
find_in_batches [#<User id: 1>, #<User id: 2>, #<User id: 3>, #<User id: 4>, #<User id: 5>]
D, [2021-09-24T16:34:43.430965 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 5], ["LIMIT", 5]]
find_in_batches [#<User id: 6>, #<User id: 7>, #<User id: 8>, #<User id: 9>, #<User id: 10>]
D, [2021-09-24T16:34:43.431623 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 10], ["LIMIT", 5]]
find_in_batches [#<User id: 11>, #<User id: 12>, #<User id: 13>]
D, [2021-09-24T16:34:43.432500 #56959] DEBUG -- :    (0.1ms)  SELECT "users"."id" FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 5]]
D, [2021-09-24T16:34:43.433135 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (?, ?, ?, ?, ?) /* loading for inspect */ LIMIT ?  [["id", 1], ["id", 2], ["id", 3], ["id", 4], ["id", 5], ["LIMIT", 11]]
in_batches #<ActiveRecord::Relation [#<User id: 1>, #<User id: 2>, #<User id: 3>, #<User id: 4>, #<User id: 5>]>
D, [2021-09-24T16:34:43.433693 #56959] DEBUG -- :    (0.1ms)  SELECT "users"."id" FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 5], ["LIMIT", 5]]
D, [2021-09-24T16:34:43.434219 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (?, ?, ?, ?, ?) /* loading for inspect */ LIMIT ?  [["id", 6], ["id", 7], ["id", 8], ["id", 9], ["id", 10], ["LIMIT", 11]]
in_batches #<ActiveRecord::Relation [#<User id: 6>, #<User id: 7>, #<User id: 8>, #<User id: 9>, #<User id: 10>]>
D, [2021-09-24T16:34:43.434849 #56959] DEBUG -- :    (0.1ms)  SELECT "users"."id" FROM "users" WHERE "users"."id" > ? ORDER BY "users"."id" ASC LIMIT ?  [["id", 10], ["LIMIT", 5]]
D, [2021-09-24T16:34:43.435362 #56959] DEBUG -- :   User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (?, ?, ?) /* loading for inspect */ LIMIT ?  [["id", 11], ["id", 12], ["id", 13], ["LIMIT", 11]]
in_batches #<ActiveRecord::Relation [#<User id: 11>, #<User id: 12>, #<User id: 13>]>
```
