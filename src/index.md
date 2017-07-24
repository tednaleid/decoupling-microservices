# Decoupling Microservices

by <a href="https://twitter.com/tednaleid">@tednaleid</a>

!SLIDE shout
# Decoupling Microservices

## by <a href="https://twitter.com/tednaleid">@tednaleid</a>


!SLIDE quietest shout
# Microservices<br/><br/>&ldquo;loosley coupled Service-Oriented Architecture with bounded contexts&rdquo; <br/><br/>&mdash; Adrian Cockcroft (AWS/Netflix)

!SLIDE shout
# Bounded Contexts

## from Domain-Driven Design, a conceptual model where <br/>a specific domain model applies


!SLIDE shout quieter

# Microservices all share a single domain model?

!SLIDE shout quieter

# You've built a<br/>"distributed monolith"

## Watch: "[Don't Build a Distributed Monolith](https://www.microservices.com/talks/dont-build-a-distributed-monolith/)"<br/>by Ben Christensen (Facebook/Netflix)


!SLIDE shout
# Loosley Coupled

!SLIDE quietest shout
# Golden Rule of Component Reliability<br/><br/>any critical component must be 10 times as reliable as the overall system's target, so that its contribution to system unreliability is noise

## from: ["The Calculus of Service Availability", ACM Queue, Vol 15, Issue 2](https://queue.acm.org/detail.cfm?id=3096459)


!SLIDE shout

# Tight Coupling

## Services directly calling services

!SLIDE shout

# Looser Coupling

## some caching, reverse proxy, grace periods, circuit breakers<br/><br/>issues: cache invalidation rules, cold start/cache flush,<br/>emergency changes, race conditions


!SLIDE shout

# Ideal Coupling

## totally functional during a partition<br/><br/>only thing more "loose" is no dependency at all

!SLIDE quieter shout

# #1 thing you can do for scalability and fault-tolerance?

!SLIDE quieter shout

#  separate your<br/>reads and writes

## don't do this if you don't need scalability &amp; fault-tolerance


!SLIDE quieter shout

# read requests are likely many orders of magnitude more frequent

## with exceptions (logging/metrics) where writes are <em>far</em> more frequent

!SLIDE quieter shout

# write requests have consequences

## cache-invalidation, notifications, etc

!SLIDE quieter shout

# decouple and scale reads with materialized views

## also called "derived data"

!SLIDE quieter shout

# optimized for the query patterns of each microservice

!SLIDE quieter shout

# decouple and scale writes with event buffers

## example patterns include Event Sourcing/CQRS<br/>(Command Query Responsibility Segregation)

!SLIDE light 
# Separate Reads and Writes
<img src="images/read_write.png" alt="" height="600px"/>

!SLIDE shout

# Kafka

!SLIDE shout

# A log-based<br/>(append-only) message broker

## combines databases (durable storage) and messaging (queuing and publish/subscribe)

!SLIDE shout 
# Producer (API)<br/>&darr;<br/>Kafka Broker<br/>&darr;<br/>Consumer (API)

!SLIDE shout
# Kafka cluster is made of many brokers

## uses Zookeeper for leader-election and broker metadata 

!SLIDE shout
# Brokers have many named topics

## replication across brokers configured per topic 

!SLIDE light
# Each topic has 1..N partitions
<img src="images/topic_partitions.png" alt="" height="500px"/>


!SLIDE 
# Producers push data to a topic's partitions<br/><br/><br/>

    # send message {"id":"123", value: "foo"} with key "123"
    
    echo '123,{"id":"123", value: "foo"}' |\
        kafkacat -P -K ',' -b 127.0.0.1:9092 -t the-topic
        
## <br/><br/><br/>Message payload is binary data<br/>Can be String/JSON/Avro/Protocol Buffers/Whatever
      
