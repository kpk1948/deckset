# Elasticsearch Reference

## Indices APIs

---

# Postman Shared Collection

<http://goo.gl/Px9jf2>

---

# Indices

---

# Create Indices

```bash
$ curl -XPUT localhost:9200/twitter/

$ curl -XPUT localhost:9200/twitter/ -d '
index :
  number_of_shards : 3
  number_of_replicas : 2
'
```

More about Index Settings => **`Index Modules`**

---

```bash
$ curl -XPUT localhost:9200/twitter/ -d '
{
  "settings" : {
    "index" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 2
    }
  }
}
'
```

or, simply

```bash
$ curl -XPUT localhost:9200/twitter/ -d '
{
  "settings" : {
    "number_of_shards" : 3,
    "number_of_replicas" : 2
  }
}
'
```

---

# Delete Indices

```bash
$ curl -XDELETE localhost:9200/twitter/
```

---

# Indices Exist?

```bash
$ curl -XHEAD localhost:9200/twitter
```

---

# Open/Close Indices

```bash
$ curl -XPOST localhost:9200/twitter/_close

$ curl -XPOST localhost:9200/twitter/_open
```

---

# Update Index Settings

```bash
$ curl -XPUT localhost:9200/twitter/_settings -d '
{
  "index" : {
    "number_of_replicas" : 4
  }
}
'
```

---

# Get Index Settings

```bash
$ curl -XGET localhost:9200/twitter/_settings
```

Supports multiple indices.

```bash
$ curl -XGET localhost:9200/twitter,kimchy/_settings

$ curl -XGET localhost:9200/_all/_settings

$ curl -XGET localhost:9200/2013-*/_settings
```

---

Filter settings with `prefix` and `name` options.

```bash
$ curl -XGET localhost:9200/twitter/_settings?prefix=index.

$ curl -XGET localhost:9200/_all/_settings?prefix=index.routing.allocation.

$ curl -XGET localhost:9200/2013-*/_settings?name=index.merge.*

$ curl -XGET localhost:9200/2013-*/_settings/index.merge.*
```

---

# Mappings

---

# Put Mappings

```bash
$ curl -XPUT localhost:9200/twitter/tweet/_mapping -d '
{
  "tweet" : {
    "properties" : {
      "message" : {"type" : "string", "store" : true }
    }
  }
}
'
```

More about Mappings => **`Mapping`**

---

Supports multi indices.

```bash
$ curl -XPUT localhost:9200/twitter,facebook/tweet/_mapping -d '
{
  "tweet" : {
    "properties" : {
      "message" : {"type" : "string", "store" : true }
    }
  }
}
'
```

---

# Get Mappings

```bash
$ curl -XGET localhost:9200/twitter/tweet/_mapping
```

Also, supports multi indices and types.

```bash
$ curl -XGET localhost:9200/twitter,facebook/_mapping

$ curl -XGET localhost:9200/_all/tweet,post/_mapping
```

---

# Get Field Mappings

```bash
$ curl -XGET localhost:9200/twitter/tweet/_mapping/field/text
```

Of course, also supports multi indices, types and fields.

```bash
$ curl -XGET localhost:9200/twitter,facebook/_mapping/field/message

$ curl -XGET localhost:9200/_all/tweet,post/_mapping/field/message,user.id

$ curl -XGET localhost:9200/_all/tw*/_mapping/field/*.id
```

---

# Types Exist?

```bash
$ curl -XHEAD localhost:9200/twitter/tweet
```

---

# Delete Mappings

```bash
$ curl -XDELETE localhost:9200/twitter/tweet/_mapping
$ curl -XDELETE localhost:9200/twitter/tweet
```

# [fit] Deleting Mapping == Deleting Types

---

# Aliases

---

# Index Aliases

Alias == **`View`** (in RDB)

## Add Aliases

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    { "add" : { "index" : "twitter", "alias" : "alias1" } }
  ]
}
'
```

---

## Get Aliases

```bash
$ curl -XGET localhost:9200/_aliases
```

## Remove Aliases

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    { "remove" : { "index" : "twitter", "alias" : "alias1" } }
  ]
}
'
```

