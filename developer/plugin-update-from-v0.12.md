# Updating plugin for v1 from v0.12

This guide is for plugin authors to show how to update
input/output/filter plugins written for Fluentd v0.12 or earlier.

There are something to be considered (see following "Updating Plugins
Overview" section for details):

-   Plugins which use v0.12 API will be supported in Fluentd v1
    (will be obsoleted at v2).
-   Users can use new features of Fluentd v1 only with plugins using
    new API.
-   Plugins which use new API don't work on Fluentd v0.12.x.

Fluentd core team strongly recommend to use v1 API to make your
plugins stable, consistent and easy to test.


## Updating Plugins Overview

These are steps to update your plugins safely.

1.  release a latest version for Fluentd v0.12.x
2.  update dependency
3.  update code and tests
4.  update CI environments
5.  release the newer version for Fluentd v1 and later


### 1. release a latest version

At first, you should make a git branch named as `v0.12` (if you are
using git for that plugin), and release a latest patch version from that
branch without any changes, except for fixing dependency about
`Fluentd ~> 0.12.0`. This makes you possible to fix bugs and release
newer versions for Fluentd v0.12 users without breaking anything.

-   make a branch for Fluentd v0.12 versions
-   fix dependency about `Fluentd` to `~> 0.12.0` (or later:
    `~> 0.12.26`)
-   bump your gem's version up to next patch version (for example:
    `0.4.1` -\> `0.4.2`)
-   release it to rubygems.org


### 2. update dependency

Following updates are on master branch. You should update dependency in
gemspec at first to depend on Fluentd v1.

-   fix dependency about `Fluentd` to `[">= 1", "< 2"]`
-   execute `bundle install`

Recommended dependency in gemspec:

```
# in gemspec
Gem::Specification.new do |gem|
  gem.name = "fluent-plugin-my_awesome"
  # ...
  gem.add_runtime_dependency "fluentd", [">= 1", "< 2"]
end
```


### 3. update code and tests

There are many difference between plugin types about updating code and
tests. See "Updating Plugin Code" section below for each types of
plugins.

-   update code and tests
-   confirm to run `bundle exec rake test`


### 4. update CI environments

If you have CI configurations like `.travis.yml` and `appvayor.yml`,
these should be updated to support Fluentd v1. Fluentd v1 supports Ruby 2.4 or later.
CI environments should not include Ruby 2.3 or earlier.

-   remove Ruby 2.3 or earlier from CI environments
-   add Ruby 2.4 (or other latest version) to CI environments


### 5. add requirements section

Add requirements section to `README.md` like following:

```
## Requirements

| fluent-plugin-may_awesome | Fluentd    | Ruby   |
|:--------------------------|:-----------|:-------|
| >= 1.0.0                  | >= v1      | >= 2.4 |
| < 1.0.0                   | >= v0.12.0 | >= 2.1 |
```

This helps that plugin users can understand plugin requirements.


### 6. release new version

This is last step. The new version should be major or minor version up,
not patch version up. If the current major version of your gem is equal
or larger than 1, you should bump major version up (e.g., from 1 to 2).
If the current major version is 0, you should bump minor version up
(e.g., from `0.4.2` to `0.5.0`). Then, you can publish a new release
which is available with Fluentd v1.

-   bump the version up
-   release it to rubygems.org


## Updating Plugin Code

For all types of plugins, take care about these things:

-   require files which contains definitions of classes/modules referred
    in your plugin code
-   call `super` in `#initialize`, `#configure`, `#start` and
    `#shutdown`
-   use `router.emit` to emit events into Fluentd instead of
    `Engine.emit`

About updating tests, see "Test code" section for all plugin types.


### Input plugins

For input plugins, points to be fixed are:

-   require "fluent/plugin/input" instead of "fluent/input"
-   fix superclass from `Fluent::Input` to `Fluent::Plugin::Input`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   use `Fluent::Engine.now` or `Fluent::EventTime.now` to create
    current time object instead of `Time.now.to_i`
-   update test code

Plugins will work well only with changes above.

Moreover, most input plugins create threads, timers, network servers
and/or parsers. It's better to use plugin helpers to simplify code and
to make tests stable. For more details, see [Plugin Helper Overview](/developer/plugin-helper-overview.md).


### Filter plugins

For filter plugins, points to be fixed are:

