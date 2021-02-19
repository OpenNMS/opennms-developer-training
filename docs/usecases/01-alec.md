# ALEC

## Goal

Understand the [ALEC (Architecture for Learning Enabled Correlation)](https://github.com/OpenNMS/alec/) framework.

## Links

* Documentation: https://alec.opennms.com/alec/1.1.0/index.html
* Presentation from All Things Open 2019: [Smart ALEC](assets/smart-alec-all-things-open-2019.pdf)
* Groovy script used to generated inventory graph: https://github.com/OpenNMS/alec/blob/v1.1.1/datasource/opennms-direct/src/main/resources/inventory.groovy

## Demo

Start Elasticsearch:
```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "search.max_open_scroll_context=50000" docker.elastic.co/elasticsearch/elasticsearch:7.11.0
```

Setup oce-tools:
```
wget https://github.com/OpenNMS/oce-tools/releases/download/v1.0.0/oce-tools.jar
mkdir -p ~/labs/alec/
mv oce-tools.jar ~/labs/alec/
alias oce-tools="java -jar ~/labs/alec/oce-tools.jar"
mkdir -p ~/.oce
echo 'clusters:
  localhost:
    url: http://localhost:9200
    read-timeout: 120000
    conn-timeout: 30000
    opennms-event-index: opennms-events-raw-*
    opennms-alarm-index: opennms-alarms-*' > ~/.oce/es-config.yaml
```

Load the dataset:
```
mkdir /tmp/alec-demo
cp ~/labs/alec/Aug_19/Aug_01* /tmp/alec-demo
cp ~/labs/alec/Aug_19/Aug_02* /tmp/alec-demo
oce-tools cpn-csv-import --source="/tmp/alec-demo/" --timezone="America/Chicago"
```

Show the indices:
```
curl -X GET "localhost:9200/_cat/indices/?v=true&s=index&pretty"
```

Show a ticket:
```
curl -X GET "localhost:9200/tickets/_search?pretty=true&size=1"
```

Export the data from ES:
```
mkdir /tmp/demo
oce-tools cpn-oce-export --from "Aug 1 2019" --to "Aug 2 2019" --output /tmp/
```

Show the files that were generated as a result of the export:
```
ls /tmp/cpn.*
```


Download and start Karaf:
```
wget http://archive.apache.org/dist/karaf/4.2.10/apache-karaf-4.2.10.tar.gz
tar zxvf apache-karaf-4.2.10.tar.gz
cd apache-karaf-4.2.10
./bin/karaf
```

Install ALEC in the Karaf shell:
```
feature:repo-add mvn:org.opennms.alec/alec-karaf-features/1.1.1/xml
feature:install alec-features-shell alec-engine-dbscan
```

Run a simulation:
```
opennms-alec:process-alarms --alarms-in /tmp/cpn.alarms.xml --inventory-in /tmp/cpn.inventory.xml --situations-out /tmp/alec.situations.xml --engine dbscan
```

Generate a score for the similuation against the baseline:
```
opennms-alec:score-situations -s peer /tmp/cpn.situations.xml /tmp/alec.situations.xml
```

Visualize the results:
```
cp /tmp/cpn.alarms.xml /tmp/alec.alarms.xml
cp /tmp/cpn.inventory.xml /tmp/alec.inventory.xml
docker run -p 8082:8080 -v /tmp/:/dataset opennms/alec-viz
```

Access via: http://localhost:8082/static/index.html
