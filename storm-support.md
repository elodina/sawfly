# Apache Storm Bolts & Spouts for Apache Kafka on Apache Mesos & DC/OS with the Elodina Stack Deploy

## Overview
This repo contains various supporting pieces for running Kafka-based topologies in the Elodina environment. These pieces
are:
 
- Kafka 0.8 and 0.9 bolt
- Kafka 0.8 and 0.9 spout
- Sample Kafka mirroring topologies
- Marathon jsons to run Storm Nimbus and Storm UI
- Vagrant image. See `/vagrant` dir for details

**Supported version:** Currently supported Storm version is 0.10.0

## Bolts and spouts Usage

The provided bolts and spouts are contained in the appropriate java libraries. There are two libraries this project 
provides. For creating Kafka 0.8 topologies - `storm-kafka-08-java`, and for the Kafka 0.9 - `storm-kafka-09-java`. 
There are several ways of using the libraries in the newly created topologies. Some of them are:

### Directly from this project
Simplest way to build topologies using the libraries is to create a topology inside this project. In order to do this, 
simply write the topology code using the provided pieces, then create Bazel build configuration similar to the one from
the example topologies: `examples/BUILD`

### From other Bazel project
There are two ways to use the provided libraries in the different project. First, is to check out this repo to 
your machine and use this project as the local repository. In order to do this, download this repo and specify the 
following in your project's `WORKSPACE` file:

```
local_repository(
    name = "elodina-storm-support",
    path = "<local_filepath_to_checked_out_repo>",
)
```

After that, you may add the libraries to your Bazel project by specifying these in the `deps` args of the target you're 
building: `@elodina-storm-support//storm-kafka:storm-kafka-08-java` or 
`@elodina-storm-support//storm-kafka:storm-kafka-09-java` 

Other way around is to use this project as an HTTP archive dependency. Provided this repo is compressed to archive and 
served online, you may use it by adding the following in your `WORKSPACE` file:

```
http_archive(
    name = "elodina-storm-support",
    url = "<http_address_of_the_archive>.zip",
    sha256 = "<sha256_sum>",
)
```

Then, as in the previous case, the library targets will look like 
`@elodina-storm-support//storm-kafka:storm-kafka-08-java` or `@elodina-storm-support//storm-kafka:storm-kafka-09-java`

**Note:** Generally, it is recommended to use `local_repository` approach, because that way if you'll need to exclude 
some transitive dependencies, on which Storm Kafka relies, from the resulting topology fat jar, you'll simply need to 
set the want-to-exclude library's `neverlink` parameter to 1.  

### From different build system
It's kind of trickier to use the Storm Kafka libraries outside of the Bazel environment, mostly because you can't manage
transitive dependencies across the build systems, and I haven't seen solutions to build something like pom.xml out of 
Bazel library. So in order to use the library efficiently, you'll need to first build library jar without dependencies:

```
bazel build //storm-kafka:storm-kafka-08-java
# or
bazel build //storm-kafka:storm-kafka-09-java
```

The library jars then will be found at these paths:`bazel-bin/storm-kafka/libstorm-kafka-08-java.jar` and 
`bazel-bin/storm-kafka/libstorm-kafka-08-java.jar`. Include this to your build, and then include all the libraries 
Storm Kafka depends on manually. For the list of these libraries, please see `storm-kafka/BUILD` file. Note, that some
of the libraries are neverlink'ed, thus only used for compilation, and generally should be excluded from the resulting 
jar.

# Stack Deploy

These stacks deploy the whole Storm infrastructure and showcase deployment of all necessary infrastructure pieces 
required to get example topologies going. Designed to not interfere with any infrastructure pieces already deployed and
to be easy to clean-out and re-deploy.

## Deployed pieces

- Exhibitor on Mesos
- Exhibitor node
- Kafka on Mesos
- Kafka 0.9 broker
- Storm Nimbus 0.10.0
- Storm UI 0.10.0

On top of this, sample 0.9 mirror topology is submitted

