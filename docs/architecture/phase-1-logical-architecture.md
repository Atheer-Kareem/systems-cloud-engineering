# Phase 1 Logical Architecture

## Status and purpose

This document records the currently approved logical design for Phase 1 of the Systems & Cloud Engineering Lab. The project is at the architecture and repository-bootstrap stage: the topology, hosts, services, and automation described here are planned and have not yet been implemented.

## Architecture goals

Phase 1 will provide a reproducible environment in which to:

- build and operate real Linux service dependencies across segmented networks;
- practise diagnosis and recovery across host, service, DNS, storage, permissions, and network boundaries;
- develop observable behaviour before introducing operational automation;
- apply clear ownership boundaries among infrastructure lifecycle, configuration management, shell operations, and higher-level tooling; and
- keep networking sufficient for meaningful systems troubleshooting without making it the dominant engineering subject.

The operating lifecycle is:

**provision → configure → deploy → observe → operate → fail → diagnose → repair → validate → improve/automate**

The troubleshooting lifecycle is:

**observe → detect → isolate → diagnose → reason → repair → validate → prevent/automate → document**

## Logical topology

The runtime environment is planned primarily for Cisco Modeling Labs 2.10. The five RHEL 9 systems run as nodes inside CML during Phase 1. External placement is not a Phase 1 option and may be reconsidered in a future phase only if an actual requirement justifies it.

```text
                         10.50.0.0/30 transit
                  .1                               .2
              +---------+                     +---------+
              | EDGE-R1 |---------------------| CORE-R1 |
              +---------+                     +---------+
                   |                                |
       10.50.10.0/24 frontend        +--------------+--------------+
                   |                 |              |              |
             +---------+     10.50.20.0/24  10.50.30.0/24  10.50.40.0/24
             | proxy01 |       application        data      operations/services
             +---------+            |              |          |         |
                                +-------+        +------+   +-------+ +-------+
                                | app01 |        | db01 |   | ops01 | | dns01 |
                                +-------+        +------+   +-------+ +-------+
```

EDGE-R1 and CORE-R1 are planned as lightweight Cisco IOL nodes. Unmanaged CML switches will provide Layer 2 connectivity where required. Static routing is sufficient initially. Exact interface names, switch placement, client location, and any external connectivity are undecided.

## Zones, subnets, and planned addressing

| Zone | Subnet | Gateway or router address | Planned hosts | Purpose |
|---|---|---|---|---|
| Transit | `10.50.0.0/30` | EDGE-R1 `10.50.0.1`; CORE-R1 `10.50.0.2` | Routers only | Point-to-point routing between edge and core |
| Frontend | `10.50.10.0/24` | EDGE-R1 `10.50.10.1` | proxy01 `10.50.10.10` | Client-facing proxy tier |
| Application | `10.50.20.0/24` | CORE-R1 `10.50.20.1` | app01 `10.50.20.10` | Application tier |
| Data | `10.50.30.0/24` | CORE-R1 `10.50.30.1` | db01 `10.50.30.10` | Database tier |
| Operations/Services | `10.50.40.0/24` | CORE-R1 `10.50.40.1` | ops01 `10.50.40.10`; dns01 `10.50.40.53` | Administration and shared infrastructure services |

The five server nodes—proxy01, app01, db01, ops01, and dns01—are planned as RHEL 9 systems. Hostname-to-address records within `lab.test`, default routes, and detailed interface assignments remain to be defined during implementation.

## Host responsibilities

| Node | Planned responsibility |
|---|---|
| EDGE-R1 | Route between the transit and frontend networks |
| CORE-R1 | Route between the transit, application, data, and operations/services networks |
| proxy01 | Receive client requests and proxy them to app01 |
| app01 | Run the application service and access db01 |
| db01 | Provide the application's database service |
| dns01 | Provide internal DNS for the `lab.test` namespace |
| ops01 | Serve as the primary origin for operational administration and, as the design matures, operational tooling |

The proxy, application, database, DNS, logging, metrics, and monitoring products and their exact configurations are undecided. No service implementation is implied by these responsibility assignments.

## Service, management, and dependency paths

The primary planned service path is:

```text
client → proxy01 → app01 → db01
```

The client's location and the exact application protocol and ports are undecided.

