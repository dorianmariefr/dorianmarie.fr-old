---
title: Using Packwerk to isolate components in your Rails app
---

I want to write how I would use packwerk because we might use at work (Doctolib) and with the goal to contribute to packwerk's documentation.

- [packwerk](https://github.com/Shopify/packwerk)
- [packwerk's USAGE.md](https://github.com/Shopify/packwerk/blob/main/USAGE.md)

![cohesion](https://i.imgur.com/GlsETV9m.png)

### Why?

Let's say you have Rails app. It's growing fast and it starts to be difficult to know what each controller action is doing for instance.

You might be already doing model and controller concerns, services, factories, etc.

But it's not enough, any part of your code has access to any class in the whole codebase so it starts to be difficult to make changes.

Packwerk will allow you to split your codes into components (packwerk calls them packages).

There will be privacy: the component will only have it's public classes accessible.

There will be dependencies: the component can only use a defined set of public classes.

### Setup

In your `Gemfile`, add:

```ruby
gem "packwerk"
```

Then `bundle install`.

(nothing crazy :) )

Then `bundle exec packwerk init`.

### Generated files

- `/bin/packwerk` is because of the whole spring/bundle exec mess, you might need it, or running `packwerk` might just work fine like it did for me
- `/package.yml` is defining your whole app as a package, you can keep the defaults:

```yaml
enforce_dependencies: true
enforce_privacy: false
```

- `/packwerk.yml` is the packwerk configuration, I added the `components` folder so it's clear which bits of code are migrated but you can define packages in any folder

```yaml
load_paths:
- app/controllers
- app/controllers/concerns
- app/helpers
- app/jobs
- app/mailers
- app/models
- app/models/concerns
- app/services
- components/
```

### Your first package

In my case (I'm testing packwerk on a personal project before trying it at work), I had a service that could easily be a package, called `EventFilter`, it takes an user and a list of events and returns a filtered list of events.

So I created the directories:

```bash
mkdir -p components/event_filter/app/public
```

- `components` is where our components will live
- `components/event_filter` is for the EventFilter component
- `components/event_filter/app/public` is for the public interface of EventFilter

Then create a strict `package.yml` in `components/events_filter/package.yml`:

```yaml
enforce_privacy: true
enforce_dependencies: true
```

Then create `components/event_filter/app/public/event_filter.rb`:

```ruby
class EventFilter
  def initialize(events:, user:)
    @events = events
    @user = user
  end

  def filtered_events
    events =
      @events.includes(
        :attendances,
        category: {
          image_attachment: :blob
        },
        user: {
          image_attachment: :blob
        },
        image_attachment: :blob
      )

    return events if @user.admin?

    events = events.approved

    ...

    events.merge(
      Event
        .not_canceled
        .or(Event.canceled.where(id: @user.attendances.pluck(:event_id)))
        .or(@user.events)
    )
  end
end
```

Then run `packwerk check`.

You will get:

```
(...)

components/events_filter/app/public/event_filter.rb:33:6
Dependency violation: ::Event belongs to '.', but 'components/events_filter' does not specify a dependency on '.'.

(...)
```

It makes sense, because we are using the `Event` class but it's not defined in our dependencies

So, let's fix this with: `packwerk update-deprecations`.

It will add `components/events_filter/deprecated_references.yml` with:

```yaml
---
".":
  "::Event":
    violations:
    - dependency
    files:
    - components/events_filter/app/public/event_filter.rb
```

Now you can run `packwerk check` and everything is good:

```
No offenses detected 🎉
```

We just added a dependency, that could try to remove if it doesn't make sense.

The idea of `packwerk update-deprecations` is to "stop the bleeding" so that you can migrate to isolated components and then work to make them more isolated during refactoring.

### Going further

You can then define multiple components, have dependencies between components, etc.

Your components can have private class files to handle your logic (outside of `app/public`).

Last updated: 2021-03-25
