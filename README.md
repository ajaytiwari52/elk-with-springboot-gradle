# ELK with springboot and gradle
In microservices envrionment, it is a big challenge to handle logs logged by multiple services. There are multiple open source projects to solve the problem. I providing setup details on how you can setup that in windows 10 machine for learning/testing purpose. For production environment I will provide in next project.

## My environment
* OS : Windows 10
* JAVA: open-jdk-8
## Install Elastic search
1. Download Elasticsearch from this download page and unzip it any folder. I have downloaded `elasticsearch-7.2.0`.
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
## Install Kibana
1. Download kibana from [here](https://www.elastic.co/downloads/kibana). I downloaded `kibana-7.2.0-windows-x86_64`.
2. Unzip it and open kibana.yml file inside config foler. Uncomment and set elasticsearch.hosts as  `elasticsearch.hosts: ["http://localhost:9200"]`.
3. Start the kibana from bin\kibana.bat.
4. Open `http://localhost:5601` in browser.

## Install Logstash
1. Download logstash from [here](https://www.elastic.co/downloads/logstash). I downloaded `logstash-7.2.0`.

## Create springboot application
1. Go to [here](https://start.spring.io/).
2. Select project as `Gradle` project.
3. Select Language as `Java`. You can choose any language. In my case, I chose `Java`.
4. I select springboot version `2.1.6`.
5. Click generate the project.

It will download the project in your local disk. Unzip it and add rest controller.

```
import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.Date;
 
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.HttpMethod;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
 
@SpringBootApplication
public class SpringBootApplication {
     public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication.class, args);
    }
}
 
@RestController
class TestController {
    private static final Logger LOG = Logger.getLogger(TestController.class.getName());
 
    @Autowired
    RestTemplate restTemplete;
 
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
 
    @RequestMapping(value = "/elktest")
    public String helloWorld() {
        String response = "Hello user ! " + new Date();
        LOG.log(Level.INFO, "/elktest - &gt; " + response);
 
        return response;
    }
 
    @RequestMapping(value = "/elk")
    public String helloWorld1() {
 
        String response = restTemplete.exchange("http://localhost:8080/elktest", HttpMethod.GET, null, new ParameterizedTypeReference() {
        }).getBody();
        LOG.log(Level.INFO, "/elk - &gt; " + response);
 
        try {
            String exceptionrsp = restTemplete.exchange("http://localhost:8080/exception", HttpMethod.GET, null, new ParameterizedTypeReference() {
            }).getBody();
            LOG.log(Level.INFO, "/elk trying to print exception - &gt; " + exceptionrsp);
            response = response + " === " + exceptionrsp;
        } catch (Exception e) {
            // catch
        }
 
        return response;
    }
 
    @RequestMapping(value = "/exception")
    public String exception() {
        String rsp = "";
        try {
            int i = 1 / 0;
            // should get exception
        } catch (Exception e) {
            e.printStackTrace();
            LOG.error(e);
             
            StringWriter sw = new StringWriter();
            PrintWriter pw = new PrintWriter(sw);
            e.printStackTrace(pw);
            String sStackTrace = sw.toString(); // stack trace as a string
            LOG.error("Exception As String :: - &gt; "+sStackTrace);
             
            rsp = sStackTrace;
        }
 
        return rsp;
    }
}

```
## Add log file name
In springboot application.properties file add below lines
```
logging.file=C:/elk/spring-boot-elk.log
```

## Configure logstash
Go to installation directory of logstash `..\logstash-7.2.0\bin` and create file logstash.conf and put below contents
```
input {
  file {
    type => "java"
    path => "C:/elk/spring-boot-elk.log"
    codec => multiline {
      pattern => "^%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}.*"
      negate => "true"
      what => "previous"
    }
  }
}

filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }

}

output {

  stdout {
    codec => rubydebug
  }

  # Sending properly parsed log events to elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}
```
Now open the command prompt from `/logastash/bin` and run `logstash -f logstash.conf to start logstash

Open Kibana and add pattern logstash-*.

##### References
https://howtodoinjava.com/microservices/elk-stack-tutorial-example/      
https://www.javainuse.com/spring/springboot-microservice-elk


