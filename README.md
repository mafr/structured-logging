# Structured Logging

This repository contains configuration for setting up Structured Logging
in Spring Boot based Java projects.


## Spring Boot

Add the dependency to
[logstash-logback-encoder](https://github.com/logstash/logstash-logback-encoder)
which gives us the mechanism to log data in JSON format:

```XML
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>4.9</version>
    </dependency>
```

If you have multiple Spring Boot services that cooperate to fulfill a request,
you can optionally add
[Spring Cloud Sleuth](http://cloud.spring.io/spring-cloud-sleuth/) to gain
correlation IDs:

```XML
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
```

No further setup is required for Sleuth unless you want to send data to
[Zipkin](http://zipkin.io/) as well.

Now set the `spring.application.name` property in your Spring Boot
configuration.  You can inject Maven's `project.name` into
`application.properties` or `application.yml` at compile time:

```
  spring.application.name = @project.name@
```

The application name is added to the JSON log object. It is also used by
Sleuth if set up to send data to Zipkin.


## Logback

This repository contains a `logback-spring.xml` file that overrides the
auto-configured logging configuration. It has appenders for the console,
a traditional plain text format, and the JSON log file. Remove the ones
that you don't need and adjust to include the data you need.

While there are some aspects specific to Spring Boot, with minor editing
the configuration file should be usable in other projects as well.


## Application Logging

Use the static factories on [StructuredArguments](https://github.com/logstash/logstash-logback-encoder/blob/master/src/main/java/net/logstash/logback/argument/StructuredArguments.java) to create log statements. For example: 

```Java
   LOG.info("{}; {} {}", value("action", "accepted order"),
	     keyValue("customer", customerId), keyValue("order", orderId));
```

In the JSON log file, this adds fields for "action", "customer", and "order"
among several other fields provided by the logging system. In the plain text
format this leads to the following output:

```
   ... INFO ... : accepted order; customer=4711 order=123456
```

Note that the key of the `value()` factory is only present in the JSON output,
not in the plain text format. Also, only arguments listed in the first
parameter of the log statement are part of the plain text format (in the
JSON file, all are present).

You may want to define your own static factories to `StructuredArgument` to
keep repetition minimal.


## Filebeat

Once the application logs data in JSON format, you can use the
[Filebeat](https://www.elastic.co/products/beats/filebeat) agent to
tail the log files and hand them off to another system, storing them
in ElasticSearch eventually. From there, you can search and visualize
the logs in [Kibana](https://www.elastic.co/products/kibana).

We need two different types of configuration for this:

   * Configuration tailing a log file and shipping it to ElasticSearch
   * ElasticSearch mappings matching the Logback logger configuration