Operational administration should originate primarily from `ops01`. The authentication model, administrative protocols, access-control rules, and any bootstrap access path are undecided.

`dns01` is a shared infrastructure dependency, not an optional convenience. Service configuration should use names in the internal `lab.test` namespace where appropriate instead of hard-coded service IP addresses, making the primary service environment meaningfully dependent on DNS. The complete record set, forwarding, recursion, implementation product, DNS security policy, redundancy, and upstream resolver behaviour are undecided.

## Trust and connectivity boundaries

Each subnet is a distinct logical zone. Connectivity should be limited to the paths required for service delivery, administration, name resolution, and observability. The intended direction is:

- client traffic reaches proxy01 in the frontend zone;
- proxy01 reaches the required application endpoint on app01;
- app01 reaches the required database endpoint on db01;
- managed systems reach dns01 for approved DNS functions;
- operational administration originates primarily from ops01; and
- observability traffic crosses zones only where required by the selected design.

Exact ports, host firewall rules, router filtering, trust policy, SELinux policy requirements, and Internet access are undecided. They must be documented when service and security requirements are selected. Security controls must support diagnosis without being disabled merely to suppress symptoms.

## Observability direction

Phase 1 should make service health and failures visible across the primary path and its DNS dependency. The eventual design should provide enough evidence to distinguish application, service-manager, name-resolution, filesystem, permissions, and connectivity failures. Specific metrics, logs, probes, collection architecture, retention, dashboards, and products are undecided. Automatic remediation is excluded from Phase 1.

## Tool ownership

| Tool | Owns | Does not own |
|---|---|---|
| Terraform | Infrastructure lifecycle: whether an infrastructure resource exists | Linux machine and service configuration |
| Ansible | Desired configuration of machines and services | Infrastructure resource lifecycle |
| Bash | Small, transparent Linux operational actions and diagnostic glue | Large application-like automation with substantial state or logic |
| Python | Structured diagnostics, health checks, validation, inventory/discovery, operational CLI tooling, and logic requiring robust testing or error handling | Configuration tasks better expressed declaratively in Ansible |

Terraform may manage the CML topology where appropriate. Whether and how CML resources will be represented is undecided. AWS resource ownership belongs to a later phase; no AWS implementation is part of Phase 1. The intended real AWS environment for this public, personal project is the engineer's personal AWS account. Company AWS may provide separate professional learning exposure, but it must not become a dependency of this repository, and no company credentials, data, architecture, configuration, or proprietary information belong here.

## Deliberate exclusions

Phase 1 excludes:

- AWS infrastructure and LocalStack;
- Kubernetes, ECS, and EKS;
- containers for the principal Linux services;
- high availability and redundant routers;
- dynamic routing;
- complex CI/CD;
- automatic remediation;
- multi-region architecture;
- service mesh and Kafka;
- unnecessary microservices; and
- elaborate secrets infrastructure.

Implementation directories for these or other technologies will not be added until justified by active work.

## First planned failure scenarios

The initial exercises will cover:

1. broken systemd or application configuration;
2. DNS resolution failure;
3. filesystem pressure on db01;
4. a blocked app01-to-db01 path; and
5. incorrect service permissions or ownership.

Each exercise should preserve the opportunity for manual investigation. The human engineer should use the troubleshooting lifecycle to gather evidence and explain the root cause before repair and later automation. Exact injection mechanisms, expected signals, validation procedures, and prevention measures are undecided.

## Phase 1 exit criteria

Phase 1 is complete when:

- the approved topology and addressing are reproducibly established in CML;
- static routing and required, documented cross-zone connectivity support the service, management, DNS, and observability paths;
- the five RHEL 9 hosts have documented roles and reproducible configuration;
- the `client → proxy01 → app01 → db01` service path operates and can be validated;
- internal `lab.test` DNS supports the environment's documented name-resolution requirements;
- administration can originate primarily from ops01 through documented controls;
- selected observability provides evidence useful for isolating the planned failure classes;
- all five initial failure scenarios have been manually investigated, repaired, validated, and documented at least once;
- appropriate repetitive work is automated only after its manual behaviour is understood; and
- documentation reflects the implemented system, including any approved departures from this logical design.

The concrete service stack, observability stack, security policy, validation tooling, and provisioning approach must be decided and documented before their corresponding exit criteria can be evaluated.
