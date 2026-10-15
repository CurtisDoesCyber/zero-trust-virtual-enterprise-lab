# Virtual Enterprise Lab Setup and Reproduction Guide

## 1. Purpose

This guide documents the virtual lab used in the project *Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab*.

The lab compares a traditional perimeter-based security architecture (P) against a Zero Trust-oriented security architecture (Z). Both configurations use the same virtual machines, network, and services. The primary experimental difference is the firewall access policy.

The environment is isolated from external networks and uses host-based UFW firewall rules to demonstrate segmentation and least-privilege access.

## 2. Software and Virtual Machines

The lab was built using Oracle VirtualBox and four virtual machines.

| Virtual Machine | Operating System | IP Address | Purpose |
|---|---|---|---|
| Protected-Server-A | Ubuntu Server 26.04.1 | 192.168.50.10 | Protected SSH and HTTP services |
| Protected-Server-B | Ubuntu Server 26.04.1 | 192.168.50.20 | Protected SSH and HTTP services and shared status resource |
| Ubuntu-Workstation | Ubuntu Desktop 24.04.5 | 192.168.50.30 | Authorized internal workstation |
| Kali-Attacker | Kali Linux 2026.2 | 192.168.50.40 | Simulated unauthorized client and permitted shared-service client |

All systems use the `192.168.50.0/24` network.

## 3. VirtualBox Network Configuration

Configure each VM using the following settings:

1. Open VirtualBox Manager.
2. Select the virtual machine.
3. Open **Settings → Network**.
4. Enable **Adapter 1**.
5. Set **Attached to** to **Internal Network**.
6. Enter `enterprise-lab` as the network name.
7. Disable Adapters 2, 3, and 4.
8. Repeat these steps for all four VMs.

The internal network provides connectivity between the lab machines without exposing the experiment to the host's external network.

No default gateway or DNS server is required for the isolated testing network.

## 4. IPv4 Address Configuration

Assign the following static IPv4 addresses:

- Protected-Server-A: `192.168.50.10/24`
- Protected-Server-B: `192.168.50.20/24`
- Ubuntu-Workstation: `192.168.50.30/24`
- Kali-Attacker: `192.168.50.40/24`

Confirm the actual interface name on each machine using:

```bash
ip -br addr
```

For temporary manual configuration, an address can be assigned using:

```bash
sudo ip addr add ADDRESS/24 dev INTERFACE
```

Replace `ADDRESS` and `INTERFACE` with the correct values for that VM.

For example, the Ubuntu workstation used `enp0s3` and Kali used `eth0` during troubleshooting.

**Important:** The `ip addr add` command is temporary and may not survive a reboot. Configure persistent addressing using the operating system's supported network configuration method before relying on the environment for repeated testing.

During the experiment, temporary loss of client IP addresses required manual restoration. Those interruptions are disclosed in the results and limitations documentation.

## 5. Protected Server Services

Both protected servers require SSH and Nginx HTTP services.

On each Ubuntu server, install the packages if they are not already installed:

```bash
sudo apt update
sudo apt install -y openssh-server nginx ufw
```

Confirm SSH is running:

```bash
sudo systemctl status ssh
```

Confirm Nginx is running:

```bash
sudo systemctl status nginx
```

Check listening ports:

```bash
sudo ss -tlnp
```

SSH should listen on TCP port 22,
