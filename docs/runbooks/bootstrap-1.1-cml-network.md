# Bootstrap 1.1: Manual CML Network Skeleton

## Objective

Manually build and validate the approved Phase 1 network skeleton in Cisco Modeling Labs (CML) 2.10. This bootstrap is an interface-addressing and static-routing exercise: reason through each interface, connected network, next hop, and verification result rather than pasting complete router configurations.

## What will exist at the end

- two running Cisco IOL routers named `EDGE-R1` and `CORE-R1`;
- a direct transit link between the routers;
- four separate unmanaged Layer 2 segments for frontend, application, data, and operations/services;
- approved router interface addressing on all five networks;
- static routes between the frontend network and the three networks behind `CORE-R1`; and
- verified router-to-router reachability, with no Linux hosts or automation added.

Dynamic routing, ACL policy, NAT, external connectivity, and host deployment are outside this bootstrap.

## CML nodes to add

Create a new CML 2.10 lab and add only:

| Quantity | Node type | Names |
|---|---|---|
| 2 | Cisco IOL router | `EDGE-R1`, `CORE-R1` |
| 4 | Unmanaged switch | `SW-FRONTEND`, `SW-APPLICATION`, `SW-DATA`, `SW-OPS-SERVICES` |

Do not add RHEL nodes, external connectors, NAT nodes, automation hosts, or additional network devices.

## Logical switch and segment placement

Use a direct point-to-point link for the transit network. Attach one unmanaged switch to each router-facing multi-access segment:

```text
SW-FRONTEND -- EDGE-R1 -- transit -- CORE-R1 -- SW-APPLICATION
                                         |  \
                                         |   `-- SW-DATA
                                         `------ SW-OPS-SERVICES
```

Keep the four switches independent. VLANs and trunks are not required because each switch represents one logical Layer 2 segment.

| Segment | Network | Attachment |
|---|---|---|
| Transit | `10.50.0.0/30` | Direct link between both routers |
| Frontend | `10.50.10.0/24` | `EDGE-R1` to `SW-FRONTEND` |
| Application | `10.50.20.0/24` | `CORE-R1` to `SW-APPLICATION` |
| Data | `10.50.30.0/24` | `CORE-R1` to `SW-DATA` |
| Operations/Services | `10.50.40.0/24` | `CORE-R1` to `SW-OPS-SERVICES` |

## Suggested interface-to-segment mapping

Interface names can vary with the selected IOL image. Before configuring anything, inspect the available interfaces and record the actual names. The following mapping is a suggestion, not a substitute for checking the nodes:

| Router | Suggested interface | Segment | Address |
|---|---|---|---|
| `EDGE-R1` | first Ethernet interface | Transit | `10.50.0.1/30` |
| `EDGE-R1` | second Ethernet interface | Frontend | `10.50.10.1/24` |
| `CORE-R1` | first Ethernet interface | Transit | `10.50.0.2/30` |
| `CORE-R1` | second Ethernet interface | Application | `10.50.20.1/24` |
| `CORE-R1` | third Ethernet interface | Data | `10.50.30.1/24` |
| `CORE-R1` | fourth Ethernet interface | Operations/Services | `10.50.40.1/24` |

Use CML link labels or interface descriptions to make each mapping visible. If the selected IOL node lacks the required interfaces, stop and correct the node definition or image choice rather than changing the approved topology.

## Manual router configuration sequence

1. Add, name, and connect the six CML nodes while they are stopped.
2. Check every link endpoint against the mapping table before starting the lab.
3. Start all six nodes and use `show ip interface brief` on both routers to identify the actual interface names and states.
4. Set each router hostname manually.
5. On one interface at a time, enter interface configuration mode, add a meaningful description, apply the approved IP address and mask, and enable the interface.
6. Return to operational mode and confirm that interface before configuring the next one.
7. Verify transit reachability before adding static routes.
8. Derive and enter the required static routes from the routing table below.
9. Verify the complete router-only forwarding paths in both directions.
10. Save each working configuration with `copy running-config startup-config`.

