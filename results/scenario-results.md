# Experimental Scenario Results

## 1. Experiment Overview

**Project:** Evaluating Segmentation and Least Privilege in a Virtual Enterprise Lab

**Research question:** In a controlled virtual enterprise network environment, how does a Zero Trust-oriented security architecture compare with a traditional perimeter-based security architecture in preventing unauthorized access and limiting lateral movement under defined cybersecurity attack scenarios?

Two security configurations were evaluated:

- **Condition P:** Traditional perimeter-based architecture using subnet-wide SSH and HTTP firewall permissions.
- **Condition Z:** Zero Trust-oriented architecture using source-specific firewall permissions.

The experiment included six scenarios. Each scenario was evaluated under both configurations using five post-troubleshooting validation repetitions.

The six scenarios are the primary analytic units. The five repetitions per condition are consistency checks, not independent experimental samples.

## 2. Lab Environment

| System | IP Address | Role |
|---|---|---|
| Protected-Server-A | 192.168.50.10 | Protected SSH and HTTP server |
| Protected-Server-B | 192.168.50.20 | Protected SSH, HTTP, and shared status resource |
| Ubuntu-Workstation | 192.168.50.30 | Authorized workstation |
| Kali-Attacker | 192.168.50.40 | Simulated unauthorized client and permitted shared-resource client |

All four virtual machines were connected to the isolated VirtualBox internal network `enterprise-lab`.

## 3. Scenario Results

| Scenario | Connection Tested | Traditional P — Observed | Zero Trust Z — Observed | Confirmed Repetitions |
|---|---|---|---|---|
| S1 | Workstation → Server A HTTP | HTTP 200; allowed | HTTP 200; allowed | 5/5 in P and 5/5 in Z |
| S2 | Workstation → Server A SSH | Authenticated login allowed | Authenticated login allowed | 5/5 in P and 5/5 in Z |
| S3 | Kali → Server A SSH | SSH service reachable; incorrect password rejected | Connection timed out; firewall blocked | 5/5 in P and 5/5 in Z |
| S4 | Workstation → Server B SSH | SSH service reachable; password prompt observed | Connection timed out; firewall blocked | 5/5 in P and 5/5 in Z |
| S5 | Workstation → Server B HTTP | HTTP 200; allowed | Connection timed out; firewall blocked | 5/5 in P and 5/5 in Z |
| S6 | Kali → Server B `/status.txt` | Shared status response received | Shared status response received | 5/5 in P and 5/5 in Z |

**Interpretation:** All six scenarios produced their expected outcomes during the five confirmed post-troubleshooting repetitions in each condition. Additional preliminary attempts experienced connectivity or command-entry problems and are disclosed below.

## 4. Comparison of Security Conditions

### 4.1 Access Preserved

Three intended connections remained available under Condition Z:

- **S1:** Authorized workstation HTTP access to Server A.
- **S2:** Authorized workstation SSH access to Server A.
- **S6:** Kali HTTP access to Server B's shared status resource.

This demonstrates that the restrictive firewall policy preserved the specific network paths designated as permitted.

### 4.2 Unnecessary Network Access Restricted

Three connections that were reachable under Condition P were blocked under Condition Z:

- **S3:** Kali SSH access to Server A.
- **S4:** Workstation SSH access to Server B.
- **S5:** Workstation HTTP access to Server B.

Condition P exposed the relevant network services to the entire internal subnet. Condition Z restricted those services using explicit source-IP rules.

The results show reduced network-level service reachability, not proof that an attacker successfully compromised or laterally moved through the perimeter-based environment.

### 4.3 Overall Result

Condition Z blocked three of the six tested connection paths while preserving the other three intended paths.

Condition P permitted network-level access for all six paths, although SSH authentication was not successful in every scenario.

This distinction is important: reaching an SSH login prompt is not equivalent to obtaining unauthorized access to the system.

## 5. Repetition and Consistency

The experimental plan specified:

- Six scenarios.
- Two security conditions.
- Five validation repetitions per scenario per condition.

This corresponds to **60 confirmed post-troubleshooting validation observations**.

The repetitions were used to check whether the observed network behavior remained consistent once the lab was correctly configured.

They were not treated as 60 independent analytic units, and no statistical significance testing was performed.

The 60 confirmed observations should not be interpreted as all attempted executions. Preliminary failed attempts occurred and were excluded from the post-troubleshooting validation counts.

## 6. Troubleshooting and Deviations

Several issues arose during setup and testing.

### 6.1 Temporary Loss of Client IP Addresses

The Ubuntu workstation and Kali VM temporarily lost their intended static IPv4 addresses during parts of the experiment.

The workstation address was restored to `192.168.50.30/24`, and Kali's address was restored to `192.168.50.40/24`.

For example, temporary manual address restoration used commands of this form:

```bash
sudo ip addr add 192.168.50.30/24 dev enp0s3
```

```bash
sudo ip addr add 192.168.50.40/24 dev eth0
```

These commands restore an address for the active session but do not guarantee persistence across restarts.

### 6.2 Preliminary HTTP Connectivity Failures

During Zero Trust Scenario S1, an initial HTTP 200 response was followed by four failed preliminary requests after the workstation lost its expected IP address.

The workstation address was corrected, and five subsequent validation requests returned HTTP 200.

Temporary connectivity failures also occurred during some perimeter-based HTTP testing before successful validation repetitions were completed.

### 6.3 SSH Command-Entry Errors

During Zero Trust Scenario S3, an initial SSH test loop contained an unmatched quotation mark.

A subsequent command incorrectly spelled an SSH option, producing local configuration errors rather than valid network-test results.

After correcting the commands, five SSH connection attempts timed out as expected under the restrictive firewall policy.

### 6.4 HTTP Output Formatting Issue

During Zero Trust Scenario S5, the output-formatting expression used with `curl` did not print the intended HTTP status code.

However, the repeated `curl: (28)` connection timeout errors provided evidence that the HTTP requests could not reach the service.

### 6.5 Validation Counting

The results table reports five confirmed observations per scenario and condition after troubleshooting.

The preliminary failures and command-entry errors were not counted as successful policy validations. The full number of preliminary attempts was not systematically recorded.

Accordingly, the results support consistency of the confirmed validation repetitions but do not establish an overall first-attempt success rate.

## 7. Limitations and Conclusion

The experiment has several limitations:

1. The Zero Trust-oriented condition uses static source-IP-based UFW rules. It does not implement a full identity-aware Zero Trust Architecture.
2. The scenarios primarily measure network service reachability and authorized SSH authentication, not exploitation or demonstrated lateral movement.
3. No unauthorized successful SSH authentication was observed under Condition P.
4. The shared HTTP permission on Server B allows Kali to reach TCP port 80 generally, rather than enforcing access only to `/status.txt`.
5. Temporary client IP-address instability affected preliminary test attempts.
6. The small, isolated four-VM lab does not represent the complexity of a production enterprise network.
7. Five repeated validations per scenario establish limited within-lab consistency, not statistical generalizability.

**Conclusion:** In this controlled experiment, the Zero Trust-oriented firewall policy removed three unnecessary network-level access paths while preserving three intended paths. The findings support the value of explicit least-privilege firewall rules for reducing service exposure within this lab, while remaining limited to the specific configurations and scenarios tested.
