PUT demo1

GET _cat/shards/demo1

PUT demo3
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}


GET /_cluster/state?filter_path=metadata.indices.demo3.in_sync_allocations.*,routing_table.indices.demo3

GET _cat/indices/demo3

GET _cat/shards/demo3

POST /mukesh_index/_doc
{
  "name": "Mobile",
  "vendors" : ["ABC","XYZ"],
  "description": "this is low end phone available in India in Hyd and Pune in 20 $",
  "price" : 10000,
  "details" : {
  "RAM": "4GB",
  "CPU" : "dualcore"
  },
  "category" : 1
}

POST /sahitya_index/_doc
{
  "name": "Mobile",
  "vendors" : ["ABC","XYZ"],
  "description": "this is low end phone available in India in Hyd and Pune in 20 $",
  "price" : 10000,
  "details" : {
    "RAM": "4GB",
    "CPU" : "dualcore"
  },
  "category" : 1
}


PUT demo3/_doc/3
{
  "name": "Mobile",
  "vendors" : ["ABC","XYZ"],
  "description": "this is high end phone available in India in Bangalore and NCR",
  "price" : 30000,
  "details" : {
    "RAM": "8GB",
    "CPU" : "octacore"
  },
  "category" : 2
}


POST demo3/_doc/3/_update
{
  "doc":{
    "country":"india"
  }
}


GET demo3/_search
{
}


GET demo3/_search
{ "query": {
  "match": {
    "description": "this is india"
  }
}
}

POST demo3/_doc/
{
  "name": "masaba",
  "vendors" : ["ABC","XYZ"],
  "description": "this is high end phone available in India in america",
  "price" : 30000,
  "details" : {
    "RAM": "8GB",
    "CPU" : "octacore"
  },
  "category" : 2
}


GET demo3/_search
{ "query": {
    "match": {
      "description": "america"
    }
  }
}


PUT demo4
{
  "mappings": {
    "dynamic": "true",
    "properties": {
      "name" : {
        "type": "text", 
        "fields": {
          "keyword": {
          }
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}


PUT demo4/_doc/1
{
  "name":"suniti",
  "age":22,
  "city":"delhi"
}


GET demo4/_search
{ "query": {
    "match": {
      "city": "it is good in delhi"
    }
  }
}


PUT demo6
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }, 
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "exact": {
            "type": "keyword"
          }
        }
      },
      "age": {
        "type": "integer"
      }
    }
}
}


PUT demo6/_doc/1
{
  "name":"suniti",
  "age":22,
  "City":"delhi" # will give error since field not defined
}


PUT demo7
{
   "settings":{
      "number_of_shards":2,
      "number_of_replicas":1
   },
   "mappings":{
      "dynamic":"false",
      "properties":{
         "name":{
            "type":"text",
            "fields":{
               "exact":{
                  "type":"keyword"
               }
            }
         },
         "age":{
            "type":"integer"
         }
      }
   }
}


PUT demo7/_doc/1
{
  "name":"suniti",
  "age":22,
  "city":"delhi"
}


GET demo7/_search
{
  "query":{
    "match":{
      "city":"delhi"
    }
  }
}


GET demo7/_search
{
  "query":{
    "match":{
     "name":"suniti"
    }
  }
}


PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_english": { 
          "type":      "standard",
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    
      "properties": {
        "my_text": {
          "type":     "text",
          "analyzer": "standard", 
          "fields": {
            "english": {
              "type":     "text",
              "analyzer": "std_english" 
            }
          }
        }
      }
    
  }
}


POST my_index/_analyze
{
  "field": "my_text",
  "text": "the old brown delhi cow in the jumped for 200$"
}


PUT my_index2
{
   "mappings":{
      "properties":{
         "my_text":{
            "type":"text",
            "analyzer":"my_analyzer",
            "search_analyzer":"my_analyzer"
         }
      }
   },
   "settings":{
      "analysis":{
         "char_filter":{
            "$_to_doller":{
               "type":"mapping",
               "mappings":[
                  "$ => doller"
               ]
            }
         },
         "filter":{
            "my_stopwords":{
               "type":"stop",
               "stopwords":[
                  "the",
                  "a"
               ]
            }
         },
         "analyzer":{
            "my_analyzer":{
               "type":"custom",
               "char_filter":[
                  "html_strip",
                  "$_to_doller"
               ],
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "my_stopwords"
               ]
            }
         }
      }
   }
}


