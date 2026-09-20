# Project Instructions

## Purpose and authority

This repository is a hands-on systems and cloud infrastructure engineering learning project. Preserve its focus on Linux systems, infrastructure troubleshooting, operations, reliability, recovery, and progressive automation across host, network, and later AWS boundaries.

The human engineer retains final architectural authority. Surface material design choices, trade-offs, and departures from approved documentation for human decision rather than silently establishing them.

## Scope discipline

- Preserve the distinction from the separate Network Change Delivery Platform (NCDP), which owns deep network automation and NetDevOps concerns. Networking here supports systems troubleshooting, segmentation, and service dependencies; do not turn this repository into another network automation platform.
- Add a technology only when a documented engineering requirement justifies it.
- Do not create empty scaffolding for hypothetical future work. Add directories when they contain an active implementation or documentation need.
- Do not hide the primary Linux systems behind containers.
- Do not introduce Kubernetes by default.
- Do not introduce AWS or LocalStack before an explicitly planned phase authorizes them.
- Treat the engineer's personal AWS account as the intended real cloud environment for a later approved phase. Company AWS may provide separate professional learning exposure, but it must not become a repository dependency.
- Keep Phase 1 exclusions in the README and architecture documentation unless the human engineer approves a phase or scope change.
- Prefer small, understandable changes over unnecessary abstraction.

## Learning and troubleshooting

- Follow the principle: understand manual system behaviour first, then automate it after it is understood.
- Do not automatically solve deliberately injected, learning-critical troubleshooting exercises. Preserve the exercise and assist with scaffolding or evidence collection only as requested; diagnose or repair it when the human engineer explicitly asks after investigating.
- Prefer root-cause analysis over symptom suppression.
- Never disable a security mechanism such as SELinux merely to make a problem disappear. Establish the cause, preserve evidence, and implement an understood, documented correction.
- Use the troubleshooting lifecycle: **observe → detect → isolate → diagnose → reason → repair → validate → prevent/automate → document**.

## Tool ownership

- Terraform owns infrastructure resource lifecycle: whether a resource exists. Do not use Terraform as a Linux configuration-management system.
- Ansible owns machine and service configuration: packages, accounts, SSH, files, systemd units, services, and host firewall configuration.
- Bash owns small, transparent Linux operational actions and diagnostic glue. Move substantial state, logic, testing, or error handling to Python.
- Python owns structured diagnostics, health checking, validation, discovery, operational CLI tooling, cloud API operations in an approved cloud phase, and higher-level automation.
- Document any justified exception to these boundaries before implementing it.

## Safety, quality, and documentation

- Never commit credentials, secrets, tokens, private keys, local environment files containing sensitive values, or company credentials, data, architecture, configuration, or proprietary information.
- Never execute destructive cloud or infrastructure operations without explicit human instruction. Resolve and state the exact target before any authorized destructive action.
- Run validation and tests appropriate to the risk and scope of every future implementation change.
- Keep documentation aligned with actual implementation.
- Clearly label current, planned, and future capabilities. Never represent a planned capability as implemented.
- Preserve useful security controls and least-privilege boundaries.
- Keep changes reviewable and avoid unrelated refactoring.
