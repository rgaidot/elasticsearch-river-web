Elasticsearch River Web
=======================

## Overview

Elasticsearch River Web Plugin is a web crawler for Elasticsearch.
This plugin provides a feature to crawl web sites and extract the content by CSS Query.

## Version

| River Web | elasticsearch |
|:---------:|:-------------:|
| master    | 0.90.x        |
| 1.0.1     | 0.90.7        |
| 1.0.0     | 0.90.5        |


## Installation

### Install Quartz Plugin

River Web plugin depends on Quartz plugin. 
[Quartz plugin](https://github.com/codelibs/elasticsearch-quartz) needs to be installed before installing River Web plugin.

    $ $ES_HOME/bin/plugin -install org.codelibs/elasticsearch-quartz/1.0.0

### Install River Web Plugin

    $ $ES_HOME/bin/plugin -install org.codelibs/elasticsearch-river-web/1.0.1

### Create Index For Crawling

River Web Plugin needs 'robot' index for web crawling.
Therefore, you need to create it before starting the crawl.
Type the following commands to create 'robot' index:

    $ curl -XPUT 'localhost:9200/robot/'

## Usage

### Create Index To Store Crawl Data

An index is needed to store crawl data before starting a river.
For example, to store data to "webindex", create it as below:

    $ curl -XPUT 'localhost:9200/webindex'

and then create a mapping setting if using "overwrite" option:

    $ curl -XPUT "localhost:9200/compassion_test/my_web/_mapping" -d '
    {
      "my_web" : {
        "dynamic_templates" : [
          {
            "url" : {
              "match" : "url",
              "mapping" : {
                "type" : "string",
                "store" : "yes",
                "index" : "not_analyzed"
              }
            }
          },
          {
            "method" : {
              "match" : "method",
              "mapping" : {
                "type" : "string",
                "store" : "yes",
                "index" : "not_analyzed"
              }
            }
          },
          {
            "charSet" : {
              "match" : "charSet",
              "mapping" : {
                "type" : "string",
                "store" : "yes",
                "index" : "not_analyzed"
              }
            }
          },
          {
            "mimeType" : {
              "match" : "mimeType",
              "mapping" : {
                "type" : "string",
                "store" : "yes",
                "index" : "not_analyzed"
              }
            }
          }
        ]
      }
    }'

"my\_web" is a type given by your river name or "crawl.type".

### Register Crawl Data

A crawling configuration is created by registering a river as below.
This example crawls sites of http://www.codelibs.org/ and http://fess.codelibs.org/ at 6:00am.

    $ curl -XPUT 'localhost:9200/_river/my_web/_meta' -d '{
        "type" : "web",
        "crawl" : {
            "index" : "webindex",
            "url" : ["http://www.codelibs.org/", "http://fess.codelibs.org/"],
            "includeFilter" : ["http://www.codelibs.org/.*", "http://fess.codelibs.org/.*"],
            "maxDepth" : 3,
            "maxAccessCount" : 100,
            "numOfThread" : 5,
            "interval" : 1000,
            "target" : [
              {
                "pattern" : {
                  "url" : "http://www.codelibs.org/.*",
                  "mimeType" : "text/html"
                },
                "properties" : {
                  "title" : {
                    "text" : "title"
                  },
                  "body" : {
                    "text" : "body"
                  },
                  "bodyAsHtml" : {
                    "html" : "body"
                  },
                  "projects" : {
                    "text" : "ul.nav-list li a",
                    "isArray" : true
                  }
                }
              },
              {
                "pattern" : {
                  "url" : "http://fess.codelibs.org/.*",
                  "mimeType" : "text/html"
                },
                "properties" : {
                  "title" : {
                    "text" : "title"
                  },
                  "body" : {
                    "text" : "body",
                    "trimSpaces" : true
                  },
                  "menus" : {
                    "text" : "ul.nav-list li a",
                    "isArray" : true
                  }
                }
              }
            ]
        },
        "schedule" : {
            "cron" : "0 0 6 * * ?"
        }
    }'

"my\_web" is a configuration name for River, and you can replace it with one you want.

The configuration is:

| Property                          | Type    | Description                                     |
|:----------------------------------|:-------:|:------------------------------------------------|
| crawl.index                       | string  | Stored index name.                              |
| crawl.type                        | string  | Stored type name.                               |
| crawl.url                         | array   | Start point of URL for crawling.                |
| crawl.includeFilter               | array   | White list of URL for crawling.                 |
| crawl.excludeFilter               | array   | Black list of URL for crawling.                 |
| crawl.maxDepth                    | int     | Depth of crawling documents.                    |
| crawl.maxAccessCount              | int     | The number of crawling documents.               |
| crawl.numOfThread                 | int     | The number of crawler threads.                  |
| crawl.interval                    | int     | Interval time (ms) to crawl documents.          |
| crawl.incremental                 | boolean | Incremental crawling.                           |
| crawl.overwrite                   | boolean | Delete documents of old duplicated url.         |
| crawl.authentications             | object  | Specify BASIC/DIGEST/NTLM authentication info.  |
| crawl.target.urlPattern           | string  | URL pattern to extract contents by CSS Query.   |
| crawl.target.properties.name      | string  | "name" is used as a property name in the index. |
| crawl.target.properties.name.text | string  | CSS Query for the property value.               |
| crawl.target.properties.name.html | string  | CSS Query for the property value.               |
| schedule.cron                     | string  | [Cron format](http://quartz-scheduler.org/api/2.2.0/org/quartz/CronExpression.html) to start a crawler.                 |


### Unregister Crawl Data

If you want to stop the crawler, type as below: (replace my\_web with your river name)

    $ curl -XDELETE 'localhost:9200/_river/my_web/'

## Examples

### Full Text Search for Your site (ex. http://fess.codelibs.org/)

    $ curl -XPUT 'localhost:9200/_river/fess/_meta' -d '{
        "type" : "web",
        "crawl" : {
            "index" : "webindex",
            "url" : ["http://fess.codelibs.org/"],
            "includeFilter" : ["http://fess.codelibs.org/.*"],
            "maxDepth" : 3,
            "maxAccessCount" : 1000,
            "numOfThread" : 5,
            "interval" : 1000,
            "target" : [{
                "pattern" : {
                    "url" : "http://fess.codelibs.org/.*",
                    "mimeType" : "text/html"
                },
                "properties" : {
                    "title" : {
                        "text" : "title"
                    },
                    "body" : {
                        "text" : "body",
                        "trimSpaces" : true
                    }
                }
            }]
        },
        "schedule" : {
            "cron" : "0 0 0 * * ?"
        }
    }'


### Aggregate a title/content from news.yahoo.com

    $ curl -XPUT 'localhost:9200/_river/yahoo_com/_meta' -d '{
        "type" : "web",
        "crawl" : {
            "index" : "webindex",
            "url" : ["http://news.yahoo.com/"],
            "includeFilter" : ["http://news.yahoo.com/.*"],
            "maxDepth" : 1,
            "maxAccessCount" : 100,
            "numOfThread" : 3,
            "interval" : 3000,
            "target" : [
              {
                "urlPattern" : "http://news.yahoo.com/.*html",
                "properties" : {
                  "title" : {
                    "text" : "h1.headline"
                  },
                  "content" : {
                    "text" : "section.mediacontentstory div.body p"
                  }
                }
              }
            ]
        },
        "schedule" : {
            "cron" : "0 0 * * * ?"
        }
    }'


## Others

### BASIC/DIGEST/NTLM authentication

River Web supports BASIC/DIGEST/NTLM authentication.
Set crawl.authentications object.

    ...
    "numOfThread" : 5,
    "interval" : 1000,
    "authentications":[
      {
        "scope": {
          "scheme":"BASIC"
        },
        "credentials": {
          "username":"testuser",
          "password":"secret"
        }
      }],
    "target" : [
    ...

The configuration is:

| Property                                      | Type    | Description                                     |
|:----------------------------------------------|:-------:|:------------------------------------------------|
| crawl.authentications.scope.scheme            | string  | BASIC, DIGEST or NTLM                           |
| crawl.authentications.scope.host              | string  | (Optional)Target hostname.                      |
| crawl.authentications.scope.port              | int     | (Optional)Port number.                          |
| crawl.authentications.scope.realm             | string  | (Optional)Realm name.                           |
| crawl.authentications.credentials.username    | string  | Username.                                       |
| crawl.authentications.credentials.password    | string  | Password.                                       |
| crawl.authentications.credentials.workstation | string  | (Optional)Workstation for NTLM.                 |
| crawl.authentications.credentials.domain      | string  | (Optional)Domain for NTLM.                      |

For example, if you want to use an user in ActiveDirectory, the configuration is below:

    "authentications":[
      {
        "scope": {
          "scheme":"NTLM"
        },
        "credentials": {
          "domain":"your.ad.domain",
          "username":"taro",
          "password":"himitsu"
        }
      }],


### Use Multibyte Characters

An example in Japanese environment is below.
First, put some configuration file into conf directory of Elasticsearch.

    $ cd $ES_HOME/conf    # ex. /etc/elasticsearch if using rpm package
    $ sudo wget https://raw.github.com/codelibs/fess-server/master/src/tomcat/solr/core1/conf/mapping_ja.txt
    $ sudo wget http://svn.apache.org/repos/asf/lucene/dev/trunk/solr/example/solr/collection1/conf/lang/stopwords_ja.txt 

and then create "webindex" index with analyzers for Japanese.
(If you want to use uni-gram, remove cjk\_bigram in filter)

    $ curl -XPUT "localhost:9200/webindex" -d "
    {
      \"settings\" : {
        \"analysis\" : {
          \"analyzer\" : {
            \"default\" : {
              \"type\" : \"custom\",
              \"char_filter\" : [\"mappingJa\"],
              \"tokenizer\" : \"standard\",
              \"filter\" : [\"word_delimiter\", \"lowercase\", \"cjk_width\", \"cjk_bigram\"]
            }
          },
          \"char_filter\" : {
            \"mappingJa\": {
              \"type\" : \"mapping\",
              \"mappings_path\" : \"mapping_ja.txt\"
            }
          },
          \"filter\" : {
            \"stopJa\" : {
              \"type\" : \"stop\",
              \"stopwords_path\" : \"stopwords_ja.txt\"
            }
          }
        }
      }
    }"

