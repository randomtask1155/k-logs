## Setting up the Environemnt


### Collected system information

* Make note of your ipaddres
  * MAC WIFI = ifconfig en0
  * MAC WIRE = ifconfig en1

### Download docker images

```
docker pull docker.elastic.co/kibana/kibana:6.6.2
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.6.2
docker pull docker.elastic.co/logstash/logstash:6.6.2
```

### starting environment


* Make this repository is your current working directory
* Start Kibana
https://www.elastic.co/guide/en/kibana/current/docker.html

**NOTE** update docker-compose.yml and replace the ip address for `ELASTICSEARCH_HOSTS` with the ip address of your mac

```
docker-compose up 
```

* Start Elastic Search
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.6.2
```

* Start LogStash
https://www.elastic.co/guide/en/logstash/current/docker.html

**NOTE** replace the fake ip address in the `--add-host` arg with the ip address of your mac

```
docker run --rm -it -e xpack.monitoring.elasticsearch.url="my.es.local" --add-host my.es.local:192.168.x.x -v `pwd`/pipeline/:/usr/share/logstash/pipeline/ -v `pwd`/logs:/support/logs -v `pwd`/logstash-patterns:/usr/share/logstash/patterns_extra docker.elastic.co/logstash/logstash:6.6.2
```

### Load logs files

Copy logs files into `logs/COMPONENT`.  Take gorouter for example and copy both the access and stdout logs into the gorouter folder

```
find ~/gorouter/ | egrep "access.log.*|gorouter.stdout.*" | while read line; do cp $line logs/gorouter/; done
```

logstash will import all the logs int elastic search with index name like `logstash-2019.03.18`.  You can confirm this by going to  [http://127.0.0.1:9200/_cat/indices?v](http://127.0.0.1:9200/_cat/indices?v).

Once all the data is imported you can then access kibian from [http://127.0.0.1:5601](http://127.0.0.1:5601) and create an index by going to management tab. then create an index pattern with `logstash*`.  Also Filter time by `@timestamp`.  Then go to discover and start searching the logs.




## Troubleshooting

### Attach a bash process to docker container

```
docker exec -it CONTAINERID /bin/bash
```