-   require "fluent/plugin/filter" instead of "fluent/filter"
-   fix superclass from `Fluent::Filter` to `Fluent::Plugin::Filter`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   update test code

Plugins will work well only with changes above. But if your plugin
implements `#filter_stream`, remove it if possible. Overriding
`#filter_stream` make it impossible to optimize filters\' performance.

Moreover, many filter plugins uses parsers or formatters. It's better to
use plugin helpers for them to simplify code and make it easy to
understand the way to configure the plugin.


### Non-buffered output plugins

For output plugins (subclass of `Fluent::Output`), points to be fixed
are:

-   require "fluent/plugin/output" instead of "fluent/output"
-   fix superclass from `Fluent::Output` to `Fluent::Plugin::Output`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   remove `#emit` method and implement `#process(tag, es)` method
-   update test code

If your output plugin emits events into Fluentd, follow these points
too:

-   use `event_emitter` plugin helper to introduce router
-   use `Fluent::Engine.now` or `Fluent::EventTime.now` to create
    current time object instead of `Time.now.to_i`

It's recommended to use plugin helpers if your plugin creates any of
thread, timer socket, child process and/or parsers/formatters. It's
better to use plugin helpers to simplify code and to make tests stable.
For more details, see [Plugin Helper Overview](/developer/plugin-helper-overview.md).

Before:

```
require 'fluent/output'

module Fluent
  class SomeOutput < Output
    Fluent::Plugin.register_output('NAME', self)

    def configure(conf)
      super
      ...
    end

    def start
      super
      ...
    end

    def shutdown
      super
      ...
    end

    def emit(tag, es, chain)
      chain.next
      es.each do |time,record|
        log.info "OK!"
      end
    end
  end
end
```

After:

```
require 'fluent/plugin/output'

module Fluent
  module Plugin
    class SomeOutput < Fluent::Plugin::Output
      Fluent::Plugin.register_output('NAME', self)

      helpers :compat_parameters

      def configure(conf)
        compat_parameters_convert(conf, ...)
        super
        ...
      end

      def start
        super
        ...
      end

      def shutdown
        ...
        super # This super must be at the end of shutdown method
      end

      def process(tag, es)
        es.each do |time, record|
          log.info("OK!")
        end
      end
    end
  end
end
```


### Buffered output plugins

For buffered output plugins (subclass of `Fluent::BufferedOutput`),
points to be fixed are:

-   require "fluent/plugin/output" instead of "fluent/output"
-   fix superclass from `Fluent::BufferedOutput` to
    `Fluent::Plugin::Output`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   implement `#compat_parameters_default_chunk_key` to return empty
    string to show chunk key is not specified
-   fix `config_set_default` and its parameter names to override
    parameters in `<buffer>` section
-   remove `#format_stream` method if it is implemented in your plugin
    (it is not supported)
-   update test code

It's recommended to use plugin helpers if your plugin creates any of
thread, timer socket, child process and/or parsers/formatters. It's
better to use plugin helpers to simplify code and to make tests stable.
For more details, see [Plugin Helper Overview](/developer/plugin-helper-overview.md).

Before:

```
require 'fluent/output'

module Fluent
  class SomeOutput < BufferedOutput
    Fluent::Plugin.register_output('NAME', self)

    config_param :path, :string

    def configure(conf)
      super
      ...
    end

    def start
      super
      ...
    end

    def shutdown
      super
      ...
    end

    def format(tag, time, record)
      ...
    end

    def write(chunk)
      data = chunk.read
      print data
    end

    ## Optionally, you can use chunk.msgpack_each to deserialize objects.
    #def write(chunk)
    #  chunk.msgpack_each {|(tag,time,record)|
    #  }
    #end
  end
end
```

After:

```
require 'fluent/plugin/output'

module Fluent
  module Plugin
    class SomeOutput < Fluent::Plugin::Output
      Fluent::Plugin.register_output('NAME', self)

      helpers :compat_parameters

      config_param :path, :string

      def configure(conf)
        compat_parameters_convert(conf, ...)
        super
        ...
      end

      def start
        super
        ...
      end

      def shutdown
        ...
        super # This super must be at the end of shutdown method
      end

      # method for synchronous buffered output mode
      def write(chunk)
      end

      # method for asynchronous buffered output mode
      def try_write(chunk)
      end

      def format(tag, time, record)
        ...
      end
    end
  end
end
```

