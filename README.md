# Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab

This repository contains the implementation artifact for my MSICT cybersecurity capstone project. I compared a traditional perimeter-based security architecture with a Zero Trust-oriented architecture in a controlled four-VM VirtualBox environment.

Comparative testing is complete. Source-specific host firewall rules blocked three unnecessary service paths while preserving three intended paths across six predefined scenarios.

## Research Question

In a controlled virtual enterprise network environment, how does a Zero Trust-oriented security architecture compare with a traditional perimeter-based security architecture in preventing unauthorized access and limiting lateral movement under defined cybersecurity attack scenarios?

The experiment measured service reachability, authentication outcomes, and preservation of authorized access. These measures address unnecessary service exposure and potential lateral-access paths; they do not establish successful compromise or its prevention.

## Lab Environment

The lab consists of four virtual machines connected to an isolated VirtualBox Internal Network named `enterprise-lab`.

| Virtual Machine | Role | IP Address |
|---|---|---|
| Kali-Attacker | Security testing system and permitted shared-service client | 192.168.50.40 |
| Ubuntu-Workstation | Enterprise workstation / assumed compromised endpoint | 192.168.50.30 |
| Protected-Server-A | Protected server | 192.168.50.10 |
| Protected-Server-B | Protected server and shared HTTP status service | 192.168.50.20 |

All four Adapter 1 connections use `enterprise-lab` on `192.168.50.0/24`. Adapters 2 through 4 are disabled. No gateway or DNS server is configured.

Both protected servers provide OpenSSH on TCP 22 and nginx HTTP on TCP 80. Server B hosts `/status.txt`, which returns:

```text
Server B Shared Status Service - OK
```

Client addressing must be checked before testing. Temporary address loss occurred during testing, and persistence after reboot was not confirmed for both clients.

## Architecture Conditions

### Traditional Perimeter-Based Condition (P)

The traditional condition permits internal subnet access to the declared SSH and HTTP services on both protected servers. SSH authentication and normal operating-system permissions remain enabled.

This condition represents comparatively permissive internal service access. It does not represent every perimeter-based enterprise architecture or evaluate an Internet-facing firewall.

### Zero Trust-Oriented Condition (Z)

The Zero Trust-oriented condition uses the same topology, addresses, and services but applies source-specific UFW permissions.

Both conditions use default-deny incoming and default-allow outgoing policies.

| Destination | Service | Traditional P | Zero Trust-Oriented Z |
|---|---|---|---|
| Server A `.10` | SSH / TCP 22 | Allow from `192.168.50.0/24` | Allow from workstation `.30` |
| Server A `.10` | HTTP / TCP 80 | Allow from `192.168.50.0/24` | Allow from workstation `.30` |
| Server B `.20` | SSH / TCP 22 | Allow from `192.168.50.0/24` | No inbound SSH allow rule |
| Server B `.20` | HTTP / TCP 80 | Allow from `192.168.50.0/24` | Allow from Kali `.40` |

Segmentation means destination-host filtering within the shared subnet. The experiment does not use separate VLANs, routed subnets, or separate VirtualBox networks.

The Server B HTTP permission applies to TCP 80 generally, not exclusively to `/status.txt`.

The implementation evaluates selected Zero Trust principles, particularly least privilege and reduced unnecessary reachability. It is not a complete Zero Trust Architecture.

## Threat Modeling

I used STRIDE to connect the scenarios to selected threats and control checks.

| Scenario | Path and purpose | STRIDE trace or control check |
|---|---|---|
| S1 | Workstation → Server A HTTP; authorized access | Availability check for overblocking |
| S2 | Workstation → Server A SSH; authorized login | Spoofing and availability |
| S3 | Kali → Server A SSH; unauthorized access attempt | Spoofing |
| S4 | Workstation → Server B SSH; attempted lateral access | Elevation of Privilege |
| S5 | Workstation → Server B HTTP; unnecessary service access | Information Disclosure |
| S6 | Kali → Server B shared HTTP status service; intentionally permitted | Information Disclosure risk from retained exposure |

Evidence collection supports Repudiation considerations. The experiment does not claim to test every threat within every STRIDE category.

Authorized-access scenarios verify that restrictions preserve required legitimate activity. S4 and S5 treat the workstation as an assumed compromised endpoint; no initial compromise was executed.

## Completed Experimental Testing

I evaluated all six scenarios under both architectural conditions. Each scenario produced five confirmed post-troubleshooting observations per condition, totaling 60 confirmed observations.

The six scenarios are the primary analytic units. Repetitions assessed consistency and were not treated as independent experimental samples.

