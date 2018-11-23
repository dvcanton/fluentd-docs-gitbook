# csv Formatter Plugin

The `csv` formatter plugin output an event as CSV.

``` {.CodeRay}
"value1"[delimiter]"value2"[delimiter]"value3"\n
```


## Parameters

-   [Common Parameters](/articles/plugin-common-parameters.md)
-   [Format section configurations](/articles/format-section.md)


### fields

        type              default         version
  ----------------- -------------------- ---------
   array of string   required parameter   0.14.0

Specify output fields


### delimiter (String, Optional. defaults to ",")

    type    default   version
  -------- --------- ---------
   string      ,      0.14.0

Delimiter for values.

Use `\t` or `TAB` to specify tab character.


### force\_quotes

   type   default   version
  ------ --------- ---------
   bool    true     0.14.0

If false, value won't be framed by quotes.


### add\_newline

   type   default   version
  ------ --------- ---------
   bool    true     0.14.12

Add `\n` to the result.


## Example

``` {.CodeRay}
<format>
  @type csv
  fields host,method
</format>
```

With this configuration:

``` {.CodeRay}
tag:    app.event
time:   1362020400
record: {"host":"192.168.0.1","size":777,"method":"PUT"}
```

This incoming event is formatted to:

``` {.CodeRay}
"192.168.0.1","PUT"\n
```

With `force_quotes false`, the result is:

``` {.CodeRay}
192.168.0.1,PUT\n
```


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.