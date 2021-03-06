# Storage Plugin Helper API

`storage` helper manages plugin's internal states.

Here is the code example with `storage` helper:

```
require 'fluent/plugin/input'

module Fluent::Plugin
  class ExampleInput < Input
    Fluent::Plugin.register_input('awesome_example', self)

    # 1. load storage helper
    helpers :storage, :thread

    DEFAULT_STORAGE_TYPE = 'local'

    # omit shutdown and other plugin API

    def initialize
      super
      @storage = nil
    end

    def configure(conf)
      super

      # 2. create storage with unique name
      config = conf.elements(name: 'storage').first
      @storage = storage_create(usage: 'awesome_index', conf: config, default_type: DEFAULT_STORAGE_TYPE)
    end

    def start
      super
      # 3. call storage plugin helpers get/put methods
      @storage.put(:awesome_index, 0) unless @storage.get(:awesome_index)
      thread_create(:awesome_input_runner, &method(:run))
    end

    def run
      while thread_current_running?
        current_time = Time.now.to_i
        break unless (thread_current_running? && Time.now.to_i <= current_time)
        router.emit('awesome', Fluent::Engine.now, generate)
        sleep 0.1
      end
    end

    def generate
      # 4+. update storage plugin helper's storing value
      @storage.update(:awesome_index){|v| v + 1}
    end
  end
end
```

Created storage is managed by the plugin. No need storage shutdown code
in plugin's `shutdown`. The plugin shutdowns created storages
automatically.

For more details about configuration, see [Storage section](/plugins/storage/storage-section.md).


## Methods


### storage\_create(usage: '', type: nil, conf: nil, default\_type: nil)

This method executes storage with given parameters and routine

-   `usage`: unique string value. Default is `''`.
-   `type`: storage plugin type. Default is `nil`.
-   `conf`: storage plugin configuration. Default is `nil`.
-   `default_conf`: storage plugin default configuration. Default is
    `nil`.


### Storage plugin helper instance types

  instance type          Attributes
  ---------------------- --------------------------------------------------
  Raw                    persistent && persistent\_always? `||` otherwise
  Persistent Wrapper     persistent
  Synchronized Wrapper   !synchronized?

#### Raw

Storage plugins will handle as-is.

#### Persistent Wrapper

This wrapper makes handled storage plugin should operate values
persistently.

#### Synchronized Wrapper

This wrapper makes handled storage plugin should operate values
synchronizedly.


### Common methods

Storage plugin helper encapsulates storage plugin implementation.
Normally, the following methods will be called by owner plugin through
this plugin helper.

#### load

This method loads persistently stored value by storage plugin.

#### save

This method save value persistently by storage plugin.

#### get(key)

-   `key`: symbol value.

This method obtains stored value by key.

#### fetch(key, defval)

-   `key`: symbol value.
-   `defval`: default value.

This method obtains stored value by key. If missing, it returns default
value.

#### put(key, value)

-   `key`: symbol value.
-   `value`: store value.

This method update stored value by key.

#### delete(key)

-   `key`: symbol value.

This method delete stored value by key.

#### update(key, &block)

-   `key`: symbol value.
-   `&block`: a Proc object.

This method update stored value by Proc object.


## storage used plugins

-   [in dummy](/plugins/input/dummy.md)
-   [in windows eventlog](/plugins/input/windows_eventlog.md)


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
