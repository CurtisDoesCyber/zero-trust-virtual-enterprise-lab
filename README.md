# Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab

This repository contains the implementation artifact for my MSICT cybersecurity capstone project. The project compares a traditional perimeter-based security architecture with a Zero Trust-oriented architecture in a controlled four-VM VirtualBox environment.

## Research Question

In a controlled virtual enterprise network environment, how does a Zero Trust-oriented security architecture compare with a traditional perimeter-based security architecture in preventing unauthorized access and limiting lateral movement under defined cybersecurity attack scenarios?

## Lab Environment

The lab consists of four virtual machines connected to an isolated VirtualBox Internal Network named `enterprise-lab`.

| Virtual Machine | Role | IP Address |
|---|---|---|
| Kali-Attacker | Security testing system | 192.168.50.40 |
| Ubuntu-Workstation | Enterprise workstation / simulated compromised endpoint | 192.168.50.30 |
| Protected-Server-A | Protected server | 192.168.50.10 |
| Protected-Server-B | Protected server | 192.168.50.20 |

The protected servers provide SSH and HTTP services. Server B also contains a shared HTTP status resource used in one of the experimental scenarios.

## Architecture Conditions

### Traditional Perimeter-Based Condition (P)

The traditional condition represents a basic small-enterprise internal network. Internal systems are generally permitted to reach declared SSH and HTTP services, while authentication remains enabled.

### Zero Trust-Oriented Condition (Z)

The Zero Trust-oriented condition uses the same topology and services but applies more restrictive host-based firewall rules. Access is limited according to source, destination, and required service.

The implementation is intended to demonstrate selected Zero Trust principles such as least privilege and reduced implicit trust. It is not intended to represent a complete production Zero Trust Architecture.

## Threat Modeling

The project uses the STRIDE threat-modeling framework to connect each experimental scenario to a defined threat category and security control.

The six scenarios evaluate:

1. Authorized workstation HTTP access to Server A
2. Authorized workstation SSH access to Server A
3. Unauthorized Kali SSH access to Server A
4. Workstation lateral SSH access attempt to Server B
5. Workstation HTTP access to an unnecessary service on Server B
6. Kali access to an intentionally retained shared HTTP status service on Server B

Authorized-access scenarios are included to verify that security restrictions do not prevent required legitimate access.

## Current Artifact Status

The four-VM environment has been successfully built and configured.

Completed checkpoint activities include:

- Four virtual machines created and configured
- Static addressing configured on the `192.168.50.0/24` internal network
- Cross-VM connectivity verified
- SSH services verified on the protected servers
- Nginx HTTP services installed and verified
- Shared HTTP status resource configured on Server B
- Authorized SSH access from Ubuntu-Workstation to Server A verified
- HTTP connectivity from the workstation to both protected servers verified
- Nmap, curl, and OpenSSH verified on Kali-Attacker
- Clean pre-experiment VM snapshots created

## Next Steps

The next phase will configure and test the Traditional (P) and Zero Trust-oriented (Z) firewall conditions. Each of the six scenarios will be executed five times under each architecture condition for consistency checking.

Observed results will be compared using descriptive counts and percentages for service reachability, unauthorized-access prevention, and preservation of authorized access.

## Repository Structure

- `README.md` — project and artifact overview
- `evidence/` — screenshots documenting lab configuration and validation
- `results/` — experimental results and observation records
- `docs/` — supporting project documentation

## Tools

- Oracle VirtualBox
- Ubuntu Server
- Ubuntu Desktop
- Kali Linux
- UFW
- OpenSSH
- Nginx
- Nmap
- curl

## Status

**Artifact checkpoint:** Base four-VM environment operational. Comparative P/Z experimental testing is the next implementation phase.
