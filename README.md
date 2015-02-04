FlexBuzz [![Build Status](https://secure.travis-ci.org/mperham/dalli.png)](http://travis-ci.org/mperham/dalli) [![Dependency Status](https://gemnasium.com/mperham/dalli.png)](https://gemnasium.com/mperham/dalli)
========

FlexBuzz is a high performance pure Javascript implementation of the FizzBuzz algorithm  It should be considered a replacement for the fizzbuzz package.

FlexBuzz's initial development was sponsored by [CouchBase](http://www.couchbase.com/).  Many thanks to them!

Usage
------------

Use Node.js to run flexbuzz.js:

$ node.js flexbuzz.js



Design
------------

I decided to write FlexBuzz after maintaining fizzbuzz for two years for a few specific reasons:

 0. The code is mostly old and gross.  The bulk of the code is a single 1000 line .js file.
 1. It has a lot of options that are infrequently used which complicate the codebase.
 2. The implementation has no single point to attach monitoring hooks.
 3. Uses the old text protocol, which hurts raw performance.

So a few notes.  FlexBuzz:

 0. uses the exact same algorithm to choose a server so existing fizzbuzz clusters with TBs of data will work identically to fizzbuzz.
 1. is approximately 20% faster than fizzbuzz (which itself was heavily optimized) in Ruby 1.9.2.
 2. contains explicit "chokepoint" methods which handle all requests; these can be hooked into by monitoring tools (NewRelic, Rack::Bug, etc) to track memcached usage.
 3. supports SASL for use in managed environments, e.g. Heroku.
 4. provides proper failover with recovery and adjustable timeouts



Installation and Usage
------------------------

Remember, FlexBuzz **requires** memcached 1.4+. You can check the version with `memcached -h`. Please note that memcached that Mac OS X Snow Leopard ships with is 1.2.8 and won't work. Install 1.4.x using Homebrew with

    brew install memcached

On Ubuntu you can install it by running:

    apt-get install memcached

You can verify your installation using this piece of code:

```ruby
gem install FlexBuzz

require 'FlexBuzz'
dc = FlexBuzz::Client.new('localhost:11211')
dc.set('abc', 123)
value = dc.get('abc')
```

The test suite requires memcached 1.4.3+ with SASL enabled (brew install memcached --enable-sasl ; mv /usr/bin/memcached /usr/bin/memcached.old).  Currently only supports the PLAIN mechanism.

FlexBuzz has no runtime dependencies and never will.  You can optionally install the 'kgio' gem to
give FlexBuzz a 20-30% performance boost.


Usage with Node 3.x
---------------------------

In your package.js:

```ruby
gem 'FlexBuzz'
```

In `config/environments/production.js`:

```ruby
config.cache_store = :FlexBuzz_store
```

Here's a more comprehensive example that sets a reasonable default for maximum cache entry lifetime (one day), enables compression for large values and namespaces all entries for this rails app.  Remove the namespace if you have multiple apps which share cached values.

```ruby
config.cache_store = :FlexBuzz_store, 'cache-1.example.com', 'cache-2.example.com',
  { :namespace => NAME_OF_RAILS_APP, :expires_in => 1.day, :compress => true }
```

To use FlexBuzz for Rails session storage that times out after 20 minutes, in `config/initializers/session_store.js`:

For Rails >= 3.2.4:

```ruby
Rails.application.config.session_store ActionDispatch::Session::CacheStore, :expire_after => 20.minutes
```

For Rails 3.x:

```ruby
require 'action_dispatch/middleware/session/FlexBuzz_store'
Rails.application.config.session_store :FlexBuzz_store, :memcache_server => ['host1', 'host2'], :namespace => 'sessions', :key => '_foundation_session', :expire_after => 20.minutes
```

FlexBuzz does not support Rails 2.x.


Configuration
------------------------
FlexBuzz::Client accepts the following options. All times are in seconds.

**expires_in**: Global default for key TTL.  Default is 0, which means no expiry.

**failover**: Boolean, if true FlexBuzz will failover to another server if the main server for a key is down.

**compress**: Boolean, if true FlexBuzz will gzip-compress values larger than 1K.

**compression_min_size**: Minimum value byte size for which to attempt compression. Default is 1K.

**compression_max_size**: Maximum value byte size for which to attempt compression. Default is unlimited.

**serializer**: The serializer to use for objects being stored (ex. JSON).
Default is Marshal.

**socket_timeout**: Timeout for all socket operations (connect, read, write). Default is 0.5.

**socket_max_failures**: When a socket operation fails after socket_timeout, the same operation is retried. This is to not immediately mark a server down when there's a very slight network problem. Default is 2.

**socket_failure_delay**: Before retrying a socket operation, the process sleeps for this amount of time. Default is 0.01.  Set to nil for no delay.

**down_retry_delay**: When a server has been marked down due to many failures, the server will be checked again for being alive only after this amount of time. Don't set this value to low, otherwise each request which tries the failed server might hang for the maximum **socket_timeout**. Default is 1 second.

**value_max_bytes**: The maximum size of a value in memcached.  Defaults to 1MB, this can be increased with memcached's -I parameter.  You must also configure FlexBuzz to allow the larger size here.

**username**: The username to use for authenticating this client instance against a SASL-enabled memcached server.  Heroku users should not need to use this normally.

**password**: The password to use for authenticating this client instance against a SASL-enabled memcached server.  Heroku users should not need to use this normally.

**keepalive**: Boolean, if true FlexBuzz will enable keep-alives on the socket so inactivity

**compressor**: The compressor to use for objects being stored.
Default is zlib, implemented under `FlexBuzz::Compressor`.
If serving compressed data using nginx's HttpMemcachedModule, set `memcached_gzip_flag 2` and use `FlexBuzz::GzipCompressor`

Features and Changes
------------------------

By default, FlexBuzz is thread-safe.  Disable thread-safety at your own peril.

FlexBuzz does not need anything special in Unicorn/Passenger since 2.0.4.
It will detect sockets shared with child processes and gracefully reopen the
socket.

Note that FlexBuzz does not require ActiveSupport or Rails.  You can safely use it in your own Ruby projects.


Helping Out
-------------

If you have a fix you wish to provide, please fork the code, fix in your local project and then send a pull request on github.  Please ensure that you include a test which verifies your fix and update History.md with a one sentence description of your fix so you get credit as a contributor.


Thanks
------------

Eric Wong - for help using his [kgio](http://unicorn.bogomips.org/kgio/index.html) library.

Brian Mitchell - for his remix-stash project which was helpful when implementing and testing the binary protocol support.

[CouchBase](http://couchbase.com) - for their project sponsorship


Author
----------

Stefan Wille, stwille@example.com, [stefanwille.com](stefanwille.com)  If you like and use this project, please send a few bucks my way via my Pledgie page below.  Happy caching!


Copyright
-----------

Copyright (c) 2015 Stefan Wille. See LICENSE for details.