---

## Multi Actions

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    { "remove" : { "index" : "twitter", "alias" : "alias1" } },
    { "add" : { "index" : "twitter", "alias" : "alias2" } }
  ]
}
'

$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    { "add" : { "index" : "twitter", "alias" : "alias1" } },
    { "add" : { "index" : "facebook", "alias" : "alias1" } }
  ]
}
'
```

---

## Filtered Aliases

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    {
      "add" : {
         "index" : "twitter",
         "alias" : "alias2",
         "filter" : { "term" : { "user" : "kimchy" } }
      }
    }
  ]
}
'
```

More about Filters => **`Query DSL`**

---

## Routing

Filter by **`routing values`**

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    {
      "add" : {
         "index" : "twitter",
         "alias" : "alias1",
         "routing" : "1"
      }
    }
  ]
}
'
```

---

**`Different routing values`** for searching and indexing.

```bash
$ curl -XPOST localhost:9200/_aliases -d '
{
  "actions" : [
    {
      "add" : {
         "index" : "twitter",
         "alias" : "alias2",
         "search_routing" : "1,2",
         "index_routing" : "2"
      }
    }
  ]
}
'
```

---

# Analyze

---

# Analyze

Performs the **analysis process on a text** and return the tokens breakdown of the text.

```bash
$ curl -XPOST localhost:9200/_analyze?analyzer=standard -d 'this is a test'
```

```bash
$ curl -XPOST 'localhost:9200/_analyze?tokenizer=keyword&filters=lowercase' \
-d 'this is a test'

$ curl -XPOST 'localhost:9200/_analyze?tokenizer=keyword&token_filters=lowercase
&char_filters=html_strip' -d 'this is a <b>test</b>'
```

```bash
$ curl -XGET localhost:9200/twitter/_analyze?text=this+is+a+test
```

---

# Templates

---

# Index Templates

Index templates allow to define **templates** that will **automatically** be applied to **new indices created**.

---

## Create Templates

```bash
$ curl -XPUT localhost:9200/_template/template_1 -d '
{
  "template" : "te*",
  "settings" : {
    "number_of_shards" : 1
  },
  "mappings" : {
    "type1" : {
      "_source" : { "enabled" : false }
    }
  }
}
'
```

Will be applied to the indices with **`te*`** name pattern.

---

## `{index}` placeholder

```bash
$ curl -XPUT localhost:9200/_template/template_1 -d '
{
  "template" : "te*",
  "settings" : {
    "number_of_shards" : 1
  },
  "aliases" : {
    "alias1" : {},
    "alias2" : {
      "filter" : {
        "term" : {"user" : "kimchy" }
      },
      "routing" : "kimchy"
    },
    "{index}-alias" : {}
  }
}
'
```

---

## Delete Templates

```bash
$ curl -XDELETE localhost:9200/_template/template_1
```

## Get Templates

```bash
$ curl -XGET localhost:9200/_template/template_1
$ curl -XGET localhost:9200/_template/temp*
$ curl -XGET localhost:9200/_template/template_1,template_2
$ curl -XGET localhost:9200/_template/
```

---

## Multiple Template Matching by `order`

```bash
$ curl -XPUT localhost:9200/_template/template_1 -d '
{
  "template" : "*",
  "order" : 0,
  "settings" : {
    "number_of_shards" : 1
  },
  "mappings" : {
    "type1" : {
      "_source" : { "enabled" : false }
    }
  }
}
'

