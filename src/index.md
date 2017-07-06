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

# scale reads with materialized views

## also called "derived data"

!SLIDE quieter shout

# optimized for the query patterns of each microservice

!SLIDE light 
# Separate Reads and Writes
<img src="images/read_write.png" alt="" height="600px"/>

!SLIDE shout

# Kafka



what is is

log-based (append-only) message broker

combines databases (durable storage) and messaging (notifying consumers)

retention

key-based compaction

topics -> partitions

producers append
consumer groups offsets (__consumer_offsets)


messages can be anything, text, json, binary, avro, protocol buffers

(picture here )


`kt consume -brokers 10.63.103.135:9092 -topic my-topic -offsets oldest`
```{"partition":0,"offset":770354,"key":"7768365","value":"<text/json/binary/whatever"}```



how does it enable decoupling?

!SLIDE shout light

# enables event log-based integration between microservices 


if a consumer falls behind the event log acts as a buffer and doesn't stop producers

multiple specialized consumers can be created

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

!SLIDE shout light
!SLIDE shout light
!SLIDE shout light

!SLIDE quietest
# A list
  - item 1
  - item 2
  - item 3

!SLIDE quietest

# Some code

    "foo".bar().baz()

<br/>
<br/>
<br/>
<br/>
<br/>
<span style="font-size:50%; float: right;"><a href="https://google.com"> and a link</a></span>

!SLIDE quietest shout
# some text <br/><br/>[highlight1]() | [highlight2]() | <br/><br/>some more text