PUT my_index2/_doc/1
{
  "my_text" : "<h1>The old brown fox in new Delhi jumped for 200$"
}


GET my_index2/_search
{
   "query":{
      "match":{
         "my_text":"200doller"
      }
   }
}


GET my_index2/_search
{
   "query":{
      "match":{
         "my_text":"200$"
      }
   }
}


GET my_index2/_search
{
  "query": {
    "match": {
      "my_text": "<h1>the 200doller is my sal"
    }
  }
}


PUT /my_index5
{
   "settings":{
      "analysis":{
         "char_filter":{
            "$_to_doller":{
               "type":"mapping",
               "mappings":[
                  "$ => doller"
               ]
            }
         },
         "filter":{
            "my_stopwords":{
               "type":"stop",
               "stopwords":[
                  "the",
                  "a"
               ]
            }
         },
         "analyzer":{
            "my_analyzer":{
               "type":"custom",
               "char_filter":[
                  "html_strip",
                  "$_to_doller"
               ],
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "my_stopwords",
                  "snowball"
               ]
            }
         }
      }
   }
}


POST my_index5/_analyze
{
  "field": "my_text", 
  "text": "The old brown cow in new Delhi jumped for 200$",
  "analyzer": "my_analyzer"
}


PUT /ngramtest
{
   "settings":{
      "analysis":{
         "analyzer":{
            "my_analyzer":{
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "my_edgengram_filter"
               ]
            }
         },
         "filter":{
            "my_edgengram_filter":{
               "type":"edge_ngram",
               "min_gram":2,
               "max_gram":6
            }
         }
      }
   }
}


GET ngramtest/_analyze 
{
  "analyzer": "my_analyzer", 
  "text":     "The quick fox in New Delhi jumped"
}

PUT ngramtest/_doc/1
{
  "text":     "The quick fox in New Delhi jumped"
}


GET /ngramtest/_search
{
    "query": {
        "fuzzy": {
            "text": {
                "value": "daldi",
                "fuzziness": 4, 
                "max_expansions": 50,
                "prefix_length": 1,
                "transpositions": true,
                "rewrite": "constant_score"
            }
        }
    }
}


GET demo3/_search
{
  "query": {
    "term": {
      "description": {
        "value": "America"
      }
    }
  }
}


GET demo3/_search
{
  "query": {
    "term": {
      "description": {
        "value": "america"
      }
    }
  }
}


GET demo3/_search
{
  "query": {
    "terms": {
      "description": [
        "america",
        "phone"
      ]
    }
  }
}


GET demo3/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "price": {
              "gte": 10000,
              "lte": 30000
            }
          }
        },
        {
          "term": {
            "name": {
              "value": "mobile"
            }
          }
        },
        {
          "match": {
            "description": "bangalore is an IT City"
          }
        }
      ]
    }
  }
}


GET demo3/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "price": {
              "gte": 10000,
              "lte": 30000
            }
          }
        },
        {
          "term": {
            "name": {
              "value": "mobile"
            }
          }
        },
        {
          "bool": {
            "should": [
              {
                "term": {
                  "details.RAM": {
                    "value": "8gb"
                  }
                }
              },
              {
                "term": {
                  "details.CPU": {
                    "value": "octacore"
                  }
                }
              }
            ]
          }
        }
      ]
    }
  }
}


GET demo3/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "description": "china major economy"
          }
        }
      ]
    }
  }
}


PUT _template/template_2
{
    "index_patterns": ["application*","index*"], 
    "order" : 1,
    "settings" : {
        "number_of_shards" : 2
    }   
}

