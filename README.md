# terraform-aws-elasticache-redis

Terraform module to provision an [`ElastiCache`](https://aws.amazon.com/elasticache/) Redis Cluster


## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/ceibo-it/terraform-aws-elasticache-redis/releases).



For a complete example, see [examples/complete](examples/complete).

```hcl
  provider "aws" {
    region = var.region
  }

  module "vpc" {
    source     = "git::https://github.com/ceibo-it/terraform-aws-vpc.git?ref=tags/0.0.1"
    namespace  = var.namespace
    stage      = var.stage
    name       = var.name
    cidr_block = "172.16.0.0/16"
  }

  module "subnets" {
    source               = "git::https://github.com/ceibo-it/terraform-aws-dynamic-subnets.git?ref=tags/0.0.1"
    availability_zones   = var.availability_zones
    namespace            = var.namespace
    stage                = var.stage
    name                 = var.name
    vpc_id               = module.vpc.vpc_id
    igw_id               = module.vpc.igw_id
    cidr_block           = module.vpc.vpc_cidr_block
    nat_gateway_enabled  = true
    nat_instance_enabled = false
  }

  module "redis" {
    source                     = "git::https://github.com/ceibo-iy/terraform-aws-elasticache-redis.git?ref=master"
    availability_zones         = var.availability_zones
    namespace                  = var.namespace
    stage                      = var.stage
    name                       = var.name
    zone_id                    = var.zone_id
    vpc_id                     = module.vpc.vpc_id
    allowed_security_groups    = [module.vpc.vpc_default_security_group_id]
    subnets                    = module.subnets.private_subnet_ids
    cluster_size               = var.cluster_size
    instance_type              = var.instance_type
    apply_immediately          = true
    automatic_failover         = false
    engine_version             = var.engine_version
    family                     = var.family
    at_rest_encryption_enabled = var.at_rest_encryption_enabled
    transit_encryption_enabled = var.transit_encryption_enabled

    parameter = [
      {
        name  = "notify-keyspace-events"
        value = "lK"
      }
    ]
  }
```




## Examples

Review the [complete example](examples/simple) to see how to use this module.



## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| alarm_actions | Alarm action list | list(string) | `<list>` | no |
| alarm_cpu_threshold_percent | CPU threshold alarm level | number | `75` | no |
| alarm_memory_threshold_bytes | Ram threshold alarm level | number | `10000000` | no |
| allowed_cidr_blocks | List of CIDR blocks that are allowed ingress to the cluster's Security Group created in the module | list(string) | `<list>` | no |
| allowed_security_groups | List of Security Group IDs that are allowed ingress to the cluster's Security Group created in the module | list(string) | `<list>` | no |
| apply_immediately | Apply changes immediately | bool | `true` | no |
| at_rest_encryption_enabled | Enable encryption at rest | bool | `false` | no |
| attributes | Additional attributes (_e.g._ "1") | list(string) | `<list>` | no |
| auth_token | Auth token for password protecting redis, `transit_encryption_enabled` must be set to `true`. Password must be longer than 16 chars | string | `null` | no |
| automatic_failover_enabled | Automatic failover (Not available for T1/T2 instances) | bool | `false` | no |
| availability_zones | Availability zone IDs | list(string) | `<list>` | no |
| cluster_mode_enabled | Flag to enable/disable creation of a native redis cluster. `automatic_failover_enabled` must be set to `true`. Only 1 `cluster_mode` block is allowed | bool | `false` | no |
| cluster_mode_num_node_groups | Number of node groups (shards) for this Redis replication group. Changing this number will trigger an online resizing operation before other settings modifications | number | `0` | no |
| cluster_mode_replicas_per_node_group | Number of replica nodes in each node group. Valid values are 0 to 5. Changing this number will force a new resource | number | `0` | no |
| cluster_size | Number of nodes in cluster. *Ignored when `cluster_mode_enabled` == `true`* | number | `1` | no |
| delimiter | Delimiter between `name`, `namespace`, `stage` and `attributes` | string | `-` | no |
| dns_subdomain | The subdomain to use for the CNAME record. If not provided then the CNAME record will use var.name. | string | `` | no |
| elasticache_subnet_group_name | Subnet group name for the ElastiCache instance | string | `` | no |
| enabled | Set to false to prevent the module from creating any resources | bool | `true` | no |
| engine_version | Redis engine version | string | `4.0.10` | no |
| existing_security_groups | List of existing Security Group IDs to place the cluster into. Set `use_existing_security_groups` to `true` to enable using `existing_security_groups` as Security Groups for the cluster | list(string) | `<list>` | no |
| family | Redis family | string | `redis4.0` | no |
| instance_type | Elastic cache instance type | string | `cache.t2.micro` | no |
| maintenance_window | Maintenance window | string | `wed:03:00-wed:04:00` | no |
| name | Name of the application | string | - | yes |
| namespace | Namespace (e.g. `eg` or `cp`) | string | `` | no |
| notification_topic_arn | Notification topic arn | string | `` | no |
| ok_actions | The list of actions to execute when this alarm transitions into an OK state from any other state. Each action is specified as an Amazon Resource Number (ARN) | list(string) | `<list>` | no |
| parameter | A list of Redis parameters to apply. Note that parameters may differ from one Redis family to another | object | `<list>` | no |
| port | Redis port | number | `6379` | no |
| replication_group_id | Replication group ID with the following constraints:  A name must contain from 1 to 20 alphanumeric characters or hyphens.   The first character must be a letter.   A name cannot end with a hyphen or contain two consecutive hyphens. | string | `` | no |
| snapshot_retention_limit | The number of days for which ElastiCache will retain automatic cache cluster snapshots before deleting them. | number | `0` | no |
| snapshot_window | The daily time range (in UTC) during which ElastiCache will begin taking a daily snapshot of your cache cluster. | string | `06:30-07:30` | no |
| stage | Stage (e.g. `prod`, `dev`, `staging`) | string | `` | no |
| subnets | Subnet IDs | list(string) | `<list>` | no |
| tags | Additional tags (_e.g._ map("BusinessUnit","ABC") | map(string) | `<map>` | no |
| transit_encryption_enabled | Enable TLS | bool | `true` | no |
| use_existing_security_groups | Flag to enable/disable creation of Security Group in the module. Set to `true` to disable Security Group creation and provide a list of existing security Group IDs in `existing_security_groups` to place the cluster into | bool | `false` | no |
| vpc_id | VPC ID | string | - | yes |
| zone_id | Route53 DNS Zone ID | string | `` | no |


## Outputs

| Name              | Description            |
| ----------------- | ---------------------- |
| endpoint          | Redis primary endpoint |
| host              | Redis hostname         |
| id                | Redis cluster ID       |
| port              | Redis port             |
| security_group_id | Security group ID      |


## Copyright

Copyright © 2020 [Ceibo IT](https://ceibo.it/copyright)



## License 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.


## Trademarks

All other trademarks referenced herein are the property of their respective owners.


## NOTICE

terraform-aws-elasticache-redis
Copyright 2020 Ceibo


This product includes software developed by
Cloud Posse, LLC (c) (https://cloudposse.com/)
Licensed under Apache License, Version 2.0
Copyright © 2017-2019 Cloud Posse, LLC