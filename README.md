# Wazuh SIEM/XDR Home Lab — Documentation

**Author:** Skand
**Date:** July 24, 2026
**Lab focus:** Wazuh manager deployment + endpoint agent onboarding + File Integrity Monitoring (FIM)

---

## 1. Objective

Deploy a self-hosted SIEM/XDR stack using Wazuh, onboard a Windows endpoint as a monitored agent, and validate detection coverage by configuring and testing **File Integrity Monitoring (FIM)**. This lab demonstrates practical experience with agent-manager architecture, endpoint telemetry, and security event triage in a Wazuh environment.

## 2. Lab Architecture

| Component | Details |
|---|---|
| Wazuh Manager | Ubuntu VM, `stb-VMware-Virtual-Platform`, IP `192.168.136.133` |
| Wazuh Version | v4.14.6 |
| Endpoint (Agent) | Windows 11 Home Single Language (build 10.0.26200.8875), host `MSI-STB` |
| Agent ID | 001 (`windows-agent`), IP `192.168.136.1` |
| Endpoint Hardware | 13th Gen Intel Core i7-13620H, 16 cores, 15.7 GB RAM |
| Deployment model | Manager on Ubuntu, agent installed on Windows host, agent registered to manager over the internal VMware network |

The manager and agent communicate over a host-only/NAT VMware network, with the agent configured to report to the manager IP and authenticate using a manager-issued key.

## 3. Setup Steps

1. **Deployed the Wazuh manager** on an Ubuntu VM and confirmed the stack initialized correctly — API connection, API version, and alerts/monitoring/statistics index patterns all passed their startup checks.
2. **Installed the Wazuh agent** on a Windows 11 endpoint and configured it to point at the manager (`Manager IP: 192.168.136.133`), applying the authentication key issued by the manager.
3. **Registered and verified the agent** — confirmed status `active`, correct OS fingerprinting (Windows 11 Home Single Language), and hardware inventory (CPU, cores, memory, serial number) reporting correctly into the manager's system inventory.
4. **Confirmed manager visibility** — endpoint summary showed 1 active agent, 0 disconnected, with alert counts by severity populating on the Overview dashboard.

### Startup verification

![Wazuh startup checks](screenshots/04-wazuh-startup-check.png)

### Agent-side connection status

![Wazuh Agent connected to manager](screenshots/01-agent-connection.png)

### Manager-side agent registration

![Agent summary dashboard](screenshots/02-agent-summary-dashboard.png)

## 4. File Integrity Monitoring (FIM) — Configuration & Test

To validate that the manager was receiving and correctly parsing real-time endpoint telemetry, I ran a controlled FIM test:

1. Created a new file (`new text document.txt`) inside a monitored path: `C:\Users\skand\Downloads\wazuh-test\`.
2. Created a second file (`test1.txt`) in the same monitored path.
3. Deleted both files.

Wazuh's `syscheck` module picked up all four filesystem events in near real time and generated corresponding alerts:

| Rule ID | Description | Rule Level |
|---|---|---|
| 554 | File added to the system | 5 |
| 553 | File deleted | 7 |

### FIM events as captured by the manager

![FIM recent events](screenshots/03-fim-recent-events.png)

This confirms end-to-end detection: filesystem change on the endpoint → agent telemetry → manager rule matching → alert surfaced in the dashboard, correctly attributed to the acting user (`skand`) and agent (`windows-agent`, ID 001).

## 5. Dashboard & Detection Coverage

The Overview dashboard confirmed the stack's endpoint security and threat intelligence modules were live and receiving data:

![Overview dashboard](screenshots/05-overview-dashboard.png)

- **Agents summary:** 1 active, 0 disconnected
- **Last 24h alerts:** 0 critical, 0 high, 483 medium, 326 low
- **MITRE ATT&CK mapping:** top tactics observed were *Defense Evasion* and *Impact*, auto-mapped from raw alerts
- **Compliance mapping:** alerts auto-tagged against PCI DSS controls (2.2, 10.6.1, 10.2.6, 11.5, 11.4)

### Endpoint detail view

![Endpoint detail view](screenshots/06-endpoint-detail-view.png)

The endpoint detail page also surfaced a **Security Configuration Assessment (SCA)** scan against the CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0:

| Policy | Passed | Failed | Not applicable | Score |
|---|---|---|---|---|
| CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0 | 124 | 349 | 9 | 26% |

This low compliance score is expected on a default/unhardened Windows install and is a useful artifact in itself — it demonstrates the lab surfaces real, actionable hardening gaps rather than a synthetic/scripted "clean" result.

## 6. Formal Report Output

Wazuh's built-in reporting module was used to generate a scoped PDF report for the FIM module (`manager.name: stb-VMware-Virtual-Platform AND rule.groups: syscheck`, 24-hour window). It independently corroborates the dashboard findings: top FIM rules (554/553), the single monitored agent, a 50/50 split between added/deleted actions, and the four-event alert summary tied to `c:\users\skand\downloads\wazuh-test\`.

📄 See attached: `wazuh-fim-report.pdf`

## 7. Summary of Skills Demonstrated

- Standing up a Wazuh manager on Linux and validating service health via the built-in startup checks
- Agent deployment and manager-agent authentication/registration on a Windows endpoint
- Configuring and functionally testing File Integrity Monitoring (FIM) with controlled file-system changes
- Reading and interpreting MITRE ATT&CK tactic mapping and PCI DSS compliance auto-tagging from raw alerts
- Interpreting Security Configuration Assessment (SCA) results against a CIS benchmark
- Generating and validating scoped, time-boxed PDF reports from the SIEM for a specific rule group

## 8. Possible Next Steps

- Harden the Windows endpoint against the failed CIS benchmark checks and re-scan to show before/after improvement
- Expand FIM scope to additional directories and enable registry monitoring (Windows-specific)
- Add a second agent (e.g., a Linux VM) to demonstrate multi-platform coverage
- Build a custom detection rule and trigger it to show custom rule authoring, not just default rule sets

---
*All screenshots and the PDF report referenced above were generated directly from the running lab environment.*
