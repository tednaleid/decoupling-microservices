# Decoupling Microservices

by <a href="https://twitter.com/tednaleid">@tednaleid</a>

!SLIDE shout
# Decoupling Microservices

## by <a href="https://twitter.com/tednaleid">@tednaleid</a>


!SLIDE shout quieter
# Overview<br/><span style="font-size:60%; text-align:left">Tight vs Loose Coupling<br/>What Kafka is/how it helps<br/>What Hollow is/how it helps</span>



!SLIDE quieter shout
# Microservices<br/><br/><span style="font-size:70%">&ldquo;loosley coupled Service-Oriented Architecture with bounded contexts&rdquo; <br/><br/>&mdash; Adrian Cockcroft (AWS/Netflix)</span>

!SLIDE shout
# Bounded Contexts

## from Domain-Driven Design, a conceptual model where <br/>a specific domain model applies


!SLIDE shout quieter

# Microservices all share a single domain model?

## is there a `.jar` file with domain classes?

!SLIDE shout quieter

# You've built a<br/>&ldquo;distributed monolith&rdquo;

## Watch: &ldquo;[Don't Build a Distributed Monolith](https://www.microservices.com/talks/dont-build-a-distributed-monolith/)&rdquo;<br/>by Ben Christensen (Facebook/Netflix)


!SLIDE shout
# Loosley Coupled

!SLIDE quietest shout
# Golden Rule of Component Reliability<br/><br/><span style="font-size: 80%">&ldquo;any critical component must be 10 times as<br/> reliable as the overall system's target, so that its contribution to system unreliability is noise&rdquo;</span>

## from: [&ldquo;The Calculus of Service Availability&rdquo;, ACM Queue, Vol 15, Issue 2](https://queue.acm.org/detail.cfm?id=3096459)


!SLIDE shout

# Tight Coupling

## Services directly calling services

!SLIDE light

<img src="images/tight_coupling.png" alt="" height="750px"/>

!SLIDE shout

# Looser Coupling

## some caching, reverse proxy, grace periods, circuit breakers<br/><br/>issues: cache invalidation rules, cold start/cache flush,<br/>emergency changes, race conditions

!SLIDE light

<img src="images/looser_coupling.png" alt="" height="750px"/>

!SLIDE shout

# Ideal Coupling

## totally functional during a partition<br/><br/>only thing more "loose" is no dependency at all

!SLIDE light
<img src="images/ideal_coupling.png" alt="" height="750px"/>

!SLIDE quieter shout

# #1 thing you can do for scalability and fault-tolerance?

!SLIDE quieter shout

#  separate your<br/>reads and writes

## don't do this if you don't need scalability &amp; fault-tolerance


!SLIDE quieter shout

# read requests are many orders of magnitude<br/>more frequent

## with exceptions (logging/metrics) where writes are <em>far</em> more frequent

!SLIDE quieter shout

# write requests have consequences

## cache-invalidation, notifications, etc

!SLIDE quieter shout

# decouple and scale reads with materialized views

## also called &ldquo;derived data&rdquo;

!SLIDE quieter shout

# optimized for the query patterns of each microservice

!SLIDE light 
# Separate Reads and Writes
<img src="images/read_write.png" alt="" height="600px"/>

!SLIDE shout

# Kafka

!SLIDE shout

# A log-based<br/>(append-only) message broker

## combines databases (durable storage) and messaging (queuing and publish/subscribe)

!SLIDE shout quieter

# Kafka brokers have few<sup>*</sup> moving parts<br/><br/><span style="font-size:60%">focused on speed, reliability, reasonability</span>

## <br/><br/><br/><br/>*compared to things like JMS, AMQP, RabbitMQ

!SLIDE shout 
# Producer (API)<br/>&darr;<br/>Kafka Broker<br/>&darr;<br/>Consumer (API)

!SLIDE shout
# Kafka cluster is made of many brokers

## uses Zookeeper for leader-election and broker metadata 

!SLIDE shout
# brokers have many named topics

## replication across brokers configured per topic 

!SLIDE light
# each topic has 1..N partitions
<img src="images/topic_partitions.png" alt="" height="500px"/>


!SLIDE 
# producers push data to a topic's partitions<br/><br/><br/>

    # send message {"id":"123", value: "foo"} with key "123"
    
    echo '123,{"id":"123", value: "foo"}' |\
        kafkacat -P -K ',' -b 127.0.0.1:9092 -t the-topic
        
# <br/><br/><span style="font-size:70%">message payload is binary data<br/>can be String/JSON/Avro/Protocol Buffers/Whatever</span>
      
!SLIDE 
# producers can publish lots of data quickly<br/><br/><br/>
        
        
    # send 100,000 messages with key 1 through 100000
    # and value {"id": "<#>", value: "bar"}
    
    seq 100000 |\
        awk '{printf "%s,{\"id\":\"%s\", value: \"bar\"}\n", $1, $1}' |\
        kafkacat -P -b 127.0.0.1:9092 -t the-topic -K ','
        