For more details, see [Writing Buffered Output Plugins](/developer/api-plugin-output.md).


### ObjectBuffered output plugins

For object buffered output plugins (subclass of
`Fluent::ObjectBufferedOutput`), points to be fixed are:

-   require "fluent/plugin/output" instead of "fluent/output"
-   fix superclass from `Fluent::ObjectBufferedOutput` to
    `Fluent::Plugin::Output`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   implement `#compat_parameters_default_chunk_key` to return `"tag"`
    to show chunk key is tag (or something else if your plugin
    overwrites `#emit` to change `key`)
-   fix `config_set_default` and its parameter names to override
    parameters in `<buffer>` section
-   fix `#write` method code not to use `chunk.key`, to use
    `chunk.metadata.tag` and `#extract_placeholders`
-   update test code

It's recommended to use plugin helpers if your plugin creates any of
thread, timer socket, child process and/or parsers/formatters. It's
better to use plugin helpers to simplify code and to make tests stable.
For more details, see [Plugin Helper Overview](/developer/plugin-helper-overview.md).

Before:

```
require 'fluent/output'

module Fluent
  class SomeOutput < ObjectBufferedOutput
    Plugin.register_output('NAME', self)
    # configure(conf), start, shutdown
    ...

    def write(chunk)
      ...
    end
  end
end
```

After: same as buffered output. For more details, see [Writing Buffered Output Plugins](/developer/api-plugin-output.md).


### TimeSliced output plugins

For time sliced output plugins (subclass of `Fluent::TimeSlicedOutput`),
points to be fixed are:

-   require "fluent/plugin/output" instead of "fluent/output"
-   fix superclass from `Fluent::TimeSlicedOutput` to
    `Fluent::Plugin::Output`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   implement `#compat_parameters_default_chunk_key` to return `"time"`
    to show chunk key is time
-   set default value of `timekey` in `<buffer>` section if your plugin
    specifies default `time_slice_format`
-   fix `config_set_default` and its parameter names to override
    parameters in `<buffer>` section
-   fix `#write` method code not to use `chunk.key`, to use
    `chunk.metadata.timekey` and `#extract_placeholders`
-   update test code

It's recommended to use plugin helpers if your plugin creates any of
thread, timer socket, child process and/or parsers/formatters. It's
better to use plugin helpers to simplify code and to make tests stable.
For more details, see [Plugin Helper Overview](/developer/plugin-helper-overview.md).

Before (code):

```
require 'fluent/output'

module Fluent
  class SomeOutput < TimeSlicedOutput
    Plugin.register_output('NAME', self)
    # configure(conf), start, shutdown
    ...

    def write(chunk)
      day = chunk.key
      ...
    end
  end
end
```

Before (configuration):

```
<match *>
  @type ...
  time_slice_format %Y%m%d%H
</match>
```

After (code): same as buffered output. For more details, see [Writing Buffered Output Plugins](/developer/api-plugin-output.md).

After (configuration):

Use `<buffer>` section to customize chunking.

```
<match *>
  @type ...
  <buffer time>
    timekey 1h
  </buffer>
</match>
```