For an individual interface, the command pattern is:

```text
interface <actual-interface-name>
 description <connected-segment>
 ip address <approved-address> <dotted-decimal-mask>
 no shutdown
```

Select the correct interface, address, and mask from the topology. Do not assemble or paste a complete router configuration.

## Static routes required

Connected routes should appear automatically after interface configuration. Add only these remote-network routes:

| Router | Destination | Next hop |
|---|---|---|
| `EDGE-R1` | `10.50.20.0/24` | `10.50.0.2` |
| `EDGE-R1` | `10.50.30.0/24` | `10.50.0.2` |
| `EDGE-R1` | `10.50.40.0/24` | `10.50.0.2` |
| `CORE-R1` | `10.50.10.0/24` | `10.50.0.1` |

Use the IOS static-route form `ip route <destination> <mask> <next-hop>`. Translate each prefix length into the correct dotted-decimal mask and enter each route manually. Do not configure a dynamic routing protocol or a default route during this bootstrap.

## Verification commands

Use these commands as evidence, not merely as a checklist:

- `show ip interface brief` — confirm the intended interfaces are up/up with the approved addresses.
- `show interfaces description` — confirm physical-to-logical mapping and interface state.
- `show ip route connected` — identify the networks each router owns directly.
- `show ip route static` — confirm every required remote network has the intended next hop.
- `show ip route <network-or-address>` — inspect the forwarding decision for a specific destination.
- `ping 10.50.0.2` from `EDGE-R1` and `ping 10.50.0.1` from `CORE-R1` — validate the transit link.
- Ping each remote router gateway address after static routes are present. Where useful, use an extended ping and select a source interface from the opposite side of the topology to prove the return path.
- `show running-config | section interface` and `show running-config | include ^ip route` — compare the implemented state with the approved plan.

## Expected successful results

- All six routed interfaces show the approved address and an up/up state.
- Each router installs its directly attached networks with connected (`C`) and local (`L`) route entries.
- `EDGE-R1` installs three static (`S`) routes through `10.50.0.2`.
- `CORE-R1` installs one static (`S`) route through `10.50.0.1`.
- Each router can ping the other transit address.
- `EDGE-R1` can reach the three `CORE-R1` LAN gateway addresses.
- `CORE-R1` can reach the `EDGE-R1` frontend gateway address.
- A sourced or extended ping across the transit succeeds in both directions, demonstrating a valid forward and return route.

Empty unmanaged switches do not provide an endpoint to ping. Their segment connectivity will be exercised only after a later, explicitly approved host bootstrap.

## Troubleshooting checklist

If a check fails, preserve the evidence and work from the nearest dependency outward:

1. Confirm that both CML nodes and the relevant links are running.
2. Compare the CML link endpoints with the recorded interface mapping.
3. Check interface administrative and line-protocol state with `show ip interface brief`.
4. Check the configured address and mask on both ends; pay particular attention to the `/30` transit mask.
5. Verify direct transit pings before investigating static routing.
6. Confirm that the destination route is installed and points to the adjacent transit address.
7. Confirm that the remote router has a return route to the selected ping source.
8. Use `show ip route <destination>` and an extended ping to isolate the failing direction.
9. Correct the understood root cause, repeat the failed check, and record the result. Do not add dynamic routing, NAT, or permissive ACLs to conceal the problem.

## Stop condition

Stop Bootstrap 1.1 when the expected router-only results have been demonstrated and both configurations have been saved. Record the chosen CML interface mapping and any explained deviation from the suggested mapping.

Do not add `proxy01`, `app01`, `db01`, `dns01`, `ops01`, or any other Linux node. Do not instantiate their reserved addresses. Host placement and service configuration begin only in a later, explicitly approved bootstrap step.
