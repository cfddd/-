```json

# 查看es中的索引
GET /_cat/indices
# 查看es中的索引,同时查看状态
GET /_cat/indices?v

# 创建索引(默认rep=1)
PUT /products

# 创建订单索引(让rep=0)
PUT /orders
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}

# 删除索引
DELETE /products


# 创建商品索引 products 指定 mapping{id,title,price,created_at,description}
PUT /products
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "id":{
        "type": "integer"
      },
      "title":{
        "type": "keyword"
      },
      "price":{
        "type": "double"
      },
      "created_at":{
        "type": "date"
      },
      "description":{
        "type": "text"
      }
    }
  }
}

# 查看某个索引的映射信息 mapping
GET /products/_mapping


# 添加文档操作 手动指定 _id
POST /products/_doc/1
{
  "id" : 1,
  "title":"小浣熊",
  "price":0.5,
  "created_at":"2012-11-12",
  "description":"小老鼠真好吃"
}

# 添加文档操作 自动创建文档的 id CurihYsBq0CcYHMB1rPB
POST /products/_doc/
{
  "id" : 1,
  "title":"大香蕉",
  "price":0.5,
  "created_at":"2012-11-12",
  "description":"小老鼠真好吃"
}

# 文档查询  基于 id 查询
GET /products/_doc/CurihYsBq0CcYHMB1rPB

# 删除文档 基于 id 删除
DELETE /products/_doc/CurihYsBq0CcYHMB1rPB

# 更新文档 先删除原始文档 再重新添加 要带上原始的内容
PUT /products/_doc/1
{
  "description":"小老鼠真香"
}

PUT /products/_doc/1
{
  "id" : 1,
  "title":"小浣熊",
  "price":0.5,
  "created_at":"2012-11-12",
  "description":"小老鼠真香"
}

# 更新文档，基于指定字段进行更新
POST /products/_update/1
{
  "doc":{
    "price":1.0,
    "title":"我是cfddfc"
  }
}

GET /products/_doc/2

# 文档的批量操作 _bulk
# 不能有回车换行
POST _bulk
{"index":{"_id":2,"_index" : "products"} }
{"id" : 2,"title":"大老鼠","price":0.5,"created_at":"2012-11-12","description":"小浣熊真香！"}
{"index":{"_id":3,"_index" : "products"} }
{"id" : 3,"title":"与馒头","price":0.5,"created_at":"2012-11-12","description":"馒头真香啊"}

# 文档批量操作 添加 更新 删除
POST _bulk
{"index":{"_id":4,"_index" : "products"} }
{"id" : 4,"title":"好吃的薯片","price":100,"created_at":"2012-11-12","description":"薯片真香！"}
{"update":{"_id":3,"_index" : "products"} }
{"doc":{"description":"馒头太好吃了吧"}}
{"delete":{"_id":2,"_index" : "products"} }


# query DSL 语法 查询所有 match_all
GET /products/_search
{
  "query":{
    "match_all":{}
  }
}

# term 基于关键词查询 
GET /products/_mapping
# keyword 类型 日后搜索使用 全部内容搜索
# text 类型 默认 es 标准分词器 中文单字分词 英文单词分词
# keyword integer double date 不分词
# 1. 在 ES 中除了 text 类型分词，其余类型均不分词
# 2. 在 ES 中默认使用标准分词器 中文单字分词 英文单词分词
GET /products/_search
{
  "query": {
    "term": {
      "id": {
        "value": 1
      }
    }
  }
}

# 范围查询 ranger
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 1,
        "lte": 20
      }
    }
  }
}

# 前缀查询
GET /products/_search
{
  "query": {
    "prefix": {
      "description": {
        "value": "小"
      }
    }
  }
}
# 因为中文默认分词器是一个字一个字，所以查询不到，结果为空
GET /products/_search
{
  "query": {
    "prefix": {
      "title": {
        "value": "我是"
      }
    }
  }
}

# 通配符查询 * 多个 ? 一个
GET /products/_search
{
  "query": {
    "wildcard": {
      "description": {
        "value": "小*"
      }
    }
  }
}

# ids 多id查询 查询一组符合条件的id
GET /products/_search
{
  "query": {
    "ids": {
      "values": [3,4,2]
    }
  }
}

# fuzzy 模糊查询
GET /products/_search
{
  "query": {
    "fuzzy": {
      "description": "馒头太好吃了吧"
    }
  }
}

# bool 查询
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "ids": {
            "values": [1]
          }
        },{
          "term": {
            "title": {
              "value": "我是cfddfc"
            }
          }
        }
      ]
    }
  }
}

# 多字段查询 multi_match
# 注意：query 输入关键词 输入一段文本
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "老鼠馒头",
      "fields": ["title","description"]
    }
  }
}

# query_string
GET /products/_search
{
  "query": {
    "query_string": {
      "default_field": "description",
      "query": "*"
    }
  },
  "highlight": {
    "pre_tags": ["<span style='color:red;'>"],
    "post_tags": ["</span>"], 
    "require_field_match": "false", 
    "fields": {
      "description":{}
    }
  },
  "sort":[
    {
      "price":"desc"
    }
  ],
  "_source": ["id","title","description"]
}

# 删除所有，重新写
DELETE /products
PUT /products
{
  "mappings" : {
		"properties" : {
			"description" : {
				"type" : "text"
			},
			"price" : {
				"type": "float"
			},
			"title": {
				"type" : "keyword"
			}
		}
	}
}
# 测试索引
POST _bulk
{"index":{"_id":1,"_index" : "products"} }
{"id" : 1,"title":"洗衣液","price":25,"description":"这个洗衣液很搞笑"}
{"index":{"_id":2,"_index" : "products"} }
{"id" : 2,"title":"手机","price":1999,"description":"很好用"}
{"index":{"_id":3,"_index" : "products"} }
{"id" : 3,"title":"小老鼠","price":0,"description":"很好吃"}
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
GET /products/_search
{
  "query":{
    "term":{
      "description":{
        "value":"很"
      }
    }
  }
}

# 分词器测试
POST /_analyze
{
  "analyzer":"ik_smart",
  "text":"常凤迪"
}

PUT /test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "text",
        "analyzer": "keyword"
      }
    }
  }
}
PUT /test/_doc/1
{
  "title":"我是我是我是我是cfd a interasting person"
}
DELETE /test
GET /test/_search
{
  "query": {
    "match_all": {}
  }
}
GET /test/_search
{
  "query": {
    "term": {
      "title": {
        "value": "我是我是我是我是cfd a interasting person"
      }
    }
  }
}

# filter过滤
GET /products/_search
{
  "query":{
    "match_all":{}
  }
}
PUT /products/_doc/4
{
  "id":4,
  "title":"飞机",
  "price":10000000,
  "description":"很快"
}
GET /products/_search
{
  "query":{
    "bool": {
      "filter": [
        {"term": {"description": "很"}},
        {"term": {"description": "好"}},
        {"terms": {
          "description": [
            "很",
            "好"
          ]
        }}
      ]
    }
  }
}
GET /products/_search
{
  "query":{
    "bool": {
      "filter": [
        {"range": {
          "price": {
            "gte": 1,
            "lte": 100
          }
        }}
      ]
    }
  }
}
GET /products/_search
{
  "query":{
    "bool": {
      "filter": [
        {"exists": {
          "field": "id"
        }}
      ]
    }
  }
}
GET /products/_search
{
  "query":{
    "bool": {
      "filter": [
        {"ids": {
          "values": [
            "1"
          ]
        }}
      ]
    }
  }
}

# 创建索引 index 和映射 mapping 
PUT /friends
{
  "mappings": {
    "properties": {
      "name":{
        "type":"keyword"
      },
      "age":{
        "type": "integer"
      },
      "gender":{
        "type": "boolean"
      },
      "description":{
        "type": "text",
        "analyzer": "ik_smart"
      }
    }
  }
}
# 放入测试用例
PUT _bulk
{"index":{"_id":1,"_index" : "friends"} }
{"name" : "cfd","age":200,"gender":true,"description":"这个人很搞笑"}
{"index":{"_id":2,"_index" : "friends"} }
{"name" : "cdf","age":100,"gender":true,"description":"这个人很聪明"}
{"index":{"_id":3,"_index" : "friends"} }
{"name" : "fcd","age":50,"gender":true,"description":"这个人很乐观"}
{"index":{"_id":4,"_index" : "friends"} }
{"name" : "fdc","age":25,"gender":true,"description":"这个人很帅气"}
{"index":{"_id":5,"_index" : "friends"} }
{"name" : "dcf","age":10,"gender":true,"description":"这个人很积极"}
{"index":{"_id":6,"_index" : "friends"} }
{"name" : "dfc","age":20,"gender":true,"description":"这个人很喜欢"}

GET /friends/_search
{
  "query": {
    "match_all": {}
  }
}

# 根据某个字段分组 统计数量 
GET /friends/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "price_group": {
      "terms": {
        "field": "age"
      }
    }
  }
}
GET /friends/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0, 
  "aggs": {
    "max_price": {
      "max": {
        "field": "age"
      }
    }
  }
}
GET /friends/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "min_price": {
      "min": {
        "field": "age"
      }
    }
  }
}
GET /friends/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "sum_price": {
      "sum": {
        "field": "age"
      }
    }
  }
}






```