| Scenario | Traditional P | Zero Trust-Oriented Z | Confirmed observations |
|---|---|---|---|
| S1 Workstation → A HTTP | Allowed; HTTP 200 | Allowed; HTTP 200 | 5/5 P; 5/5 Z |
| S2 Workstation → A SSH | Authorized login successful | Authorized login successful | 5/5 P; 5/5 Z |
| S3 Kali → A SSH | Service reachable; incorrect password rejected | Connection blocked; timeout | 5/5 P; 5/5 Z |
| S4 Workstation → B SSH | Service reachable; password prompt displayed | Connection blocked; timeout | 5/5 P; 5/5 Z |
| S5 Workstation → B HTTP | Allowed; HTTP 200 | Connection blocked; timeout | 5/5 P; 5/5 Z |
| S6 Kali → B `/status.txt` | Expected shared status response | Expected shared status response | 5/5 P; 5/5 Z |

### Results Interpretation

All six tested source-to-service paths were reachable under P. Under Z, S1, S2, and S6 remained available, while S3, S4, and S5 were blocked.

- Reachable paths decreased from **6/6 (100%) to 3/6 (50%)**.
- The three unnecessary paths decreased from **3/3 reachable to 0/3**.
- All three intended paths remained available under both conditions.

These counts describe the fixed scenario set, not enterprise-wide exposure.

An SSH password prompt demonstrates service reachability, not successful authentication. P already rejected the incorrect password in S3. No successful unauthorized SSH authentication, actual compromise, or completed lateral-movement attack was demonstrated under P.

S6 intentionally remained reachable under both conditions and documents residual exposure associated with a permitted shared service.

## Troubleshooting and Limitations

Preliminary testing included temporary client IPv4 address loss, HTTP connectivity failures, SSH command-entry errors, and a curl output-formatting error. I investigated these problems before recording the confirmed validations.

The 60 confirmed observations exclude preliminary configuration failures and command-entry errors. The complete number of preliminary attempts was not systematically recorded, so the confirmed results do not establish an overall first-attempt success rate.

Additional limitations include:

- Four VMs and six fixed scenarios limit generalizability.
- Static source-IP and TCP-port rules do not establish user or device identity.
- The lab does not implement continuous verification, device-posture assessment, a centralized policy engine, or application-level resource authorization.
- All hosts share one internal network.
- Address spoofing, stolen valid credentials, enforcement-host compromise, and production-scale performance were not evaluated.
- Client address persistence after reboot and separate-session reconstruction remain unverified.
- Some P screenshots show individual tests rather than every repetition.
- Repetitions do not establish statistical significance or production-scale effectiveness.

Detailed troubleshooting and limitations are documented in [`docs/troubleshooting-and-limitations.md`](docs/troubleshooting-and-limitations.md).

## Evidence and Reproducibility

Supporting documentation includes:

- [`docs/lab-setup.md`](docs/lab-setup.md) — topology, addressing, services, and reconstruction guidance
- [`docs/firewall-configurations.md`](docs/firewall-configurations.md) — P and Z firewall policies
- [`docs/troubleshooting-and-limitations.md`](docs/troubleshooting-and-limitations.md) — preliminary failures and implementation limits
- [`results/scenario-results.md`](results/scenario-results.md) — scenario outcomes and descriptive comparisons
- [`evidence/`](evidence/) — setup, configuration, testing, and troubleshooting screenshots

My documented evidence set includes 14 primary P/Z screenshots and three troubleshooting screenshots, alongside setup evidence and the topology diagram.

Local laboratory snapshots include:

- `Lab-Network-Ready`
- `Base-Lab-Services-Ready`
- `Traditional-P-Tests-Complete`
- `Zero-Trust-Z-Tests-Complete`

Before repeating tests, verify client addresses, required services, and active firewall rules. Preserve the distinction between service reachability, authentication denial, authorized login, and network blocking.

## Ethics and AI Use

I conducted all testing within an isolated laboratory under my control. No testing targeted production systems, employer systems, external organizations, public Internet targets, or systems without authorization. The project involved no human subjects or sensitive organizational datasets and excluded destructive malware, persistence mechanisms, and uncontrolled exploitation.

AI assistance supported technical questions and manuscript revision, including organization of previously reported results. AI did not execute the experiments or generate new experimental measurements.

I configured and operated the laboratory and remain responsible for the evidence, technical statements, references, interpretations, and final written work. Significant AI interactions should be documented under the course AI-use policy.

Public project materials are limited to synthetic laboratory data and configuration information. Passwords and sensitive information are excluded.

## Repository Structure

- `README.md` — project overview and completed findings
- `evidence/` — laboratory configuration and validation screenshots
- `results/` — experimental results and observation records
- `docs/` — setup, firewall, troubleshooting, and supporting documentation

## Tools

- Oracle VirtualBox
- Ubuntu Server
- Ubuntu Desktop
- Kali Linux
- UFW
- OpenSSH
- nginx
- Nmap
- curl

## Limitations and Future Research

Persistent client addressing after reboot and a separate-session reconstruction remain unverified.

Future work could evaluate identity-aware policies, application-level authorization, additional network segments, and realistic lateral-movement attempts.

## Status

**Comparative P/Z testing is complete.** The configured host firewall rules blocked three unnecessary service paths while preserving three intended paths. Results, supporting evidence, troubleshooting, and limitations are documented.
