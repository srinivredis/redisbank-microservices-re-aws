# About this repository

This repository uses Redis core data structures, Streams, RediSearch, RedisJSON and TimeSeries to build a distributed
Java/Spring Boot/Spring Data Redis Reactive application that shows a searchable transaction overview with realtime updates
as well as a personal finance management overview with realtime balance and biggest spenders updates. UI in Bootstrap/CSS/Vue.

Features in this demo:

- Redis Streams for the transactions events
- Redis TimeSeries for the balance over time
- RediSearch for searching transactions
- Sorted Sets for the 'biggest spenders'
- RedisJSON for storing transactions and accounts
- Redis hashes for session storage (via Spring Session)

## Architecture
Contrary to the monolithic version of this application (<https://github.com/Redislabs-Solution-Architects/redisbank>), this repo is setup in a distributed micro-services fashion. This allows components/features to be deployed individually and distributed across clouds/VMs/k8s clusters as desired. See diagram for more info.

## Prerequisites

1. JDK 17 or higher (<https://openjdk.java.net/install/index.html>). Not needed when using Docker.
1. Docker Desktop (<https://www.docker.com/products/docker-desktop>), or Colima with a docker/k8s/containerd runtime.
1. For running on Kubernetes: a Kubernetes cluster
1. A running Redis Stack (<https://redis.io/docs/stack/>) instance or Redis Enterprise (<https://redis.com/redis-enterprise-software/overview/>) cluster (not needed for building/running tests as TestContainers is used there)

## How to run

### EKS

For now this repo has files for deploying on EKS, other k8s distros to be added in the future. Note: this will most likely work on just about any k8s distro with a few modifications here and there. Expect more documentation here in the future.


### Questions, support, issues?
Hit that `New Issue` button, or reach out to the author directly.