PUT /_template/template_1
{
    "index_patterns": ["applicationfk*","indexfk*"], 
    "order" : 0,
    "settings" : {
        "number_of_shards" : 3,
		"number_of_replicas" : 2
    },
    "mappings" : {
       
            "_source" : { "enabled" : false }
        }
    
}

PUT applicationfk1
GET applicationfk1/_settings
GET applicationfk1/_mapping


PUT demo6
{
  "settings": {"number_of_shards": 2}
}

POST _reindex
{
  "source": {"index": "demo1"
    
  },
  "dest": {"index": "demo8"}
}

GET demo8/_search

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "demo1",
        "alias": "demoalias"
      }
    }
  ]
}


POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "demo8",
        "alias": "demoalias"
      }
    },
    {
      "remove": {"index": "demo1",
         "alias": "demoalias"
      }
    }
  ]
}

GET demoalias/_search

POST _aliases
{
  "actions": [
    {
      "add": {
         "index": "demo8",
        "alias": "demoalias",
        "is_write_index": true
      }
    }
  ]
}

PUT demoalias/_doc/5
{
  "description": "this is new phone"
}

GET _cat/nodeattrs

GET demo1/_segments

PUT demo1/_doc/4
{
  "name":"demo"
}

GET demo1/_segments

POST demo1/_flush

GET demo1/_segments

POST demo1/_forcemerge
 
GET demo1/_segments

PUT demo7/_doc/1
{
  "name": "Mukesh Kumar"
}


PUT demo7/_doc/2
{
  "name": "Sanjay Kumar"
}


GET demo7/_search
{
  "query": {"match_all": {}},
 "sort": [
   {
     "name.keyword": {
       "order": "desc"
     }
   }
 ]
}

GET demo7/_search
{
  "aggs": {
    "NAME": {
      "terms": {
        "field": "name.keyword",
        "size": 10
      }
    }
  },
  "size": 0
}

GET demo1/_settings

PUT demo10
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}

PUT demo10/_doc/1
{
  "name": "Mukesh Kumar"
}


PUT demo10/_doc/2
{
  "name": "Sanjay Kumar"
}

GET demo10/_search
{
  "aggs": {
    "NAME": {
      "terms": {
        "field": "name",
        "size": 10
      }
    }
  },
  "size": 0
}

GET demo10/_search
{
  "query": {"match_all": {}},
  "sort": [
    {
      "name": {
        "order": "desc"
      }
    }
  ]
}


GET /_nodes/stats

POST demo10/_cache/clear

POST demo10/_cache/clear?fielddata
POST demo10/_cache/clear?query
POST demo10/_cache/clear?request

POST _cache/clear

PUT /test-000001 
{
  "aliases": {
    "test_write": {}
  }
}

PUT test-000001/_doc/1
{
  "data": "msg1"
}

PUT test-000001/_doc/3
{
  "data": "msg3"
}

PUT test-000001/_doc/2
{
  "data": "msg2"
}

PUT test-000001/_doc/4
{
  "data": "msg4"
}

POST test_write/_refresh

POST /test_write/_rollover 
{
  "conditions": {
    "max_age":   "7d",
    "max_docs":  2,
    "max_size":  "5gb"
   
  }
}

PUT _ilm/policy/my-lifecycle-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "60d",
        "actions": {
          "searchable_snapshot": {
            "snapshot_repository": "found-snapshots"
          }
        }
      },
      "frozen": {
        "min_age": "90d",
        "actions": {
          "searchable_snapshot": {
            "snapshot_repository": "found-snapshots"
          }
        }
      },
      "delete": {
        "min_age": "735d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}


PUT _component_template/my-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "wildcard"
        }
      }
    }
  },
  "_meta": {
    "description": "Mappings for @timestamp and message fields",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}

# Creates a component template for index settings
PUT _component_template/my-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "my-lifecycle-policy"
    }
  },
  "_meta": {
    "description": "Settings for ILM",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}