$ curl -XPUT localhost:9200/_template/template_2 -d '
{
  "template" : "te*",
  "order" : 1,
  "settings" : {
    "number_of_shards" : 1
  },
  "mappings" : {
    "type1" : {
      "_source" : { "enabled" : true }
    }
  }
}
'
```

---

## Config

Index templates can also be placed
within **`config/templates`** directory.

---

# Warmers

---

# Warmers

**Index warming** allows to run registered search requests to _warm up the index_ before it is available for search.

Warmup searches typically include requests that require **heavy loading** of data, such as faceting or sorting on specific fields.

---

## Index Creation with Warmers

```bash
$ curl -XPUT localhost:9200/test -d '
{
  "warmers" : {
    "warmer_1" : {
      "types" : [],
      "source" : {
        "query" : {
          "match_all" : {}
        },
        "facets" : {
          "facet_1" : {
            "terms" : {
              "field" : "field"
            }
          }
        }
      }
    }
  }
}
'
```

---

## Put Warmers

```bash
$ curl -XPUT localhost:9200/test/_warmer/warmer_1 -d '
{
  "query" : {
    "match_all" : {}
  },
  "facets" : {
    "facet_1" : {
      "terms" : {
        "field" : "field"
      }
    }
  }
}
'
```

---

## Delete Warmers

```bash
$ curl -XDELETE localhost:9200/test/_warmer/warmer_1
```

## Get Warmers

```bash
$ curl -XGET localhost:9200/test/_warmer/warmer_1
$ curl -XGET localhost:9200/test/_warmer/warm*
$ curl -XGET localhost:9200/test/_warmer/
```

---

# More GETs

---

# Status

The indices status API allows to get a **comprehensive status information** of one or more indices.

```bash
$ curl -XGET localhost:9200/twitter/_status
$ curl -XGET localhost:9200/twitter,kimchy/_status
$ curl -XGET localhost:9200/_status
```

---

# Stats

Indices level stats provide **statistics on different operations** happening on an index.

```bash
$ curl -XGET localhost:9200/twitter/_stats
$ curl -XGET localhost:9200/twitter,kimchy/_stats
$ curl -XGET localhost:9200/_stats
```

---

# Segments

Provides **low level segments information** that a Lucene index (shard level) is built with.

```bash
$ curl -XGET localhost:9200/twitter/_segments
$ curl -XGET localhost:9200/twitter,kimchy/_segments
$ curl -XGET localhost:9200/_segments
```

---

# Recovery

The indices recovery API provides insight into **on-going shard recoveries**. Recovery status may be reported for specific indices, or cluster-wide.

```bash
$ curl -XGET localhost:9200/twitter/_recovery
$ curl -XGET localhost:9200/twitter,kimchy/_recovery
$ curl -XGET localhost:9200/_recovery

$ curl -XGET localhost:9200/twitter/_recovery?detailed=true
```

---

# More POSTs

---

# Clear Cache

The clear cache API allows to **clear either all caches or specific caches** associated with one ore more indices.

```bash
$ curl -XPOST localhost:9200/twitter/_cache/clear
$ curl -XPOST localhost:9200/twitter,kimchy/_cache/clear
$ curl -XPOST localhost:9200/_cache/clear
```

---

## Cache Types

- **`filter`**
- **`field_data`**
- **`id_cache`**

```bash
$ curl -XPOST localhost:9200/twitter/_cache/clear?filter=true
```

cf. The `filter` cache will be cleared within 60 seconds.

---

# Flush

The **flush** process of an index basically **frees memory** from the index by flushing data to the _index storage_ and clearing the internal _transaction log_.

```bash
$ curl -XPOST localhost:9200/twitter/_flush
$ curl -XPOST localhost:9200/twitter,kimchy/_flush
$ curl -XPOST localhost:9200/_flush
```

---

# Refresh

The refresh API allows to explicitly **refresh** one or more index, _making all operations performed_ since the last refresh available for search.

```bash
$ curl -XPOST localhost:9200/twitter/_refresh
$ curl -XPOST localhost:9200/twitter,kimchy/_refresh
$ curl -XPOST localhost:9200/_refresh
```

---

# Optimize

The **optimize** process basically optimizes the index _for faster search_ operations.

The optimize operation allows to _reduce the number of segments_ by merging them.

```bash
$ curl -XPOST localhost:9200/twitter/_optimize
$ curl -XPOST localhost:9200/twitter,kimchy/_optimize
$ curl -XPOST localhost:9200/_optimize
```

---

# Thank You!

## by Daniel Ku (<http://kjunine.net>)
