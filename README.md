# Developement Elk Stack

This project contains a basic docker-compose ELK stack.

## Table of Contents
* [Contributor covenant code of conduct](#contributor-covenant-code-of-conduct)
* [Pipeline](#pipeline)
* [Architecture](#architecture)
  * [ELK Stack Diagram](#elk-stack-diagram)
  * [ELK Stack Scripts Diagram](#elk-stack-scripts-diagram)
  * [Persistant Redis Script Sequence Diagram](#persistant-redis-script-sequence-diagram)
  * [Persistant Redis Sequence Diagram](#persistant-redis-sequence-diagram)
* [Metricbeat vSphere module](#metricbeat-vsphere-module)
* [Packetbeat module configuration](#packetbeat-module-configuration)
* [Run the stack](#run-the-stack)
* [Docker compose](#docker-compose)
  * [Install and release](#install-and-release)
  * [Working with containers and images](#working-with-containers-and-images)
  * [Publish a docker image](#publish-a-docker-image)
* [Drone](#drone)
  * [Drone Docker Flow Diagram](#drone-docker-flow-diagram)
  * [Drone Docker Hierarchy Diagram](#drone-docker-hierarchy-diagram)
  * [Drone setup](#drone-setup)
* [Common bugs](#common-bugs)
* [Tools](#tools)
* [Links](#links)
  * [Drone](#drone)
  * [Docker](#docker)
  * [Redis](#redis)
  * [Beats](#beats)
  * [Markdown](#markdown)

## Contributor covenant code of conduct

Please acknowledge our [code of conduct](./CODE_OF_CONDUCT.md).
  
## Pipeline

(user) => kibana => elasticsearch => logstash => redis => proxy => filebeat/metricbeat => (data)

  Container | Description | Port
------------ | ------------- | -------------
[Kibana](https://www.elastic.co/products/kibana) | Open source data visualization plugin for Elasticsearch | 5601 (exposed)
[Elasticsearch](https://www.elastic.co/products/elasticsearch) | Search engine based on the Lucene library | 9200 (exposed)
[Logstash](https://www.elastic.co/products/logstash) | Open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash.” | 5044 (not exposed)
[Redis](https://redis.io/) | Open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker | 6379 (exposed)
[Proxy](https://www.elastic.co/products/logstash) | Open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash.” | 5044 (not exposed)
[Filebeat](https://www.elastic.co/products/beats/filebeat) | Lightweight Shipper for Logs | default (not exposed)
[Metricbeat](https://www.elastic.co/products/beats/metricbeat) | Lightweight Shipper for Metrics | default (not exposed)
[Packetbeat](https://www.elastic.co/products/beats/packetbeat) | Lightweight Shipper for Network Data | default (not exposed)
[Auditbeat](https://www.elastic.co/products/beats/auditbeat) | Lightweight Shipper for Audit Data | default (not exposed)

> Note: Some of the containers listed here are commented out in the dokcer-compose.yml file. You can use them by uncomment them.

There's 3 queues between proxy and redis:

     1)UL_GEN => data_type: "list"

      2)UL_REPLAY_ENA => data_type: "list"

      3)UL_REPLAY_MPO => data_type: "list"

## Architecture

A complete description of this stack architecture can be found below.

### ELK Stack Diagram

![ELK_Stack_Diagram](Documentation/ELK_Stack/ELK_Stack_Diagram.jpg?raw=true "ELK_Stack_Diagram")

### ELK Stack Scripts Diagram

![ELK Stack Scripts Diagram](Documentation/ELK_Stack/ELK_Stack_Scripts_Diagram.jpg?raw=true "ELK Stack Scripts Diagram")

### Persistant Redis Script Sequence Diagram

![Persistant Redis Script Sequence Diagram](Documentation/Persistant_Redis/PersistantRedisScriptsSequenceDiagram.jpg?raw=true "Persistant Redis Script Sequence Diagram")

### Persistant Redis Sequence Diagram

![Persistant Redis Sequence Diagram](Documentation/Persistant_Redis/PersistantRedisSequenceDiagram.jpg?raw=true "Persistant Redis Sequence Diagram")

## Metricbeat vSphere module

A complete description of this module and it's fields can be found here: 
[vSphere](VSPHEREMODULE.md)

## Packetbeat module configuration

To enable packetbeat to sniff packets in a container inside the stack (in this example the redis container) this as to be done in the docker-compose.yml file.

```
cap_add: ['NET_RAW', 'NET_ADMIN']
network_mode: "container:redis"
```

## Run the stack

In the same command line session where you previously exported your environnement variables:

- Run with output
```
$ docker-compose up
```

- Run without output
```
$ docker-compose up -d
```

- Follow the output when the stack runs in daemon (-d)
```
$ docker-compose logs -f | grep -v kibana_1
```

- Restart filebeat and reinject all logs (while the stack is up)
```
$ docker-compose stop filebeat; rm filebeat/config/data/registry; docker-compose start filebeat
```

- Monitor redis
```
$ redis-cli --stat
```

## Docker compose

### Install and release

[Install](https://docs.docker.com/compose/install/)

[Release](https://github.com/docker/compose/releases)

### Working with containers and images

- Open container
```
$ docker exec -ti [container name(ex: bd04...)] /bin/bash
```

- Remove all containers and images
```
#!/bin/bash
# Delete all containers
$ docker rm $(docker ps -a -q)
# Delete all images
4 docker rmi $(docker images -q)
# Maintenant la commande purge existe:
$ docker [image|container|volume|network|system] purge
```

- List stopped containers
```
docker ps -q --filter "status=exited"
```

- Remove stopped containers
```
docker rm $(docker ps -q --filter "status=exited")
```

- Clean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container)
```
docker system prune
```

- Remove dangling volumes: 
[Remove dangling volumes](https://stackoverflow.com/questions/27812807/orphaned-docker-mounted-host-volumes)

- List all orphaned volumes and Eliminate all of them
```
$ docker volume ls -qf dangling=true
$ docker volume rm $(docker volume ls -qf dangling=true)
```

- Alias to have the container name in stats
```
alias ds='docker stats --format "table {{.Name}}\t{{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}\t{{.MemPerc}}\t{{.PIDs}}"'
```

### Publish a docker image

An example of a script used to publish docker images from their respective Dockerfile.
[Build Image And Push Script](https://gitea.avd.ulaval.ca/AVD/avddockers/src/branch/master/drill/buildimage_and_push.sh)


## Drone

### Drone Docker Flow Diagram

The common development flow while working with drone in this stack.

![Drone_Docker_Flow_Diagram](Documentation/Drone/Pretty_Drone_Docker_Flow_Diagram.jpg?raw=true "Drone Docker Flow Diagram")

<!-- ![Drone_Docker_Flow_Diagram](Documentation/Drone/Drone_Docker_Flow_Diagram.jpg?raw=true "Drone Docker Flow Diagram") -->

### Drone Docker Hierarchy Diagram

The docker-compose eand Dockerfile hierarchy of this stack.

![Drone Docker Hierarchy Diagram](Documentation/Drone/Drone_Docker_Hierarchy_Diagram.jpg?raw=true "Drone Docker Hierarchy Diagram")

### Drone setup

- Setup Drone In Your Project (Tutorial):
[How To Set Up Drone](https://www.digitalocean.com/community/tutorials/how-to-set-up-continuous-integration-pipelines-with-drone-on-ubuntu-16-04)

- Setup Drone steps:
[Drone Pipeline Steps](https://docs.drone.io/config/pipeline/steps/)

- Using Drone with python:
[Drone Python](https://github.com/drone/drone-python/blob/master/.drone.yml)

## Common bugs

- Low cluster health: Can be fixed with API requests as described here: 
[Elasticsearch API](https://gitea.avd.ulaval.ca/ember89/stack_elk_proxy_redis/src/commit/09d360490bbf997d7106b4f81b815e512c9f5855/ELASTICSEARCH_API.md)

## Tools

- [AVDCLI redis tools](https://gitea.avd.ulaval.ca/ember89/avdcli_redis_tools)

- [Docker compose](https://docs.docker.com/compose/install/)

- [Redis desktop manager](https://redisdesktop.com/)

- [Redis command line tool](https://redis.io/topics/rediscli)

- [Draw.io](https://www.draw.io/)

## Links

### Drone

- [Drone official documentation](https://drone.io/)

- [Set Up Continuous Integration Pipelines with Drone on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-continuous-integration-pipelines-with-drone-on-ubuntu-16-04)

### Docker

- [Pass environment variables to containers](https://docs.docker.com/compose/environment-variables/#pass-environment-variables-to-containers)

- [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

- [10 tips for docker-compose with multiple dockerfiles](https://nickjanetakis.com/blog/docker-tip-10-project-structure-with-multiple-dockerfiles-and-docker-compose)

- [Multiple dockerfiles in project](https://stackoverflow.com/questions/27409761/multiple-dockerfiles-in-project)

- [Docker ELK stack for devops](https://medium.com/tech-tajawal/elk-stack-docker-playground-for-devops-221179ca00dd)

### Redis

- [Top-5 redis performance metrics](https://www.datadoghq.com/pdf/Understanding-the-Top-5-Redis-Performance-Metrics.pdf)

- [Official redis documentation](https://redis.io/)

- [Redis with python (redis library documentation)](https://redis-py.readthedocs.io/en/latest/)

### Beats

- [Filebeat](https://www.elastic.co/products/beats/filebeat)

- [Metricbeat](https://www.elastic.co/products/beats/metricbeat)

- [Packetbeat](https://www.elastic.co/products/beats/packetbeat)

- [Auditbeat](https://www.elastic.co/products/beats/auditbeat)

### Markdown

- [Mastering markdown guide](https://guides.github.com/features/mastering-markdown/)

- [Markdown real-time collaboration tool](https://hackmd.io/)
