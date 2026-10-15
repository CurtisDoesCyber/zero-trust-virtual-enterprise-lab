Virtual Enterprise Lab Setup and Reproduction Guide

1. Purpose

This guide documents the virtual lab used in the project Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab.

The lab compares a traditional perimeter-based security architecture (P) against a Zero Trust-oriented security architecture (Z). Both configurations use the same virtual machines, network, and services. The primary experimental difference is the firewall access policy.

The environment is isolated from external networks and uses host-based UFW firewall rules to demonstrate segmentation and least-privilege access.

2. Software and Virtual Machines

The lab was built using Oracle VirtualBox and four virtual machines.

Virtual Machine

Operating System

IP Address

Purpose

Protected-Server-A

Ubuntu Server 26.04.1

192.168.50.10

Protected SSH and HTTP services

Protected-Server-B

Ubuntu Server 26.04.1

192.168.50.20

Protected SSH and HTTP services and shared status resource

Ubuntu-Workstation

Ubuntu Desktop 24.04.5

192.168.50.30

Authorized internal workstation

Kali-Attacker

Kali Linux 2026.2

192.168.50.40

Simulated unauthorized client and permitted shared-service client

All systems use the 192.168.50.0/24 network.

3. VirtualBox Network Configuration

Configure each VM using the following settings:

Open VirtualBox Manager.

Select the virtual machine.

Open Settings → Network.

Enable Adapter 1.

Set Attached to to Internal Network.

Enter enterprise-lab as the network name.

Disable Adapters 2, 3, and 4.

Repeat these steps for all four VMs.

The internal network provides connectivity between the lab machines without exposing the experiment to the host's external network.

No default gateway or DNS server is required for the isolated testing network.

4. IPv4 Address Configuration

Assign the following static IPv4 addresses:

Protected-Server-A: 192.168.50.10/24

Protected-Server-B: 192.168.50.20/24

Ubuntu-Workstation: 192.168.50.30/24

Kali-Attacker: 192.168.50.40/24

Confirm the actual interface name on each machine using:

ip -br addr

For temporary manual configuration, an address can be assigned using:

sudo ip addr add ADDRESS/24 dev INTERFACE

Replace ADDRESS and INTERFACE with the correct values for that VM.

For example, the Ubuntu workstation used enp0s3 and Kali used eth0 during troubleshooting.

Important: The ip addr add command is temporary and may not survive a reboot. Configure persistent addressing using the operating system's supported network configuration method before relying on the environment for repeated testing.

During the experiment, temporary loss of client IP addresses required manual restoration. Those interruptions are disclosed in the results and limitations documentation.

5. Protected Server Services

Both protected servers require SSH and Nginx HTTP services.

On each Ubuntu server, install the packages if they are not already installed:

sudo apt update
sudo apt install -y openssh-server nginx ufw

Confirm SSH is running:

sudo systemctl status ssh

Confirm Nginx is running:

sudo systemctl status nginx

Check listening ports:

sudo ss -tlnp

SSH should listen on TCP port 22, and Nginx should listen on TCP port 80.

6. Server A Authorized SSH Account

Server A uses an SSH account named analyst for the authorized workstation access scenario.

If rebuilding the lab from scratch, create the account on Server A:

sudo adduser analyst

Set a test-only password when prompted. Do not publish passwords or other credentials in the GitHub repository.

Successful authentication is tested separately from basic network reachability.

7. Server B Shared HTTP Resource

Server B hosts the shared status file used in Scenario 6.

Create the file on Server B:

echo 'Server B Shared Status Service - OK' | sudo tee /var/www/html/status.txt

Verify it locally:

curl http://127.0.0.1/status.txt

The expected response is:

Server B Shared Status Service - OK

The experiment uses this resource to demonstrate an explicitly permitted HTTP connection under the Zero Trust-oriented policy.

8. Apply the Two Security Conditions

Condition P — Traditional Perimeter

Apply the traditional perimeter-based UFW rules to both protected servers.

These rules permit SSH and HTTP traffic from the entire 192.168.50.0/24 subnet.

Condition Z — Zero Trust-Oriented

Replace the perimeter rules with source-specific allow rules.

Server A allows SSH and HTTP from the Ubuntu workstation only.

Server B allows HTTP from Kali only.

Other inbound connections are denied by default.

The exact commands for both conditions are documented in:

docs/firewall-configurations.md

Reset and apply the correct configuration before testing each condition.

9. Scenario Testing

Six scenarios are used as the primary units of analysis.

Scenario

Source

Destination

Service

Expected P

Expected Z

S1

Workstation

Server A

HTTP

Allowed

Allowed

S2

Workstation

Server A

SSH

Allowed with valid credentials

Allowed with valid credentials

S3

Kali

Server A

SSH

Service reachable; invalid credentials rejected

Firewall blocked

S4

Workstation

Server B

SSH

Service reachable; authentication not established

Firewall blocked

S5

Workstation

Server B

HTTP

Allowed

Firewall blocked

S6

Kali

Server B

Shared HTTP resource

Allowed

Allowed

Use curl to test HTTP connectivity and ssh to test SSH connectivity.

For each scenario, repeat the test five times under Condition P and five times under Condition Z.

The five repetitions are consistency checks, not five independent scenarios.

Record whether the observed behavior matches the expected access policy. Distinguish between an SSH connection reaching the authentication stage and a user successfully authenticating.

10. Preserve Experimental States

VirtualBox snapshots were used to preserve important lab configurations.

Snapshot names included:

Base-Lab-Services-Ready

Traditional-P-Tests-Complete

Zero-Trust-Z-Tests-Complete

Snapshots allow the researcher to return to an earlier configuration without rebuilding the environment.

Restoring a snapshot can overwrite the current VM state. Preserve the existing snapshots and review the intended state before restoring one.

11. Reproduction and Verification

A reviewer can reproduce the central comparison by:

Creating the four VMs and isolated internal network.

Assigning the documented IP addresses.

Installing and verifying SSH and Nginx.

Creating the Server A analyst account and Server B shared status file.

Applying Condition P firewall rules to both servers.

Running and recording all six scenarios.

Replacing the firewall rules with Condition Z.

Repeating the same six scenarios.

Comparing service reachability and successful authentication separately.

Checking the results against the documented access-policy expectations.

12. Experimental Limitations

This is a controlled demonstration of host-based network access restrictions, not a complete enterprise Zero Trust deployment.

The implementation does not include continuous identity verification, centralized policy decision points, or dynamic device posture assessment.

The test scenarios primarily evaluate network service reachability and authorized SSH access. They do not demonstrate successful lateral movement or exploitation.

Server B's UFW rule allows Kali to reach TCP port 80 generally; it does not restrict access to only /status.txt.

Some preliminary test attempts were interrupted by temporary client IP-address loss and command-entry errors. Five post-troubleshooting validation observations were collected for each scenario and condition. These interruptions must be considered when interpreting reproducibility and consistency.

The findings apply to this small virtual lab and should not be generalized to production enterprise environments without additional testing.