!SLIDE 
# Producers can publish lots of data quickly<br/><br/><br/>
        
        
    # send 100,000 messages with key 1 through 100000
    # and value {"id": "<#>", value: "bar"}
    
    seq 100000 |\
        awk '{printf "%s,{\"id\":\"%s\", value: \"bar\"}\n", $1, $1}' |\
        kafkacat -P -b 127.0.0.1:9092 -t the-topic -K ','
        
!SLIDE shout 

# consistent hashing<br/><br><span style="font-size:70%">messages with the same key always go to the same partition</span>


!SLIDE shout quieter
# Consumers are pull-based<br/><br/><span style="font-size:70%">they maintain per-partition offsets</span>

## by default in a special topic called `__consumer_offsets`


!SLIDE shout quieter
# Consumption is not destructive<br/><br/><span style="font-size:70%">messages have a retention period (default 24-hours)</span>


!SLIDE shout quieter
# within a consumer group there can be as many consumers as there are partitions


!SLIDE shout quieter
# Message compaction keeps one message per key



!SLIDE light
# Compaction keeps the latest value per key
<img src="images/compaction.png" alt="" height="650px"/>


!SLIDE shout quieter

# Kafka Brokers have few<sup>*</sup> moving parts<br/><br/><span style="font-size:60%">focused on speed, reliability, reasonability</span>

## <br/><br/><br/><br/>*compared to things like JMS, AMQP, RabbitMQ

!SLIDE shout

# How does Kafka enable decoupling?

!SLIDE shout light

enables event-based integration between microservices 

!SLIDE shout light

if a consumer falls behind the event log acts as a buffer and doesn't stop producers

!SLIDE shout light
many consumers per partition

!SLIDE shout light
multiple specialized consumers can be created (ES for search, JSON payload in S3 buckets for SPA, logging/metrics driven off kafka)

!SLIDE shout light
clear separation point, can more easily determine where errors are happening, during triage, check kafka, if it's there you know the issue is after kafka, else before

!SLIDE shout light
materialized views can be easily rebuilt 

!SLIDE shout

# Async event log<br/><br/>many writers<br/>many readers

!SLIDE shout

# scale to multiple availability zones, datacenters, or even multiple cloud providers

!SLIDE shout

# great for Blue/Green Deployments

new fields get added to write side first
then modify materialized view creator so it has the new fields
in whatever format it wants, spin up a new green cluster of materialized view creator
that creates new green materialized views and then new green web services that read from those
new materialized views, and you’re done


if there is an issue, when you flip over to green, you can have left your old blue
cluster up, still populating the old way of doing things and can switch back as quickly as
flipping your load balancers back over


!SLIDE shout

What should the materialized view creator be?

akka app
simple java app/ratpack app with kafka client consumer libraries
stream processing framework like flink, spark, etc (apache has 10+ of these alone but they tend to be _heavy_)

need good lag monitoring, pretty easy to do with kafka API or something like burrow

hollow (now transition to hollow)

!SLIDE shout

# Hollow


!SLIDE shout

# Homework


!SLIDE

# Places to Start<br/><br/>[hollow-reference-implementation](https://github.com/Netflix/hollow-reference-implementation)<br/><br/>try hollow for simple lookup data<br/><br/>[kafka/zookeeper docker-compose](https://github.com/wurstmeister/kafka-docker/blob/master/docker-compose.yml) + [kafkacat](https://github.com/edenhill/kafkacat)<br/><br/>kafka + telegraf/influx for metrics

!SLIDE shout

# Resources


!SLIDE shout

# Learn More Kafka<br/><br/>[Confluent's Blog](https://www.confluent.io/blog/)

## [https://www.confluent.io/blog/](https://www.confluent.io/blog/)

!SLIDE shout

# Learn More Hollow<br/><br/>[Hollow's Docs](http://hollow.how/)

## [http://hollow.how/](http://hollow.how/)

!SLIDE shout

<img src="images/DDIA.png" alt="" height="480px"/>
## [dataintensive.net](http://dataintensive.net/) - ideas for "read-after-write consistency" solutions




!SLIDE shout
# @tednaleid

!SLIDE shout
# Questions?


