# This Readme shows the fastest way to stand up the ELK stack for development.

The Elastic stack has a nice website that provides the links to the docker images and documentation. This example, is using the version v6.4.3 for all the elastic stack.

**1. Run ElasticSearch**

_docker run -d --name my-es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.4.3_

After the container starts running, find out the internal docker IP the ElasticSearch server is using. This will be used for configuring the Logstash and Kibana.

_docker inspect -f '{{ .NetworkSettings.IPAddress }}' my-es_

**2. Run Logstash**

Create a configuration file in the host directory, and bind mount to the container configuration file.

_vim ~/logconf/logstash.conf_

- Enter the following and save with :wq. Note that we created an index pattern that starts with leafyjava-*.

```
input {
    tcp {
        port => 5050
        codec => json
    }
}
output {
    elasticsearch {
        hosts => ["http://172.17.0.7:9200"]
        index => "leafyjava-%{appName}"
    }
}
```

- Now create index pattern for Isilon.

_vim ~/logconf/isilon-syslog.conf_

```
Copy and past from git file isilon-syslog.conf
```


Run the container. We need to use -f flag to ask Logstash to use the config file we just bound.

_docker run -d --name my-logstash -p 5050:5050 -v ~/elk/config/logstash.conf:/config-dir/logstash.conf docker.elastic.co/logstash/logstash:6.4.3 -f /config-dir/logstash.conf_

**3. Run Kibana**

_docker run -d --name my-kibana -e "ELASTICSEARCH_URL=http://172.17.0.7:9200" -p 5601:5601 docker.elastic.co/kibana/kibana:6.4.3_

Go to _localhost:5601_, and you will see the Kibana console up and running. Now you are ready for integrating your applications to use this elastic stack.
