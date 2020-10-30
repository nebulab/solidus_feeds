# Solidus Feeds

[![CircleCI](https://circleci.com/gh/nebulab/solidus_feeds.svg?style=shield)](https://circleci.com/gh/nebulab/solidus_feeds)
[![codecov](https://codecov.io/gh/nebulab/solidus_feeds/branch/master/graph/badge.svg)](https://codecov.io/gh/nebulab/solidus_feeds)

<!-- Explain what your extension does. -->
A framework for producing and publishing feeds on Solidus.

## Installation

Add solidus_feeds to your Gemfile:

```ruby
gem 'solidus_feeds'
```

Bundle your dependencies and run the installation generator:

```shell
bin/rails generate solidus_feeds:install
```

## Usage

<!-- Explain how to use your extension once it's been installed. -->

⚠️ *This is work in progress, once done the following is expected to be the final interface* ⚠️

Define and publish your own feeds

```ruby
# initializer/spree.rb

SolidusFeeds.register :google_merchant_shoes do |feed|
  taxon = Spree::Taxon.find_by(name: "Shoes")
  products = Spree::Product.available.in_taxon(taxon)

  feed.publisher = SolidusFeeds::S3Publisher.new(credentials: …,filename: …)
  feed.generator = SolidusFeeds::GoogleMerchantXMLGenerator.new(products)
end
```

Both the generator and the publisher are expected to respond to `#call`.
The publisher's `#call` method is expected to yield an IO-like object that responds to `#<<`.

At this point the feed can be generated and published using:

```ruby
SolidusFeeds.find(:google_merchant_shoes).publish
```

### Serving the feed from the products controller (legacy)

Support the legacy behavior of `solidus_product_feed` by prepending the product controller decorator.

```ruby
# initializer/spree.rb

Rails.application.config.to_prepare {
  ::Spree::ProductsController.prepend SolidusProductFeed::Spree::ProductsControllerDecorator
}
```

### Serving the feed from an external object storage

Define a background job for your feed

```ruby
class FeedPublishingJob < ApplicationJob
  def perform(feed_name)
    SolidusFeeds.publish(feed_name)
  end
end
```

And then periodically run it to keep the data fresh

1. Manually from the backend dashboard
2. With cron or other scheduling mechanism provided by the server (e.g. Heroku scheduler)
3. With a webhook
4. After specific solidus events

### Publishing backends

- S3
- ActiveStorage
- Rails cache
- Static file
- FTP

### Builtin Generators

- GoogleMerchantXML (also works for Facebook and Instagram)

## Development

### Testing the extension

First bundle your dependencies, then run `bin/rake`. `bin/rake` will default to building the dummy
app if it does not exist, then it will run specs. The dummy app can be regenerated by using
`bin/rake extension:test_app`.

```shell
bin/rake
```

To run [Rubocop](https://github.com/bbatsov/rubocop) static code analysis run

```shell
bundle exec rubocop
```

When testing your application's integration with this extension you may use its factories.
Simply add this require statement to your spec_helper:

```ruby
require 'solidus_feeds/factories'
```

### Running the sandbox

To run this extension in a sandboxed Solidus application, you can run `bin/sandbox`. The path for
the sandbox app is `./sandbox` and `bin/rails` will forward any Rails commands to
`sandbox/bin/rails`.

Here's an example:

```
$ bin/rails server
=> Booting Puma
=> Rails 6.0.2.1 application starting in development
* Listening on tcp://127.0.0.1:3000
Use Ctrl-C to stop
```

### Updating the changelog

Before and after releases the changelog should be updated to reflect the up-to-date status of
the project:

```shell
bin/rake changelog
git add CHANGELOG.md
git commit -m "Update the changelog"
```

### Releasing new versions

Please refer to the dedicated [page](https://github.com/solidusio/solidus/wiki/How-to-release-extensions) on Solidus wiki.

## License

Copyright (c) 2020 [name of extension author], released under the New BSD License.
