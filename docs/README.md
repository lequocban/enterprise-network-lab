# Technical Documentation Index

Project: [Enterprise Network Design & Implementation](../README.md)  
Review date: **2026-10-09**

| Document | State | Description |
|---|---|---|
| [Network architecture](network-architecture.md) | HQ as-built + enterprise target draft | Device roles, 10 physical links, risk/HA boundaries |
| [IP Addressing Plan](ip-addressing-plan.md) | HQ defined; WAN candidate | VLAN plan, HSRP gateway addresses, Branch/DMZ/Transit/Loopbacks |
| [Sprint 0 Validation](test-results/sprint-00-validation.md) | Evidence captured + evidence gaps | Environment, ping, VLAN10, HSRP |
| [Sprint 0 Report](sprint-reports/sprint-00-report.md) | Prepared for sign-off | Sprint review, outcomes, risks and Sprint 1 entry |
| [Project Roadmap](enterprise-network-design-implementation-roadmap.md) | Source | User-provided roadmap with Sprint 0–11 |
| [HQ topology screenshot](../diagrams/hq-campus-topology.svg) | As-built HQ | Interface labels from current EVE-NG lab |

## Document statuses

- **Verified**: supported by command output / screenshot in the project conversation.
- **Reported**: user confirmed behavior but full command evidence was not captured in the repository.
- **Planned / Draft**: intended for subsequent Sprints, not yet deployed or verified.

## Sprint 0 closure criteria (roadmap)

- [x] EVE-NG environment operational and Cisco IOSv/IOL tests performed.
- [x] HQ campus topology diagram and interface mapping captured.
- [x] HQ subnet/HSRP plan and draft extended-enterprise IP plan documented.
- [x] GitHub public repository exists, with README and `.gitignore` visible on `master`.
- [ ] Full enterprise logical/physical topology exports (HQ/Edge/DMZ/Branches) finalized.
- [ ] ASAv and Alpine boot/service evidence added to repository.
- [ ] Documentation PR reviewed and merged into `master`.
- [ ] WAN transit subnet allocation reviewed against final firewall/edge interface layout.

> Technical NET-002 HQ prototype was reported complete by the lab owner. Formal **Sprint 0 DONE** requires the outstanding Definition of Done artifacts above.