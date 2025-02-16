# Infrastructure as Code

This document defines how infrastructure as code is used as part of this architecture, including a number of definitions of code structure and testing.

## Table of Contents

1. [Tooling](#tooling)
2. [Code Structure](#code-structure)
    1. [Hub Repositories](#hub-repositories)
    2. [Module Repositories](#module-repositories)
    3. [Library Repositories](#library-repositories)

## Tooling

Within this account, OpenTofu is used for managing infrastructure - with a number of modules publicly available (go on, use them!).

OpenTofu is a fork of Terraform. Details can be found [here](https://opentofu.org/).

## Code Structure

Code should be structured using the following three types of module:

* Hubs (`hub`)
* Modules (`mod`)
* Libraries (`lib`)

![Types of Repository](./images/iac-types.svg)

Please note that within OpenTofu/Terraform, both Modules and Libraries (as defined in this document) can be referred to as modules - to avoid confusion, the word "module" will be capitalised if referring to the definition given in this document or in lower-case if referring to a OpenTofu/Terraform module.

Further details on each type can be found below:

### Hub Repositories

Hub repositories are where we concretely define the infrastructure we wish to see. It should exclusively use Modules (defined below), with environment-specific parameter values passed in to produce the desired outcome.

References to Modules should always use semantic versioning tags, to avoid accidental upgrades of infrastructure.

It is expected that multiple hub repositories would exist at a time, aligned with the [zone model defined in the repository](./README.md#structure). This aligns better with the differing audiences working at each tier - e.g. a development team is unlikely to have access to organisation level infrastructure definitions, but will absolutely need access to service level definitions. A security team, or IT team, may have the inverse requirements.

It is recommended to define hub repositories using multiple directories and subdirectories to striate infrastructure into smaller, logical chunks. E.g. separating infraastructure definitions by environment, by service and/or by logical layer.

### Module Repositories

Module repositories contain `resource` definitions and `module` references (to other Modules or Libraries).

They are intended to encapsulate a set of resources to provide a specific use case in a repeatable and reusable way with parameterisation and reference to libraries. Encapsulation, in this format, provides a number of benefits including avoiding repetition of similar code and enforcing security or organisational standards.

Modules should be versioned using semantic versioning to ensure that changes are not forced out before operators can prepare for them.

Example use cases for Module repositories:

* Deploying an EC2 server (with or without EBS volumes, public IP addresses, multiple IP addresses, etc)
* Creating an S3 bucket (supporting multiple configurations e.g. versioning enabled/disabled, public/private access, bucket policies, etc)

### Library Repositories

Library modules define `data` resources and `local` variables (interpolating data resources) to provide reusable outputs for use in modules.

It is expected that Library modules will be striated based on the [zone model defined in the repository](./README.md#structure). For example, it would make sense to have a library module to return data on the organisation, on the environment and on specific zones (or all zones). These libraries can then be used within module creating infrastructure within these layers.

Example use cases for Library repositories:

* Return data on an AWS Organizations structure, plus any other pertinent data at organisation level
* Return data on subnets including information such as subnets per availability zone per zone, subnet CIDR ranges, NAT server addresses, etc
* Return data on compute clusters for deployment targetting
