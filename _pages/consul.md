---
layout: default
title: Consul
---
{% include links.md %}

Consul is an open-source [Hashicorp project](https://consul.io). It implements a distributed gossip network, synchronous RPC channels, and consistent RAFT storage between agents deployed to all instances, and a central cluster of servers in a datacenter.

## Datacenters

Consul servers form a cluster local to a datacenter, called a `LAN Cluster`. Datacenters are conceptual analogies to a traditional physical-plant model, in which servers in a physical datacenter typically have high-throughput, low-latency interconnects, relative to Internet links. Applied to hosted compute environments, a Datacenter maps to a Virtual Private Cloud (VPC), or hosting region, in which virtual machines share the same virtual network realm.

A Consul server cluster can also form a global cluster across [multiple Datacenters][consul datacenters], called a `WAN Cluster`. Consul agents may specify Datacenter identifiers when making requests to their LAN server cluster, which the local servers will forward to the respective Datacenter's server cluster for processing.

## Registration, Health Checking, and Discovery

Consul provides a partition-tolerant, consistent catalog service. Agents report local services to their LAN server cluster, which synchronizes and presents a consistent view of the Datacenter to consumers.

Agents associate health checks with the services that they announce, and consumers may specify filters to exclude unhealthy or healthy nodes from their query results. By default, all services are bound to their agent's own health check; when the LAN server cluster looses contact with an agent, it will mark all of the agent's services unhealthy, and remove them from the catalog when the agent leaves the cluster, or times our after a catastrophic failure.

Consumers may specify a Datacenter identifier to query a remote Datacenter's catalog. This provides all services with a global catalog from which to query dependencies.

## Distributed Locking

Consul servers' RAFT consensus provides a synchronous, distributed [locking service][consul locking] to consumers. This is extremely useful for configuring and maintaining other distributed services that do not, themselves, have a locking mechanism.
