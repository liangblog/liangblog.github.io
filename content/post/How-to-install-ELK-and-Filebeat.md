---
title: How to install ELK and Filebeat
date: 2017-09-12 14:18:52
tags:
- Elasticsearch
- Logstash
- Kibana
- Filebeat
- Ubuntu
---


### ELK Server Environment
```
$ cat /etc/issue
  Ubuntu 14.04.4 LTS
$ ifconfig eth0 | grep inet
  inet addr:192.168.0.110
$ cat /proc/cpuinfo | grep processor
  processor	: 0
  processor	: 1
  processor	: 2
  processor	: 3
$ cat /proc/meminfo | grep MemTotal
  MemTotal:        8125540 kB    
```

### Filebeat Client Server Environment
```
$ cat /etc/issue
  Ubuntu 14.04.5 LTS
$ ifconfig eth0 | grep inet
  inet addr:192.168.0.120
```
<!--more-->

### Dependence Software Version

```
Elasticsearch 2.2.x
Logstash 2.2.x
Kibana 4.4.x
Filebeat:1.3.1
Java 1.8
```

### Server Distribution
![](/images/elk.png)


- Filebeat offers a lightweight way to forward and centralize logs and files without using SSH
- Logstash collects and filters logs from client server then sends the logs to elasticsearch
- Elasticsearch is a distributed, scalable, real-time search engine. Here it 's used to store the logs.
- Kibana enables visual exploration and real-time search of your data in elasticsearh


### Install Java8(openJDK)
```
$ sudo add-apt-repository ppa:openjdk-r/ppa
$ sudo apt-get update 
$ sudo apt-get install -y openjdk-8-jdk
```

Check java version
```
$ javac -version
javac 1.8.0_141
$ java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-3~14.04-b15)
OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)
```
 
### Install Elasticsearch
```
$ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
$ sudo apt-get update
$ sudo apt-get -y install elasticsearch
```

Limit your Elasticsearch api just can be accessed in local machine.
```
$ sudo vim /etc/elasticsearch/elasticsearch.yml
network.host: 127.0.0.1
$ sudo service elasticsearch restart
```

Configure auto start elasticsearch when System started
```
$ sudo update-rc.d elasticsearch defaults 95 10
```

### Install Logstash
```
$ echo 'deb http://packages.elastic.co/logstash/2.2/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash-2.2.x.list
$ sudo apt-get update
$ sudo apt-get install logstash
```
Add a listing port for receiving data
```
$ sudo vim /etc/logstash/conf.d/10-filebeat-input.conf 
```
The config file "10-filebeat-input.conf" maybe is not exist. Create a new one.

```
input {
    beats {
        port => 5044
    }
}

output {
    elasticsearch {
        hosts => ["http://127.0.0.1:9200"]
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
    }

}
```

```
$ /etc/init.d/logstash restart
```
Logstash will build a new elasticsearch index named with date then send the data which was received in port 5044 to elasticsearch

### Install Kibana
```
$ echo "deb http://packages.elastic.co/kibana/4.4/debian stable main" | sudo tee -a /etc/apt/sources.list.d/kibana-4.4.x.list
$ sudo apt-get update
$ sudo apt-get -y install kibana
```

Limit Kibana access ip
```
$ sudo vim /opt/kibana/config/kibana.yml
server.host: 127.0.0.1
$ sudo service kibana start
```

Configure auto start kibana when System started

```
$ sudo update-rc.d kibana defaults 96 9
```

### Install Filebeat In Client Server (Machine IP: 192.168.0.120)
```
$ echo "deb https://packages.elastic.co/beats/apt stable main" |  sudo tee -a /etc/apt/sources.list.d/beats.list
$ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install filebeat
```

### Configure Filebeat
```
$ vim /etc/filebeat/filebeat.yml
```
- Filebeat just read the file named "filebeat.yml" under directory "/etc/filebeat/"

Find the paths that should be crawled and fetched in this config file

```
############################# Filebeat ######################################
filebeat:
  # List of prospectors to fetch data.
  prospectors:
    # Each - is a prospector. Below are the prospector specific configurations
    -
      # Paths that should be crawled and fetched. Glob based paths.
      # To fetch all ".log" files from a specific level of subdirectories
      # /var/log/*/*.log can be used.
      # For each file found under this path, a harvester is started.
      # Make sure not file is defined twice as this can lead to unexpected behaviour.
      paths:
        - /tmp/liyuliang-test.log
```
Look at the file "/tmp/liyuliang-test.log", when it's content was changed, filebeat will send the content out.

- paths：which file will be watched . It is processed according to the glob function in Go language. 
It does not do the recursion.Such as
```
/var/log/*/*.log
```
- It will never to find the file "/var/log/*.log"


Annotate all elaticsearch output part
```
############################# Output ##########################################

# Configure what outputs to use when sending the data collected by the beat.
# Multiple outputs may be used.
output:

  ### Elasticsearch as output
  #elasticsearch:
    # Array of hosts to connect to.
    # Scheme and port can be left out and will be set to the default (http and 9200)
    # In case you specify and additional path, the scheme is required: http://localhost:9200/path
    # IPv6 addresses should always be defined as: https://[2001:db8::1]:9200
    #hosts: ["localhost:9200"]

    # Optional protocol and basic auth credentials.
    #protocol: "https"
    #username: "admin"
    #password: "s3cr3t"
```

Find the logstash output part and config logstash ip:port and elasticsearch index name
 
```
############################# Output ##########################################

# Configure what outputs to use when sending the data collected by the beat.
# Multiple outputs may be used.
output:
  ...
  ...
  ...
  ### Logstash as output
  logstash:
    # The Logstash hosts
    hosts: ["192.168.0.110:5044"]

    # Number of workers per Logstash host.
    worker: 2

    # The maximum number of events to bulk into a single batch window. The
    # default is 2048.
    bulk_max_size: 2048

    # Set gzip compression level.
    #compression_level: 3

    # Optional load balance the events between the Logstash hosts
    loadbalance: true

    # Optional index name. The default index name depends on the each beat.
    # For Packetbeat, the default is set to packetbeat, for Topbeat
    # top topbeat and for Filebeat to filebeat.
    index: "filebeat-logstash"
```

Restart filebeat
```
$ /etc/init.d/filebeat restart
```

### Test Something Input The Log
```
echo “123456789” >> /tmp/liyuliang-test.log
```

Then check the data can be received in ELK sever
```
$ curl 192.168.0.110:9200/_cat/indices
yellow open filebeat-logstash-2017.09.12 5 1 60 0    99kb    99kb 
yellow open .kibana                      1 1  1 0   3.1kb   3.1kb 
```
It's success that elasticsearch index filebeat-logstash-2017.09.12 was built.