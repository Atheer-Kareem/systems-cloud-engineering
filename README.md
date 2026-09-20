# Systems & Cloud Engineering Lab

A reproducible Linux and cloud infrastructure environment for engineering, operating, deliberately breaking, diagnosing, recovering, and progressively automating real services across host, network, and, in later phases, AWS boundaries.

## Why this project exists

This is a hands-on environment for developing systems engineering judgement through direct work with Linux services, infrastructure troubleshooting, automation, observability, security, reliability, and recovery. The work follows a deliberate progression: understand how a system behaves manually, then automate the parts that are understood and benefit from automation.

The separate Network Change Delivery Platform (NCDP) project covers deep network automation and NetDevOps concerns. This lab uses networking to provide segmentation, dependencies, and realistic failure modes for systems and cloud engineering; it is not another network automation platform.

## Engineering lifecycle

The operating lifecycle is:

**provision → configure → deploy → observe → operate → fail → diagnose → repair → validate → improve/automate**

Troubleshooting within that lifecycle follows:

**observe → detect → isolate → diagnose → reason → repair → validate → prevent/automate → document**

## Current scope: Phase 1

The project is currently at the architecture and repository-bootstrap stage. The approved Phase 1 design targets Cisco Modeling Labs (CML) 2.10 and consists of two lightweight Cisco IOL routers, unmanaged Layer 2 switches where required, and five RHEL 9 hosts. All five RHEL systems run as nodes inside CML during Phase 1:

```text
                         Transit 10.50.0.0/30
                    EDGE-R1 ---------------- CORE-R1
                       |                         |
           Frontend 10.50.10.0/24      +--------+----------------+
                       |                |        |                |
                    proxy01       Application   Data       Operations/Services
                                  10.50.20.0/24 10.50.30.0/24 10.50.40.0/24
                                        |        |          |       |
                                      app01     db01      ops01   dns01
```

The planned primary service path is `client → proxy01 → app01 → db01`. `dns01` provides the internal `lab.test` namespace as an infrastructure dependency. Service configuration should use internal DNS names where appropriate instead of hard-coded service IP addresses so that the primary service environment meaningfully depends on DNS. Operational administration is planned to originate primarily from `ops01`. Static routing is sufficient for the initial phase.

The detailed approved design and its explicitly undecided areas are recorded in [Phase 1 logical architecture](docs/architecture/phase-1-logical-architecture.md).

## Tool responsibilities

- **Terraform** owns infrastructure lifecycle: whether infrastructure resources exist. It may later manage suitable CML topology resources and, in later phases, AWS resources. It does not configure Linux as a configuration-management system.
- **Ansible** owns machine and service configuration, including packages, accounts, files, systemd units, services, and host firewall configuration.
- **Bash** owns small, transparent operational actions and diagnostic glue.
- **Python** owns structured diagnostics, validation, health checking, inventory, operational tooling, and automation that needs meaningful state, logic, testing, or error handling.

These tools are planned responsibilities. No infrastructure or service implementation has been added yet.

## Learning philosophy

Manual system behaviour should be understood before it is automated. Automation should remove repetition after the underlying behaviour is clear, not conceal it. Deliberately injected troubleshooting exercises remain for the human engineer to investigate unless help solving them is explicitly requested.

## Phase 1 exclusions

Phase 1 does not include AWS infrastructure, LocalStack, Kubernetes, ECS or EKS, containers for the principal Linux services, high availability, redundant routers, dynamic routing, complex CI/CD, automatic remediation, multi-region architecture, service mesh, Kafka, unnecessary microservices, or elaborate secrets infrastructure.

Directories and code for future technologies will be added only when an active implementation requires them.

## Status

**Current:** logical architecture and repository documentation baseline.

**Planned for Phase 1:** provision the approved CML topology, configure the RHEL hosts and services, establish observability, exercise the initial failure scenarios, and validate recovery.

**Future:** AWS infrastructure and operations, cloud networking, and related automation in a later explicitly planned phase. The intended real AWS environment is the engineer's personal AWS account. Company AWS may provide separate professional learning exposure, but this repository must not depend on it or contain company credentials, data, architecture, configuration, or proprietary information.