!SLIDE shout 

# consistent hashing<br/><br><span style="font-size:70%">messages with the same key always go to the same partition</span>


!SLIDE shout quieter
# consumers are pull-based<br/><br/><span style="font-size:70%">they maintain per-partition offsets</span>

## by default in a special topic called `__consumer_offsets`

!SLIDE shout quieter
# partitions are balanced across a consumer group
## max # of consumers for a topic is the number of partitions

!SLIDE shout quieter
# consumption is<br/>not destructive<br/><br/><span style="font-size:70%">messages have a retention period (default 24-hours)</span>

!SLIDE shout quieter
# message compaction keeps one message per key


!SLIDE light
# compaction keeps the latest value per key
<img src="images/compaction.png" alt="" height="650px"/>


!SLIDE shout

# How does Kafka enable decoupling?

!SLIDE shout quieter 

# if a consumer falls behind, the topic acts as a buffer and doesn't stop producers

!SLIDE shout quieter 
# multiple specialized consumers can be created<br/><br/><br/><span style="font-size:50%"><p>Elasticsearch index for searching</p><p>JSON payload in S3 buckets for SPA</p><p>logging/metrics driven off Kafka</p></span>

!SLIDE light

# scale to multiple availability zones, datacenters, or even multiple cloud providers

<img src="images/scale_materialized_views.png" alt="" height="500px"/>

## Kafka &ldquo;MirrorMaker&rdquo; can mirror the contents of a topic to other kafka clusters

!SLIDE shout
# materialized views can be thrown away and rebuilt

!SLIDE shout 

# great for blue/green deployments

## also for replicating data to lower environments

!SLIDE shout quieter 
# clear separation between writes and reads<br/><br/><span style="font-size:70%">triaging a bug?<br/>if it's in kafka, it's downstream, otherwise upstream</span>


!SLIDE light

# What can create materialized views?

<img src="images/materialized_view_creator.png" alt="" height="450px"/>

!SLIDE shout

# simple Java/Groovy app with Kafka consumer libraries

!SLIDE shout

# Akka Kafka<br/>Streams app<br/><br/><span style="font-size:60%">or Ratpack/RxJava/Spring Reactor something async</br>

!SLIDE shout
# stream processing framework like<br/>Spark, Flink, etc
 
## Apache alone has 10+ of these, but they tend to be _heavy_

!SLIDE shout

# Hollow<br/><span style="font-size:40%; line-height: 0px;">Netflix Java API for non-durable in-memory caches</span>

## Used in production for over 2 years

!SLIDE shout quieter

# <span style="font-size:85%">&ldquo;a way to compress your dataset in memory while still providing O(1) access to any portion of it&rdquo;</span> <br/><span style="font-size:50%">- Drew Koszewnik (lead contributor, Netflix)</span>

!SLIDE shout quieter
# Kilobytes to Megabytes, often Gigabytes,<br/>but not Terabytes

## works hard to minimize heap space and GC impact of updates

!SLIDE shout quieter
# not suitable for every kind of data problem<br/><br/><span style="font-size:70%">but great for the ones it is a fit for</span>

!SLIDE shout

# Single Producer<br/>Many Consumers

!SLIDE light

# Producer requires a Publisher<br/>and an Announcer

<img src="images/hollow_producer.png" alt="" height="500px"/>

!SLIDE light

# Consumer requires an AnnouncementWatcher and a BlobRetriever

<img src="images/hollow_consumer.png" alt="" height="400px"/>

!SLIDE shout

# Primary Use Cases


!SLIDE shout quieter

# read-heavy lookup data where objects change relatively frequently

## weekly/daily/hourly, but not every second

!SLIDE shout

# Netflix uses it for video metadata

!SLIDE shout

# why use this over memcached/redis or a full database?


!SLIDE shout

# initial load at startup then resilient to network partitions

!SLIDE shout quieter

# fewer running servers/moving pieces<br/>to be available


!SLIDE shout quieter

# faster response times,<br/>no network calls,<br/>just memory access


!SLIDE shout

# When should you use redis/memcached instead?


!SLIDE shout

# data size is<br/>quite large

!SLIDE shout

# data changes<br/>very frequently


!SLIDE shout

# data _must_ be consistent across servers


!SLIDE shout

# Other Considerations

!SLIDE shout quieter

# Single Producer<br/><br/><span style="font-size:70%">need to think about failover (zookeeper/etcd with leader election, multiple hot producers)</span>


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

## [dataintensive.net](http://dataintensive.net/) - ideas for &ldquo;read-after-write consistency&rdquo; solutions

!SLIDE shout
# @tednaleid

!SLIDE shout
# Questions?


