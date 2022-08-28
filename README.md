# ConventionalExtensions

ConventionalExtensions autoloads extensions to an object based on a file name convention.

```ruby
# lib/post.rb
class Post < SomeSuperclass
  load_extensions # Will load every Ruby file under `lib/post/extensions`.
end
```

### Defining an extension

Since the loading above happens after the `Post` constant has been defined, we can reopen the class in our extension:

```ruby
# lib/post/extensions/mailroom.rb
class Post # <- Post is reopened here and so there's no superclass mismatch error
  def mailroom
    puts "you've got mail"
  end
end
```

Now, `Post.new.mailroom` works and `Post.instance_method(:mailroom).source_location` still points to the right file and line.

#### Defining a class method in an extension

Since we're reopening the class we can also define class methods directly:

```ruby
# lib/post/extensions/cool.rb
class Post
  def self.cool
    puts "really cool"
  end
end
```

Now, `Post.cool` works and `Post.method(:cool).source_location` still points to the right file and line.

Note, any class method macro extensions are available within the top-level Post definition too:

```ruby
# lib/post.rb
class Post < SomeSuperclass
  load_extensions # Loads the `cool` extension…

  cool # …and now we can invoke the class method macro.
end
```

### Skipping class reopening boilerplate

ConventionalExtensions also supports implicit class reopening so you can skip `class Post`, like so:

```ruby
# lib/post/extensions/mailroom.rb
def mailroom
  puts "you've got mail"
end
```

With this, `Post.new.mailroom` still works and `Post.instance_method(:mailroom).source_location` still points to right file and line.

### Resolve dependencies with a manual `load_extensions`

In case you need to have more fine grained control over the loading, you can call `load_extensions` from within an extension:

```ruby
# lib/post/extensions/mailroom.rb
load_extension :named
named :sup # We're depending on the class method macro from the `named` extension, and hoisting the loading.

def mailroom
  …
end

# lib/post/extensions/named.rb
def self.named(key)
  puts key
end
```

### Supports `# frozen_string_literal: true`

Whether extensions use explicit or implicit class reopening, `# frozen_string_literal: true` is supported.

### Providing a base class that expects ConventionalExtensions loading

In case you're planning on setting up a base class, where you're expecting subclasses to use extensions, you can do this:

```ruby
class BaseClass
  extend ConventionalExtensions.load_on_inherited # This defines the `inherited` method to auto-call `load_extensions`
end

class Subclass < BaseClass
  # No need to write `load_extensions` here
end
```

## A less boilerplate heavy alternative to `ActiveSupport::Concern` for Active Records

Typically, when writing an app domain model with `ActiveSupport::Concern` you end defining an object graph like this:

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  include Cool, Mailroom
end

# app/models/post/cool.rb
module Post::Cool
  extend ActiveSupport::Concern

  class_methods do
    def cool
      puts "really cool"
    end
  end
end

# app/models/post/mailroom.rb
module Post::Mailroom
  extend ActiveSupport::Concern

  included do
    belongs_to :creator, class_name: "User"
  end

  def mailroom
    puts "you've got mail"
  end
end
```

Both the `Post::Cool` and `Post::Mailroom` modules are here immediately loaded (via Zeitwerks file naming conventions) and included. More often than not they're never referred to again, so they're practically implicit modules, yet defined explicitly with a fair bit of DSL on top.

With ConventionalExtensions you'd write this instead:

```ruby
# app/models/post.rb
class Post < ApplicationRecord # ConventionalExtensions automatically loads extensions for Active Record models.
end

# app/models/post/extensions/cool.rb
class Post
  def self.cool
    puts "really cool"
  end
end

# app/models/post/extensions/mailroom.rb
class Post
  belongs_to :creator, class_name: "User"

  def mailroom
    puts "you've got mail"
  end
end
```

## Installation

Install the gem and add to the application's Gemfile by executing:

    $ bundle add conventional_extensions

If bundler is not being used to manage dependencies, install the gem by executing:

    $ gem install conventional_extensions

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/conventional_extensions.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
