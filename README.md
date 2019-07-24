# ELK with springboot and gradle
In microservices envrionment, it is a big challenge to handle logs logged by multiple services. There are multiple open source projects to solve the problem. I providing setup details on how you can setup that in windows 10 machine for learning/testing purpose. For production environment I will provide in next project.

## My environment
* OS : Windows 10
* JAVA: open-jdk-8
## Install Elastic search
1. Download Elasticsearch from this download page and unzip it any folder. I have downloaded [elasticsearch-7.2.0]
2. Run bin\elasticsearch.bat from command prompt.
3. Open you browser and enter http://localhost:9200. You will some ouput like below

```
{
  "name" : "YOUR-PC-NAME",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Oxap0mLETSWDfPecbApDIQ",
  "version" : {
    "number" : "7.2.0",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "508c38a",
    "build_date" : "2019-06-20T15:54:18.811730Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```



