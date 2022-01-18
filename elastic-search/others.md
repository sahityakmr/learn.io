# Setup Two Nodes Elastic Search Cluster:

## Presetup (Worked without this on Ubuntu 20.10)
```
nano /etc/security/limits.conf
```

`Add following configs (for user sahitya):`

```
sahitya  -  nofile  65535
sahitya  -  nproc  4096
```

```
nano /etc/sysctl.conf
```
`Add following configs:`

```
vm.swappiness= 1
vm.max_map_count=262144
```

`Reboot`
```
sudo reboot
```

## Download ElasticSearch and copy to two directories

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.16.3-linux-x86_64.tar.gz

mkdir /opt/elastic-search/

mkdir /opt/elastic-search/es01/

mkdir /opt/elastic-search/es02/

cp elasticsearch-7.16.3-linux-x86_64.tar.gz /opt/elastic-search/es01/

cp elasticsearch-7.16.3-linux-x86_64.tar.gz /opt/elastic-search/es02/

tar -xvf /opt/elastic-search/es01/elasticsearch-7.16.3-linux-x86_64.tar.gz 

tar -xvf /opt/elastic-search/es02/elasticsearch-7.16.3-linux-x86_64.tar.gz 
```

## Edit configs for es01:

```
cd /opt/elastic-search/es01/elasticsearch-7.16.3-linux-x86_64/config

nano elasticsearch.yml
```

`Add following as the configuration`

```
cluster.name: Sahitya_ES
node.name: ES1
node.attr.rack: RC1
node.attr.type: Highend
network.host: localhost
http.port: 9700
cluster.routing.allocation.awareness.attributes: rack
cluster.initial_master_nodes: ["ES1"]
discovery.seed_hosts: ["localhost"]
node.master: true
node.data: true
node.ingest: true
node.ml: true
node.transform: false
```

## Edit configs for es02:

```
cd /opt/elastic-search/es02/elasticsearch-7.16.3-linux-x86_64/config

nano elasticsearch.yml
```

`Add following as the configuration`

```
cluster.name: Sahitya_ES
node.name: ES2
node.attr.rack: RC2
node.attr.type: Lowend
network.host: localhost
http.port: 9701
discovery.seed_hosts: ["localhost"]
node.master: true
node.data: true
node.ingest: false
node.ml: true
node.transform: true
```

## Bootstrap ES1

```
cd /opt/elastic-search/es01/elasticsearch-7.16.3-linux-x86_64/bin

nohup ./elasticsearch &

tail -f ../logs/Sahitya_ES.log

curl http://localhost:9700

curl http://localhost:9700/_cluster/health?pretty
```

## Bootstrap ES2

```
cd /opt/elastic-search/es02/elasticsearch-7.16.3-linux-x86_64/bin

nohup ./elasticsearch &

tail -f ../logs/Sahitya_ES.log

curl http://localhost:9701
```


# Setup Kibana:

## Download and setup kibana
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.16.3-linux-x86_64.tar.gz

mkdir /opt/kibana/

cp kibana-7.16.3-linux-x86_64.tar.gz /opt/kibana/

tar -xvf /opt/kibana/kibana-7.16.3-linux-x86_64.tar.gz
```

## Edit configs for kibana:

```
cd /opt/kibana/kibana-7.16.3-linux-x86_64/config

nano kibana.yml
```

`Add following as the configuration`

```
server.host: "localhost"
elasticsearch.hosts: ["http://localhost:9700","http://localhost:9701"]
```

## Bootstrap Kibana

```
cd /opt/kibana/kibana-7.16.3-linux-x86_64/bin

nohup ./kibana &

curl http://192.168.29.82:9700/_cat/nodes

curl http://192.168.29.82:9700/_cat/indices
```


# Setup LogStash:

## Download and setup logstash
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.16.3-linux-x86_64.tar.gz

mkdir /opt/logstash/

cp logstash-7.16.3-linux-x86_64.tar.gz /opt/logstash/

tar -xvf /opt/logstash/logstash-7.16.3-linux-x86_64.tar.gz
```

## Make configs for logstash:

```
cd /opt/logstash/

touch my-conf.conf

nano my-conf.conf
```

`Add following as the configuration`

```
input {
    file {
        path => "/home/sahitya/Workspace/data/logstash/apache-logs-file.logs"
        start_position => "beginning"
        sincedb_path => "/home/sahitya/Workspace/data/logstash/db2.log"
    }
}

filter {
    grok {
        match => {
            "message" => '%{COMBINEDAPACHELOG}'
        }
    }
    date {
        match => [ 
            "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" 
        ]
        locale => en
    }
    geoip {
        source => "clientip"
    }
}

output {
    stdout {
        codec => rubydebug
    }

    elasticsearch {    
        hosts => [
            "localhost:9700"
        ]
        index => "log-%{+YYYY.MM.dd}"
        document_type => "_doc"
    }
}
```

## Bootstrap Logstash

```
cd /opt/logstash/bin

./logstash -f ../my-conf.conf
```