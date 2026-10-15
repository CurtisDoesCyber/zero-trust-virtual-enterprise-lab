# Troubleshooting and Experimental Limitations

## 1. Purpose

This document records the technical difficulties, corrective actions, and methodological limitations encountered during the project *Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab*.

The experiment compared a traditional perimeter-based security configuration (P) with a Zero Trust-oriented configuration (Z) using six defined access scenarios in a four-VM VirtualBox environment.

Recording these issues helps distinguish expected security-policy behavior from unrelated lab failures and supports transparent interpretation of the findings.

## 2. Troubleshooting Record

### 2.1 Temporary Loss of IPv4 Addresses

**Issue:** During testing, the Ubuntu workstation and Kali VM temporarily lost their intended IPv4 addresses. This interrupted connectivity even when the firewall policy should have allowed access.

The intended client addresses were:

- Ubuntu-Workstation: `192.168.50.30/24`
- Kali-Attacker: `192.168.50.40/24`

**Corrective action:** The addresses were restored manually using the appropriate network interfaces.

Ubuntu workstation:

```bash
sudo ip addr add 192.168.50.30/24 dev enp0s3
```

Kali:

```bash
sudo ip addr add 192.168.50.40/24 dev eth0
```

**Outcome:** Connectivity was restored, allowing the affected tests to be repeated.

**Remaining limitation:** These commands provide temporary addressing. Persistent configuration after reboot was not verified for both clients.

### 2.2 Zero Trust Scenario S1 — Preliminary HTTP Failures

**Issue:** The first HTTP request from the workstation to Server A returned HTTP 200. Four subsequent preliminary requests failed after the workstation lost its intended IP address.

**Corrective action:** The workstation's address was corrected to `192.168.50.30/24`, and the scenario was repeated.

**Outcome:** Five subsequent validation requests returned HTTP 200.

**Interpretation:** The preliminary failures reflected client-network configuration instability rather than the intended Zero Trust firewall restriction.

### 2.3 Traditional Perimeter HTTP Testing — Temporary Connectivity Failures

**Issue:** Temporary HTTP connection failures occurred during some perimeter-based tests, including access to Server B.

**Corrective action:** Client connectivity and addressing were checked, and the affected requests were repeated.

**Outcome:** Five successful post-troubleshooting observations were recorded for each affected scenario.

**Interpretation:** The initial failures are not treated as evidence that the perimeter firewall intentionally blocked those connections.

### 2.4 Zero Trust Scenario S3 — SSH Command Errors

**Issue:** The initial repeated SSH test command contained an unmatched quotation mark. A later attempt misspelled the `BatchMode` SSH option, producing local command errors.

**Corrective action:** The test command was corrected and rerun.

**Outcome:** Five valid SSH attempts from Kali to Server A timed out under Condition Z.

**Interpretation:** The earlier command errors did not represent valid network-policy observations.

### 2.5 Zero Trust Scenario S5 — HTTP Output Formatting

**Issue:** The `curl` output-formatting expression did not display the intended HTTP status value.

**Outcome:** The five repeated attempts still produced connection timeout errors (`curl: (28)`), consistent with the expected firewall restriction.

**Interpretation:** The timeout messages were used as evidence of blocked connectivity. The incorrectly formatted status output was not treated as an HTTP response code.

### 2.6 Kali Network Configuration

**Issue:** An attempt to reactivate Kali's NetworkManager connection profile timed out and temporarily removed the active IPv4 address.

**Corrective action:** The intended address was restored manually.

**Outcome:** Testing resumed successfully.

**Remaining limitation:** Although a manual IPv4 connection profile was configured, persistence across a VM restart was not confirmed.

## 3. Methodological Limitations

### 3.1 Simplified Zero Trust Implementation

The Zero Trust-oriented condition used UFW source-IP and TCP-port allow rules to demonstrate least-privilege network access.

It did not implement all components of a complete Zero Trust Architecture, such as:

- Continuous user and device verification.
- Centralized identity-aware policy decision and enforcement.
- Dynamic device posture assessment.
- Application-level authorization for individual resources.

The findings therefore concern a Zero Trust-oriented firewall policy rather than a fully implemented enterprise Zero Trust Architecture.

### 3.2 Network Reachability Versus Compromise

The six scenarios primarily tested whether a client could reach an SSH or HTTP service.

A reachable SSH login prompt does not mean the client authenticated successfully.

No unauthorized successful SSH login or completed lateral-movement attack was demonstrated under the perimeter-based condition.

Consequently, the experiment measures restrictions on network-level access paths, not a directly observed reduction in successful compromises.

### 3.3 Shared HTTP Resource Access

Under Condition Z, Kali was permitted to access TCP port 80 on Server B to retrieve `/status.txt`.

The UFW rule was not limited to that particular file or URL path.

Therefore, the test demonstrates source-specific HTTP service access, not resource-level authorization.

### 3.4 Limited Scale and Environment

The experiment used four VMs on one isolated VirtualBox internal network.

It did not include multiple routed subnets, enterprise identity infrastructure, production traffic, endpoint management, or complex organizational access requirements.

The findings should not be generalized directly to larger enterprise deployments.

### 3.5 Repetition and Preliminary Failures

The experiment used six scenarios, two security conditions, and five confirmed post-troubleshooting validation repetitions per scenario and condition.

These produced 60 confirmed validation observations, but they do not represent all attempts made during troubleshooting.

Preliminary failures and command-entry errors occurred, and their complete count was not systematically recorded.

The five repetitions were consistency checks. They were not independent analytic units, and no statistical significance testing was conducted.

## 4. Implications for the Findings

The central result was that the Zero Trust-oriented configuration restricted three network access paths that remained reachable under the perimeter-based configuration:

- Kali to Server A SSH.
- Workstation to Server B SSH.
- Workstation to Server B HTTP.

Three intended access paths remained available:

- Workstation to Server A HTTP.
- Workstation to Server A authenticated SSH.
- Kali to Server B shared HTTP service.

These observations support a narrow conclusion: explicit source-based firewall permissions reduced unnecessary network service exposure in this lab while preserving the designated allowed connections.

They do not establish that a complete Zero Trust Architecture was deployed or that actual attacker lateral movement was prevented during a live compromise.

## 5. Conclusion

The experiment demonstrated a reproducible difference between subnet-wide access rules and more restrictive source-specific firewall rules after addressing temporary network and command-entry problems.

Documenting the preliminary failures, corrective actions, and implementation limitations provides necessary context for interpreting the results.

Future work could evaluate persistent network configuration, identity-aware access controls, application-specific authorization, additional network segments, and realistic adversarial lateral-movement attempts.