# Create a template
PUT _index_template/my-index-template
{
  "index_patterns": ["my-data-stream*"],
  "data_stream": { },
  "composed_of": [ "my-mappings", "my-settings" ],
  "priority": 500,
  "_meta": {
    "description": "Template for my time series data",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}

PUT /cars
{

"mappings" : {

"properties" : {

"make" : {

"type" : "text",

"analyzer" : "standard",
"search_analyzer": "standard",
"fielddata": true

},
"color" : {

"type" : "text",

"analyzer" : "standard",
"fielddata": true

}

}

}

}

POST /cars/_doc/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }

GET cars/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "price": {
              "gte": 10000,
              "lte": 20000
            }
          }
        },
        {
          "term": {
            "make": {
              "value": "honda"
            }
          }
        }
      ]
    }
  }
}

GET log-2014.08.27/_search


PUT _xpack/watcher/watch/apache_200_watch
{
  "trigger" : {
    "schedule" : { "interval" : "60s" } 
  },
  "input" : {
    "search" : {
      "request" : {
        "indices" : [ "log-*" ],
        "body" : {
          "query" : {
            "match" : { "response.keyword": "200" }
          }
        }
      }
    }
  },
  
  "condition" : {
    "compare" : { "ctx.payload.hits.total" : { "gt" : 100 }} 
  },
   "actions" : {
    "log_error" : {
      "logging" : {
        "text" : "Found {{ctx.payload.hits.total}} errors in the apache logs with value 200"
      }
    }
  }
}

PUT my_index35
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": ["answer"]  
        }
      }
    }
  }
}

PUT my_index35/_doc/10?refresh
{
  "description": "This is a question asked by Mukesh",
  "my_join_field": {
    "name": "question" 
  }
}

PUT my_index35/_doc/20?refresh
{
  "text": "This is a another question by the trainer",
  "my_join_field": {
    "name": "question"
  }
}

PUT my_index35/_doc/30?routing=10?refresh 
{
  "text": "This is an answer provided",
  "my_join_field": {
    "name": "answer", 
    "parent": "10" 
  }
}

PUT my_index35/_doc/40?routing=10&refresh
{
  "text": "This is another answer given by the participants",
  "my_join_field": {
    "name": "answer",
    "parent": "10"
  }
}

GET my_index35/_search
{
  "query": {
    "has_parent": {
      "parent_type": "question",
      "query": {"match": {
        "description": "mukesh"
      }}
    }
  }
}

GET my_index35/_search
{
  "query": {
    "has_child": {
      "type": "answer",
      "query": {"match": {
        "text": "participants"
      }}
    }
  }
}


PUT my_index32/_doc/1
{
  "name": "ABC",
  "user" : [{"fistname": "Mukesh","lastname": "Shukla"},{"fistname": "Mahesh","lastname": "Ambani"}]
}

user.firstname: [Mukesh,Mahesh]
user.lastname: [Shukla,Ambani]


GET my_index32/_search
{
  "query": {
    
        "bool": {
          "must": [
            { "match": { "user.fistname": "Mukesh" }},
            { "match": { "user.lastname":  "Ambani" }} 
          ]
        }
      }
    
  
}


PUT my_index30
{
  "mappings": {
      "properties": {
        "user": {
          "type": "nested" 
        }
      }
    }
  
}


PUT my_index30/_doc/1
{
  "name": "ABC",
  "user" : [{"fistname": "Mukesh","lastname": "Shukla"},{"fistname": "Mahesh","lastname": "Ambani"}]
}



GET my_index30/_search
{
  "query": {
    
        "bool": {
          "must": [
            { "match": { "user.fistname": "Mukesh" }},
            { "match": { "user.lastname":  "Ambani" }} 
          ]
        }
      }
    
  
}


GET my_index30/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.fistname": "Mukesh" }},
            { "match": { "user.lastname":  "Ambani" }} 
          ]
        }
      }
     
    }
    
  }
}

GET my_index30/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.fistname": "Mukesh" }},
            { "match": { "user.lastname":  "Shukla" }} 
          ]
        }
      }
     
    }
    
  }
}

GET my_index30/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.fistname": "Mukesh" }},
            { "match": { "user.lastname":  "Shukla" }} 
          ]
        }
      },
      "inner_hits": {}
     
    }
    
  }
}