For more details, see [Understanding Chunking and Metadata](/developer/api-plugin-output.md/#understanding-chunking-and-metadata).


### Multi output plugins

For multi output plugins (subclass of `Fluent::MultiOutput`), there are
many points to be considered.

If the plugin uses `<store>` sections and instantiates plugins per each
store section, use `Fluent::Plugin::MultiOutput`. See code to know how
to use it: `lib/fluent/plugin/multi_output.rb` or some built-in plugins
like `out_copy`, `out_roundrobin`.

Otherwise, your plugin does something curious for Fluentd. Read code of
`lib/fluent/plugin/output.rb` and `lib/fluent/plugin/bare_output.rb`,
and consider which is better for your plugin. But it is strongly
unrecommended to use `Fluent::Plugin::BareOutput` for most use cases.


### Output plugins using mixins

#### Fluent::HandleTagAndTimeMixin, Fluent::SetTagKeyMixin, Fluent::SetTimeKeyMixin

Use `inject` and `compat_parameters` plugin helper in plugin code.

Old configuration will be converted to new style configuration
automatically if plugin code uses proper plugin helpers. So plugin users
will not need to rewrite configuration immediately.

Fluentd shows converted new style configuration in startup log if user
provides old style configuration. User can rewrite configuration refer
to the log.

Before:

```
<match **>
  @type some_output
  include_tag_key true
  tag_key tag
  include_time_key true
  time_key time
  time_format %Y-%m-%d
</match>
```

After:

```
<match **>
  @type some_output
  <inject>
    tag_key tag
    time_key time
    time_format %Y-%m-%d
  </inject>
</match>
```

#### Fluent::HandleTagNameMixin

Related configurations:

-   `remove_tag_prefix`
-   `remove_tag_suffix`
-   `add_tag_prefix`
-   `add_tag_suffix`

Use `extract_placeholders(template, chunk)` in plugin code.

Use placeholders `${tag}, ${tag[0]}, ${tag[1]}` in configuration.

Before:

```
<match input.access>
  @type some_output
  remove_tag_prefix input.
  tag               some.${tag}
  <record>
    ...
  </record>
</match>
```

After:

```
<match input.access>
  @type some_output
  tag some.${tag[1]}
  <record>
    ...
  </record>
</match>
```


### Parser plugins

-   require "fluent/plugin/parser" instead of "fluent/parser"
-   fix superclass from `Fluent::Parser` to `Fluent::Plugin::Parser`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   update test code


### Formatter plugins

-   require "fluent/plugin/formatter" instead of "fluent/formatter"
-   fix superclass from `Fluent::Formatter` to
    `Fluent::Plugin::Formatter`
-   use `compat_parameters` plugin helper to keep compatibility of
    configurations to v0.12 style
-   update test code


### Test code

-   organize `test_helper.rb`
-   require "fluent/test/driver/output" and "fluent/test"
-   replace test driver from `Fluent::Test::OutputTestDrive` to
    `Fluent::Test::Driver::Output`
-   use new test driver API

For example, an output plugin's test code. For more details, see
[Writing Plugin Test Code](/developer/plugin-test-code.md)

Before:

test/test\_helper.rb

```
require 'rubygems'
require 'bundler'
begin
  Bundler.setup(:default, :development)
rescue Bundler::BundlerError => e
  $stderr.puts e.message
  $stderr.puts "Run `bundle install` to install missing gems"
  exit e.status_code
end
require 'test/unit'

$LOAD_PATH.unshift(File.join(File.dirname(__FILE__), '..', 'lib'))
$LOAD_PATH.unshift(File.dirname(__FILE__))
require 'fluent/test'
unless ENV.has_key?('VERBOSE')
  nulllogger = Object.new
  nulllogger.instance_eval {|obj|
    def method_missing(method, *args)
      # pass
    end
  }
  $log = nulllogger
end

class Test::Unit::TestCase
end
```

test/plugin/test\_some\_output.rb

```
require 'test_helper'
require 'fluent/plugin/out_some'

class SomeOutputTest < Test::Unit::TestCase
  def create_driver(conf, tag = "test")
    Fluent::Test::OutputTestDriver.new(Fluent::SomeOutput, tag).configure(conf)
  end

  def setup
    Fluent::Test.setup
  end

  def test_configure
    # many configuration related test cases
  end
end
```

After:

test\_helper.rb

```
require "bundler/setup"
require "test/unit"
$LOAD_PATH.unshift(File.join(__dir__, "..", "lib"))
$LOAD_PATH.unshift(__dir__)
require "fluent/test"
```

test/plugin/test\_some\_output.rb

```
require "test_helper"
require "fluent/test/driver/output"
require "fluent/plugin/out_some"

class SomeOutputTest < Test::Unit::TestCase
  def create_driver(conf)
    Fluent::Test::Driver::Output.new(Fluent::Plugin::SomeOutput).configure(conf)
  end

  def setup
    Fluent::Test.setup
  end

  sub_test_case "configure" do
    # configuration related tests goes here
    test "empty" do
      assert_raise(Fluent::ConfigError) do
        create_driver("")
      end
    end
    ...
  end

  sub_test_case "emit events" do
    # emit events related tests goes here
    test "emit 2 simple records" do
      d = create_driver(conf)
      d.run(default_tag: "test") do
        d.feed(time, record1)
        d.feed(time, record2)
      end
      events = d.events
      assert_equal(...)
    end
  end
end
```


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