**NOTE:** Exhibitor-Mesos framework in these stacks holds its data in the json file. Thus, it is not necessary to 
manually kill any Zookeeper nodes to restart this stack. Although, due to this specification one shouldn't use this 
stack as is in production, Zookeeper should actually be used as a framework storage in production. --storage can be changed per the [exhibitor mesos framework](https://github.com/elodina/exhibitor-mesos-framework#scheduler-configuration).

## Usage

```
# Considering one has stack deploy running on a server and SD_API, SD_USER and SD_KEY set up

# DCOS clusters:
./stack-deploy add --file dcos-storm-full-09.stack
./stack-deploy run storm-topology-full-dcos --var "storm-topology-name=mirror" --var "source-topic=foo" --var \
"target-topic=bar"

# Mesos Consul enabled clusters:
./stack-deploy add --file storm-full-09.stack
./stack-deploy run storm-topology-full --var "storm-topology-name=mirror" --var "source-topic=foo" --var \
"target-topic=bar"
```

# Local Vagrant
This vagrant environment allows running test topologies.

By default it deploys following components:
1. Mesos master + N slave(s);
2. Marathon;
3. Kafka-Mesos with Kafka 0.8.x / Kafka-0.9.x
4. Storm-Mesos with Nimbus & UI

Kafka-Mesos and Storm-Mesos are running in Marathon.

# Managing Kafka
Kafka mesos CLI and Kafka distros are installed in vagrant home dir.
After vagrant is up, `/home/vagrant` dir contains following sub-dirs:
- kafka-08 - for kafka-mesos with Kafka 0.8.x distro;
- kafka-09 - for kafka-mesos with Kafka 0.9.x distro;

By default no kafka brokers are created. Just Schedulers are running.
In order to add broker kafka-mesos cli should be used. Example:

```
# ssh vagrant@master
# cd kafka-08
# ./kafka-mesos.sh broker add 0 --cpus=0.1 --mem=100 --heap=100
broker added:
  id: 0
  active: false
  ...

# ./kafka-mesos.sh broker start 0
broker started:
  id: 0
  active: true
  state: running
  resources: cpus:0.10, mem:100, heap:100, port:auto
  failover: delay:1m, max-delay:10m
  stickiness: period:10m, hostname:master
  task:
    id: broker-0-d278ca52-1945-4e70-9e31-30bb85563761
    state: running
    endpoint: master:5000
```
Now broker of Kafka 0.8 is listening on master:5000.

For additional details of managing brokers, please refer to
https://github.com/mesos/kafka

# Launching topologies

## Build
In order to verify Vagrant image working you may launch the example topologies on the Vagrant image provided. To do so,
at first you'll need to actually build the topologies you want to run. Note that prerequisites to this would be having
a Bazel installation on your machine. Go to http://bazel.io for details on running it. After making sure to have Bazel 
installed, run the following:

```
# For 0.8 topology:
bazel build //examples:storm-kafka-08-mirror_deploy.jar

# For 0.9 topology:
bazel build //examples:storm-kafka-09-mirror_deploy.jar
```

## Launching
Then you'll need to submit a topology jar through Storm CLI of a proper version. So make sure to have Storm 0.10 on your
machine. You may grab one here: https://storm.apache.org/downloads.html In order to launch a topology:

```
# For 0.8 topology:
.<path_to_storm_distribution>/bin/storm jar -c nimbus.host=master -c nimbus.thrift.port=31056 \
bazel-bin/examples/storm-kafka-08-mirror_deploy.jar net.elodina.storm.KafkaOldMirrorTopology <topology_name> \
<bootstrap_broker> <source_topic> <target_topic> master:2181 /kafka-08
 
# For 0.9 topology:
.<path_to_storm_distribution>/bin/storm jar -c nimbus.host=master -c nimbus.thrift.port=31056 \
bazel-bin/examples/storm-kafka-09-mirror_deploy.jar net.elodina.storm.KafkaMirrorTopology <topology_name> <bootstrap_broker> \
<source_topic> <target_topic> 
```

## Verification
In order to verify correct work of the topology, one may use standard Kafka CLI clients which can be found in the 
vagrant home dir. For Kafka 0.9 execute:

```
vagrant ssh master

cd kafka-09
tar -zxf kafka_2.10-0.9.0.0.tgz
cd kafka_2.10-0.9.0.0/bin

# Type some messages into the opened shell here:
./kafka-console-producer.sh --topic <source_topic> --broker-list <bootstrap_broker>
 
# You should see your messages when consuming from the target topic here:
./kafka-console-consumer.sh --topic <target_topic> --bootstrap-server <bootstrap_broker> --new-consumer --from-beginning
```

For Kafka 0.8:

```
vagrant ssh master

cd kafka-08
tar -zxf kafka_2.10-0.8.2.2.tgz
cd kafka_2.10-0.8.2.2/bin

# Type messages here. 
./kafka-console-producer.sh --topic <source_topic> --broker-list <bootstrap_broker> --new-producer

# You should see your messages when consuming from the target topic here:
./kafka-console-consumer.sh --zookeeper master:2181/kafka-08 --topic <target_topic> --from-beginning
```

**NOTE:** you may need to send at least several messages to Kafka 0.8 producer for them to show up at the target topic,
 as 0.8 consumer reads messages with certain buffer length. When received at the topology end it doesn't flush 
 immediately
