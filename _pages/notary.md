---
layout: default
title: Notary
repo: 'volcano-lair/night-watch'
---
{% include links.md %}

Notary is a shared-key management service. It issues keys to clients, and validates request signatures for servers. This model ensures that clients need never share their secret keys with the servers that they are interacting with, and removes any requirement for implicit trust between services.

## Shared Keys

The keys that Notary issues to clients have an identifier and a secret token. Clients share their identifiers with other resources to assert their identities, and generate signatures with their secret tokens to serve as non-reversible authentication factors for their identities. Clients must never share their secret tokens with any other resources.

## Key Issuing and Storage

EC2 instances authenticate with Notary using the same identity document and PKCS7 signature as required by Warden. Notary generates new random secret token and identity strings for the key, and looks up the roles that should be associated with the key from [Politician]. The Notary service stores a copy of the new key, including the secret token, in its own database for later validation. Keys are issued with a TTL, in seconds. Clients must periodically request renewal of their keys to keep their respective TTLs from expiring, and invalidating the key.

The complete Key object

```json
{
  "identity": {
    "v": 1,
    "dc": "vpc-8de77a22c",
    "id": "t-18ad7e2df2d79a5d"
  },
  "secret": "JxC9rAORkUGp7dri29kgyQf1V5JI6kAHHrR560UFj3qIsc4u7qwI0Y96znVSh5pp",
  "roles": [],
  "ttl": 300
}
```

gets encoded into

```json
{
  "identity": "dj0xOnZwYy04ZGU3N2EyMmM6dC0xOGFkN2UyZGYyZDc5YTVk",
  "secret": "JxC9rAORkUGp7dri29kgyQf1V5JI6kAHHrR560UFj3qIsc4u7qwI0Y96znVSh5pp",
  "roles": [],
  "ttl": 300
}
```

before it is returned to the client. The `identity` field is the `base64` encoding of the ASCII encoded packed representation of the original `identity` object:

```
v=1:vpc-8de77a22c:t-18ad7e2df2d79a5d
```

## Role Bindings

When notary generates a new key, it also queries [Politician] for the roles that should be associated with the authenticated client identity. For EC2 instances, the identity is a tuple of the instance's image/autoscaling-group, VPC, and account. Notary saves the instance's roles with the new key in its database and returns the list of roles to the instance with the new key.

## Signature Verification

Upon receiving a signed request from a client, servers must send the included identity, signature, and request parameters used to generate the signature to a Notary server, which will use its copy of the client's secret token to validate the signature and respond to the server with the result. If valid, Notary will include the roles associated with the key in its response, allowing the server to query Politician for the policies that should be applied to requests with that key.

## Datacenter Federation

Notary's keys are globally unique and valid across all datacenters. Clients' keys are generated and stored by Notary instances in the clients' own respective datacenters. When a client signs a request to a service in a remote datacenter, the receiving server requests validation from Notary instances in it's own datacenter.

#### Key Caching

Upon receiving a signing request, the Notary instance (called the `verifying instance`) decodes the identity string and checks the datacenter identifier. If the key originated in a remote datacenter that the verifying Notary instance can authenticate with, the verifying instance will request the key from a Notary instance (called the `issuing instance`) in that datacenter, and cache retrieved key for the remainder of its TTL.

If a key cached on a verifying instance is re-used frequently during its TTL period, it may be optimistically re-verified from it's issuing instance and its cache TTL extended to keep ongoing verifying latency to a minimum.

#### Authentication

Notary instances authenticate with one another across datacenters with one or more central Notary clusters, called `authority clusters`. Notary instances authenticate with and acquire keys from each authority cluster on startup using the same interface that other instances within the local datacenter use to acquire their own keys.

Cross-datacenter requests for keys must be signed themselves by a key with a policy that allows `notary:key-federation`. The authority cluster(s) are responsible for issuing these keys to Notary instances in other datacenters according to constraints and policies defined by [Politician]
