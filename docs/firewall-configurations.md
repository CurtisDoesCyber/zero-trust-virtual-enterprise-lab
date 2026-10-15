# Firewall Configurations

## Project Overview

This document records the Uncomplicated Firewall (UFW) configurations used to compare a traditional perimeter-based architecture (P) with a Zero Trust-oriented architecture (Z) in an isolated VirtualBox enterprise lab.

The experiment uses two protected Ubuntu servers and two client systems connected to the VirtualBox internal network `enterprise-lab`.

| Virtual Machine | IP Address | Role |
|---|---|---|
| Protected-Server-A | 192.168.50.10 | Protected SSH and HTTP services |
| Protected-Server-B | 192.168.50.20 | Protected SSH and HTTP services, including shared status resource |
| Ubuntu-Workstation | 192.168.50.30 | Authorized workstation |
| Kali-Attacker | 192.168.50.40 | Simulated unauthorized client and explicitly authorized shared-service client |

All machines use the `192.168.50.0/24` subnet.

## 1. Traditional Perimeter-Based Configuration (P)

The perimeter-based configuration allows both SSH (TCP port 22) and HTTP (TCP port 80) from any system within the internal subnet.

The following commands were applied separately on **Protected-Server-A** and **Protected-Server-B**:

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.50.0/24 to any port 22 proto tcp
sudo ufw allow from 192.168.50.0/24 to any port 80 proto tcp
sudo ufw --force enable
sudo ufw status verbose
```

Under this condition, both the Ubuntu workstation and Kali attacker can reach the permitted services at the network level. Successful SSH authentication still requires valid account credentials.

## 2. Zero Trust-Oriented Configuration (Z)

The Zero Trust-oriented configuration replaces subnet-wide access with explicit source-IP-based rules. This is a simplified demonstration of least privilege and segmentation rather than a complete implementation of Zero Trust Architecture.

### 2.1 Protected-Server-A

Only the authorized Ubuntu workstation (`192.168.50.30`) is permitted to reach Server A's SSH and HTTP services.

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.50.30 to any port 22 proto tcp
sudo ufw allow from 192.168.50.30 to any port 80 proto tcp
sudo ufw --force enable
sudo ufw status verbose
```

Expected behavior:

- Workstation to Server A HTTP: Allowed.
- Workstation to Server A SSH: Allowed, subject to successful authentication.
- Kali to Server A SSH: Blocked by the firewall.

### 2.2 Protected-Server-B

Server B permits HTTP access from Kali (`192.168.50.40`) for the designated shared status service. No inbound SSH exception or workstation HTTP exception is configured.

```bash
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.50.40 to any port 80 proto tcp
sudo ufw --force enable
sudo ufw status verbose
```

Expected behavior:

- Workstation to Server B SSH: Blocked.
- Workstation to Server B HTTP: Blocked.
- Kali to Server B shared HTTP resource: Allowed.

The shared resource was accessed using:

```bash
curl http://192.168.50.20/status.txt
```

Expected response:

```text
Server B Shared Status Service - OK
```

## 3. Configuration Verification

After applying each configuration, the following command was used on each protected server:

```bash
sudo ufw status verbose
```

The output was inspected to confirm firewall activation, default policies, allowed source addresses, and permitted TCP ports.

Screenshots of the configuration and scenario tests are retained as experimental evidence.

## 4. Scope and Limitations

- UFW source-IP rules approximate selected Zero Trust principles but do not provide identity-aware, continuously evaluated access decisions.
- Both architectures use the same isolated internal subnet; logical segmentation is enforced by host-based firewall policies.
- Server B's HTTP rule permits Kali to access TCP port 80 generally, not only `/status.txt`. Resource-specific authorization was not implemented.
- SSH service reachability does not establish successful authentication or compromise.
- Manual IPv4 address restoration was required during some tests because client addressing was not consistently maintained. These interruptions are documented separately in the experimental results and limitations.
- Firewall configurations were applied and tested in a controlled virtual lab, not a production environment.
