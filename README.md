# Structured Logging

This repository contains configuration for setting up Structured Logging
in Spring Boot based Java projects.


## Logback for Spring Boot

Logger configuration with some Spring Boot specifics, including data from
Spring Cloud Sleuth.


## Filebeat

Log shipping to ElasticSearch:

   * ElasticSearch mappings matching the Logback logger configuration
   * Configuration tailing a log file and shipping it to ElasticSearch

