---
title: "Watcherまとめ"
date: 2019-05-28T13:37:09+09:00
slog: elasticsearch-watcher-basics
tags:
- aws
- elasticsearch
- kibana
- watcher
draft: true
archives:
- 2019/05
---

[Watcher](https://www.elastic.co/guide/jp/x-pack/current/watcher-getting-started.html) についてまとめる。

<!--more-->

Kibana メニューの「 Management 」 -> Elasticsearch「Watcher」 。

# Watcher の設定

以下の項目を設定する。（[Watch Definition](https://www.elastic.co/guide/en/elastic-stack-overview/7.1/how-watcher-works.html#watch-definition)）

- Trigger
    - Watch の実行タイミングを規定
- Input
    - 監視対象とするデータの取得方法を規定
- Condition
    - Input で取得したデータから、 Action を実施するかどうかの識別・制御ルールを規定
    - 基本的には監視対象のインデックスへ検索クエリを実行して情報を取得する
- Transform
    - Input で取得したデータをActionsで利用するための処理を規定
    - Input で取得したデータの整形等が必要ない場合は設定する必要はない（ Optional ）
- Actions
    - Condition で規定した識別・制御ルールを満たした場合の実行内容を規定

## Trigger

```javascript
"trigger" : { 
    "schedule" : {
        "interval" : "5m"
    }
},
```

## Input

```javascript
"input" : { 
    "search" : {
        "request" : {
            "indices" : "log-events",
            "body" : {
                "size" : 0,
                "query" : { "match" : { "status" : "error" } }
            }
        }
    }
},
```

## Condition

```javascript
"condition" : { 
    "compare" : { "ctx.payload.hits.total.value" : { "gt" : 5 }}
},
```

## Transform

```javascript
"transform" : { 
    "search" : {
        "request" : {
            "indices" : "log-events",
            "body" : {
                "query" : { "match" : { "status" : "error" } }
            }
        }
    }
},
```

## Actions

```javascript
"actions" : { 
    "my_webhook" : {
        "webhook" : {
            "method" : "POST",
            "host" : "mylisteninghost",
            "port" : 9200,
            "path" : "/{{watch_id}}",
            "body" : "Encountered {{ctx.payload.hits.total.value}} errors"
        }
    },
    "email_administrator" : {
        "email" : {
            "to" : "sys.admino@host.domain",
            "subject" : "Encountered {{ctx.payload.hits.total.value}} errors",
            "body" : "Too many error in the system, see attached data",
            "attachments" : {
                "attached_data" : {
                    "data" : {
                        "format" : "json"
                    }
                }
            },
            "priority" : "high"
        }
    }
}
```

# 参考

- https://qiita.com/zon_zone/items/93cda732d6c1ce74b8a3
- https://www.shin0higuchi.com/entry/2017/11/02/002955
- https://github.com/elastic/examples/tree/master/Alerting/Sample%20Watches