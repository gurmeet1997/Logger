# Opendistro for elasticsearch
Logs are a critical part of any system, they give you deep insights about your application, what your system is doing and what caused the error, when something wrong happens.Several times we came accross a situation where troubleshooting the application is required ,Debugging the error in the application across hundreds of log files on hundreds of servers can be very time consuming and complicated. A common approach to this problem is building a centralized logging application which can collect and aggregate different types of logs in one central location.There are many tools for this management many are the paid ones but we tried an approach that will be the boubust one also it should be the openSource.There are primarily four featues(funtionality) that we integrated in the present project.For that We have integrated Opendistro for elastic search with Logstash.
- Storing and collecting logs
 - Viuslizing them
 - Alerting them to the Slack Channel
 - Performance analysis of the cluster.
**We will going to look into these four parts and eventually see how we can build them in real world application environment.**
## Prerequisite 
  - JAVA- JDK 11
  - Docker
  - Ubuntu 20.04
  
Above requirements are the minimal ones

## Stroing and Collecting Logs:
   If you have ever looked in the /var/log in the linux environment then it stores the logs for various services running in linux.Different applications logs to different locations and so we want to Centralize all these logs to a central location.For this we used  An open Source Log collector. For example if you ever need to Troubleshoot an appliction then your developer must need to see it's logs and visulaize them. We will provide the whole details how we could make this happen in almost real time.For this we have used docker image for logstash [image](https://github.com/docker-library/logstash/blob/f9a68426beb578052b01cccd0ecfd87614cb9b9e/7/Dockerfile). We have written a logstash.conf.
   ## Visualizing logs:
  The logs collected from the different log inputs are indexed into elasticsearch Cluster and as said earlier elasticsearch accpets document as JSON format only.Opendistro provides you with funtionality to index a doc into the elasticsearch cluster using REST API calls.
 
We are Using OpenDistro for elastic search for Processing logs of different microservices to make them available at a central place where all the logs can be visulaized with kibana.Here two microsevices order-service and payment-service has been created in Spring boot. The logs for these two application can be collected at any destination of user choice. For simplicity i have collected them to a local storage.We have integrated the java services with Eureka Cloud for visulaization and also these services are communicating with each other with the API gatway. 
As a part of the project we need to test out model for Ruby services also. So i have created  a sample Ruby Service which is running with the java services and logs of this ruby service is also being analyzed.

The Main benefits of using opendistro is it's on premise availabiltiy of Security,Alerting and Performance Analysis.The best way to configure opendistro in the production Environement is using docker.It is also quite well documented here [documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/install/docker/#start-a-cluster).


 So first you need to pull following images into your system :
 ```
 docker pull amazon/opendistro-for-elasticsearch:1.8.0
 docker pull amazon/opendistro-for-elasticsearch-kibana:1.8.0
 docker pull amazon/opendistro-for-elasticsearch-kibana:1.8.0
 docker.elastic.co/logstash/logstash:7.8.0
 
 ```
 
 Now on the basis of above images you need to configure your docker-compose.yml file which is as below :
```

version: '3'
services:
  odfe-node1:
    image: amazon/opendistro-for-elasticsearch:1.8.0
    container_name: odfe-node1
    environment:
      - cluster.name=odfe-cluster
      - node.name=odfe-node1
      - discovery.seed_hosts=odfe-node1,odfe-node2
      - cluster.initial_master_nodes=odfe-node1,odfe-node2
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536 # maximum number of open files for the Elasticsearch user, set to at least 65536 on modern systems
        hard: 65536
    volumes:
      - odfe-data1:/usr/share/elasticsearch/data
      #- ./custom-elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9200:9200
      - 9600:9600 # required for Performance Analyzer
    networks:
      - odfe-net
  odfe-node2:
    image: amazon/opendistro-for-elasticsearch:1.8.0
    container_name: odfe-node2
    environment:
      - cluster.name=odfe-cluster
      - node.name=odfe-node2
      - discovery.seed_hosts=odfe-node1,odfe-node2
      - cluster.initial_master_nodes=odfe-node1,odfe-node2
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - odfe-data2:/usr/share/elasticsearch/data
      #- ./custom-elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    networks:
      - odfe-net
  kibana:
    image: amazon/opendistro-for-elasticsearch-kibana:1.8.0
    container_name: odfe-kibana
    ports:
      - 5601:5601
    expose:
      - "5601"
    environment:
      ELASTICSEARCH_URL: https://odfe-node1:9200
      ELASTICSEARCH_HOSTS: https://odfe-node1:9200
    networks:
      - odfe-net
  logstash:
    image: docker.elastic.co/logstash/logstash:7.8.0
    container_name: logstash
    volumes:
     -./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    -./myfile.log:/usr/share/logstash/pipeline/myfile.log
    
     ports:
      - "5011:5011"
      - "5012:5012"
      - "5013:5013"
    networks:
      - odfe-net
    depends_on:
      - odfe-node1
      
volumes:
    odfe-data1:
    odfe-data2:
  
networks:
    odfe-net:
    
```

**Make sure that both the docker-compose.yml and logstash.conf are in the same directory**
Next to checking that your elasticsearch cluster is running fine you can curl(check) it with following commands.

```

curl -XGET https://localhost:9200 -u admin:admin --insecure
curl -XGET https://localhost:9200/_cat/nodes?v -u admin:admin --insecure
curl -XGET https://localhost:9200/_cat/plugins?v -u admin:admin --insecure

```

Next you can start the container using below command :
```
sudo docker-compose up 
```

for restarting the services you first need to clean the container and then start it again.After confirming through the elasticsearch curl command you will start seeing the logs into stdout thrwon by the logstash and now it is time to start the kibana which can be done by going to the address *http://0:5601/*. There you will see the kiabana UI for Opendistro and deafult username and password for the same is username:admin and password: admin.
