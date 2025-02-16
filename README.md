# kgstf.arch

This repository defines the architecture for terraform/tofu modules defined in this account.

It's a work in progress, right now!

## Table of Contents

1. [Structure](#structure)
2. [Security](#security)
  1. [Zones](#zonal-security-groups)


## Structure

![Tiers](./images/arch-tiers.svg)

Four nested tiers are defined for services within the architecture. These are:

* Organisation: Bottom level objects giving the organisational structure. Examples: AWS Organizations, corporate networking, corporate IAM, etc
* Environment: Objects defining the scope of a specific environment. Examples: AWS Account(s), VPCs/virtual networks, logging/auditing blob storage
* Zone: Security zone and associated objects within the scope of an environment. Examples: Subnets, compute clusters (e.g. kubernetes), route tables, NAT
* Service: A specific service and associated objects. Examples: Databases, pods, compute instances (e.g. EC2), caching, application load balancers

A service belongs to a zone, a zone belongs to an environment and so on. In this manner, each service has a co-ordinate system of `org.env.zone.service`.

## Infrastructure as Code

Infrastructure as code provides the ability to have versioned, tracked and repeatable definitions of the infrastructure used within an organisation.

Details of how infrastructure as code is used as part of this model are defined in the supplemental document [IAC.md](./IAC.md).

## Security

### Zonal Security Groups

For each zone, two security groups should exist (others can be added later by individual services):

* Zone security group - this group should have no rules (or only self-referential)
* Zone ingress security group - this group should define access to all permitted zone security groups

This structure prevents cycles and gives us two security groups with different permission profiles:

* A service associated with the zone security group will have access to all services associated with the zone ingress security group, but not grant any ingress access (i.e. will have egress permissions, but not grant ingress)
* A service associated with the zone ingress security group will permit access to all services assocaited with the zone security group, but not have access to other services (i.e. will grant ingress, but have no egress permissions)
* A service associated with both will gain both ingress and egress permissions

Concrete example:

Let's define three zones for a fairly normal, basic stack:

* `public`, containing all internet-facing infrastructure
* 'frontend', containing all the services providing the frontend
* `backend`, containing all services comprising the backend

The zones have the following access matrix defined:

| Zone       | `public`           | `frontend`         | `backend`          |
| ---------- | ------------------ | ------------------ | ------------------ |
| `public`   | :heavy_check_mark: | :heavy_check_mark: | :x:                |
| `frontend` | :x:                | :heavy_check_mark: | :heavy_check_mark: |
| `backend`  | :x:                | :x:                | :heavy_check_mark: |

We, therefore, have the six following zonal security groups:

| Security Group     | Granted Permissions                                                             |
| ------------------ | ------------------------------------------------------------------------------- |
| `public`           | Grants access to services associated with the `ingress.frontend` security group |
| `ingress.public`   | N/A                                                                             |
| `frontend`         | Grants access to services associated with the `ingress.backend` security group  |
| `ingress.frontend` | Permits access from zonal security groups (`public`)                            |
| `backend`          | N/A                                                                             |
| `ingress.backend`  | Permits access fron zonal security groups (`frontend`)                          |

Note that `ingress.public` and `backend` exist but have no real purpose. These should be maintained, however, in case of any future mutations of the zone model for this architecture.
