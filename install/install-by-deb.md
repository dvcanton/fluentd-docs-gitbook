# Installing Fluentd Using deb Package

This article explains how to install the
[td-agent](https://docs.treasuredata.com/articles/td-agent) deb package,
the stable Fluentd distribution package maintained by [Treasure Data, Inc](http://www.treasuredata.com/).


## What is td-agent?

Fluentd is written in Ruby for flexibility, with performance sensitive
parts written in C. However, some users may have difficulty installing
and operating a Ruby daemon.

That's why [Treasure Data, Inc](http://www.treasuredata.com/) is
providing **the stable distribution of Fluentd**, called `td-agent`. The
differences between Fluentd and td-agent can be found
[here](https://www.fluentd.org/faqs).

This installation guide is for td-agent v3/td-agent v4.
td-agent v3/v4 uses fluentd v1 in the core. See [this page](/overview/td-agent-v2-vs-v3-vs-v4.md) for the comparison and supported OS.


## Step 0: Before Installation

Please follow the [Preinstallation Guide](/install/before-install.md) to configure
your OS properly. This will prevent many unnecessary problems.


## Step 1: Install from Apt Repository

It's HIGHLY recommended that you set up **ntpd** on the node to prevent
invalid timestamps in your logs. Please check the [Preinstallation Guide](/install/before-install.md).

Note: If td-agent doesn't support your OS, consider [gem installation](/install/install-by-gem.md) instead.

Note: treasure data doesn't verify debian packages. If you have a problem with debian packages,
send a patch to [omnibus-td-agent repository](https://github.com/treasure-data/omnibus-td-agent) or [td-agent-builder repository](https://github.com/fluent-plugins-nursery/td-agent-builder).

A shell script is provided to automate the installation process for each
version. The shell script registers a new apt repository at
`/etc/apt/sources.list.d/treasure-data.list` and installs the `td-agent`
deb package.

For Ubuntu Focal,

```
# td-agent 4
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-td-agent4.sh | sh
```

For Ubuntu Bionic,

```
# td-agent 4
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent4.sh | sh
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent3.sh | sh
```

For Ubuntu Xenial,

```
# td-agent 4
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent4.sh | sh
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent3.sh | sh
```

For Ubuntu Trusty,

```
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent3.sh | sh
```

For Debian Buster,

```
# td-agent 4
curl -L https://toolbelt.treasuredata.com/sh/install-debian-buster-td-agent4.sh | sh
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-debian-buster-td-agent3.sh | sh
```

For Debian Stretch,

```
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-debian-stretch-td-agent3.sh | sh
```

For Debian Jessie,

```
# td-agent 3
curl -L https://toolbelt.treasuredata.com/sh/install-debian-jessie-td-agent3.sh | sh
```

## Step 2: Launch Daemon


### systemd

The `/lib/systemd/system/td-agent` script is provided to start, stop, or
restart the agent.

```
$ sudo systemctl start td-agent.service
$ sudo systemctl status td-agent.service
● td-agent.service - td-agent: Fluentd based data collector for Treasure Data
   Loaded: loaded (/lib/systemd/system/td-agent.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2017-12-07 15:12:27 PST; 6min ago
     Docs: https://docs.treasuredata.com/articles/td-agent
  Process: 53192 ExecStart = /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid (code = exited, statu
 Main PID: 53198 (fluentd)
   CGroup: /system.slice/td-agent.service
           ├─53198 /opt/td-agent/embedded/bin/ruby /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent
           └─53203 /opt/td-agent/embedded/bin/ruby -Eascii-8bit:ascii-8bit /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /v

Dec 07 15:12:27 ubuntu systemd[1]: Starting td-agent: Fluentd based data collector for Treasure Data...
Dec 07 15:12:27 ubuntu systemd[1]: Started td-agent: Fluentd based data collector for Treasure Data.
```

If you want to customize systemd behaviour, put your `td-agent.service`
into `/etc/systemd/system`

NOTE: In td-agent 4, path is different. `/opt/td-agent/bin` instead of `/opt/td-agent/embedded/bin`

### init.d

This is for non systemd based system.

The `/etc/init.d/td-agent` script is provided to start, stop, or restart
the agent.

```
$ sudo /etc/init.d/td-agent restart
$ sudo /etc/init.d/td-agent status
td-agent (pid  21678) is running...
```

The following commands are supported:

```
$ sudo /etc/init.d/td-agent start
$ sudo /etc/init.d/td-agent stop
$ sudo /etc/init.d/td-agent restart
$ sudo /etc/init.d/td-agent status
```

Please make sure **your configuration file** is located at
`/etc/td-agent/td-agent.conf`.


## Step 3: Post Sample Logs via HTTP

By default, `/etc/td-agent/td-agent.conf` is configured to take logs
from HTTP and route them to stdout (`/var/log/td-agent/td-agent.log`).
You can post sample log records using the curl command.

```
$ curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
```


## Next Steps

You're now ready to collect your real logs using Fluentd. Please see the
following tutorials to learn how to collect your data from various data
sources.

-   Basic Configuration
    -   [Config File](/configuration/config-file.md)
-   Application Logs
    -   [Ruby](/language/ruby.md), [Java](/language/java.md), [Python](/language/python.md), [PHP](/language/php.md),
        [Perl](/language/perl.md), [Node.js](/language/nodejs.md), [Scala](/language/scala.md)
-   Examples
    -   [Store Apache Log into Amazon S3](/guides/apache-to-s3.md)
    -   [Store Apache Log into MongoDB](/guides/apache-to-mongodb.md)
    -   [Data Collection into HDFS](/guides/http-to-hdfs.md)

Please refer to the resources below for further steps.

-   [ChangeLog of td-agent](http://docs.treasuredata.com/articles/td-agent-changelog)
-   [Chef Cookbook](https://github.com/treasure-data/chef-td-agent/)


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
