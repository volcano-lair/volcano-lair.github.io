---
layout: default
title: Home

permalink: /
icon: home
order: 1
---
{% include links.md %}

# {{ site.name }}

**Spectre is a federated platform composed of discrete, interoperable services.**

The objective of the Spectre project is to provide a secure, distributed, scalable platform upon which to build distributed services in a flexible, inexpensive, language-agnostic fashion.

**Spectre is not a server.**

The Spectre project is a collection of services that solve common problems associated with scaling and securing distributed, ephemeral infrastructure (e.g. hosted compute, e.g. The Cloud™). Thus, one can not "deploy Spectre". The services and their respective code-bases that reside under the umbrella of the Spectre project should be referred to be their specific names whenever possible to avoid mischaracterization.

## Model

There are several basic concepts that underpin all services in the Spectre project. They are outlined with some context, below, and explained in further detail thereafter:

* Spectre services can be grouped into two topological categories: **Core** and **Edge**
* All Spectre services exist in a **Datacenter**
* Some **Core** Spectre services are able to communicate between **Datacenters**.

#### Instance

* An Instance is a singular, isolated unit of compute; likely a VM in a hosting environment. Instances are considered to be secure containers; that is, various processes running on an instance inherently trust one another's IPC requests (e.g. localhost connections).
* Instances' disk and memory resources **are not considered secure**. Cryptographic identifiers and keys issued to instances should be unique and single tenant (only issued to a single instance for all of time), temporary and auto-expiring, and revocable.
* All secret factors issued to instances **must be** logged centrally by a non-secret identity key for later audit.

#### Datacenter

* A Datacenter is a collection of instances with fairly robust, high bandwidth, low latency network connectivity. An Amazon AWS VPC is a domain-specific example of this.
* Datacenters' local network links (or virtual equivalents) **are not considered secure**. All connections between instances within and between datacenters must be encrypted and authenticated by both peers.
* Instances may safely assume some availability of services within the same datacenter, but **must** be able to handle failures gracefully and retry requests in the future.
* Instances **may not** rely upon resources in remote datacenters, nor can they make any assumption of network reliability between datacenters.

#### Edge

* Edge-topology services run on instances with other services.
* Edge-topology services may provide unauthenticated, unencrypted service only to local consumers. [Tokend], [Propsd], and the [Consul] agent are edge services.
* Edge-topology services may provide authentication/authorization gateways for external consumers to access local services. This service must only listen on encrypted channels ([Turnstile]).
* Edge-topology services are aware of the role and location of the instance on which they run, and scope their behavior to that context.
* Some edge-topology services run on core-topology service instances.

#### Core

* Core-topology services run on clusters of dedicated instances ([Consul] server, [Vault], [Notary]).
* Core-topology services provide service to other instances in their respective datacenter.
* Some core-topology services may interact securely with their peers in other datacenters ([Consul] server, [Notary]).
* Core-topology services must require clients to authenticate.
* Core-topology services must only provide encrypted interfaces to consumers.
