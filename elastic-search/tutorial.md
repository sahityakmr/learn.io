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
                "value": "daldo",
                "fuzziness": 4, 
                "max_expansions": 50,
                "prefix_length": 1,
                "transpositions": true,
                "rewrite": "constant_score"
            }
        }
    }
}

