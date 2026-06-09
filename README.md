# Incident-Report-BURNINCANDLES



> **Network Traffic Analysis | Wireshark | Kerberos | NBNS**

---

## Executive Summary

Suspicious network activity was detected in the LAN segment `10.0.19.0/24`. One Windows client (`10.0.19.14`) was identified as infected after communicating with multiple external IP addresses. The infected host attempted to reach malicious domains and Command & Control (C2) servers, indicating a full system compromise.

---

## Objectives

- Identify the victim machine involved in the malicious traffic
- Extract key attributes: IP address, MAC address, hostname, and user account
- Analyze Kerberos authentication traffic to confirm compromise
- Provide actionable recommendations to mitigate similar incidents

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Wireshark** | Packet capture analysis and protocol inspection |
| **Display Filters** | Filters such as `ip.addr == 10.0.19.14 && kerberos` to isolate suspicious traffic |
| **Kerberos Protocol Analysis** | Expanded fields (`cname → name-string`, `sname → sname-string`) to identify user accounts and domain controller |
| **NBNS Traffic Inspection** | Extracted hostname information from NetBIOS Name Service packets |
| **Ethernet & IP Headers** | Retrieved MAC and IP addresses from captured frames |

---

## Analysis — Step by Step

### Step 1 — Identify the Infected Host

**Method:** `Statistics → Endpoints → IPv4`

Compared all internal hosts in the `10.0.19.0/24` subnet. Host `10.0.19.14` stood out with **15,350 packets / 7 MB** of traffic — significantly higher than any other internal host.

<img width="608" height="488" alt="GetImage (16)" src="https://github.com/user-attachments/assets/84fd6ff8-0d77-4c9d-928b-f17cb2352b37" />


**Conclusion:** `10.0.19.14` is the infected Windows client.

---

### Step 2 — Identify the MAC Address

**Method:** `Statistics → Endpoints → Ethernet`

The MAC address with the highest packet count (15,350 packets, 7 MB) matched the traffic volume of the infected host IP `10.0.19.14`. This correlation confirms the MAC address of the compromised client.

| Attribute | Value |
|-----------|-------|
| **IP Address** | `10.0.19.14` |
| **MAC Address** | `00:60:52:b7:33:0f` |

<img width="971" height="521" alt="GetImage (17)" src="https://github.com/user-attachments/assets/ccbffd15-3cac-400a-a5c6-df8b25199526" />


---

### Step 3 — Identify the Hostname

**Method:** NBNS (NetBIOS Name Service) traffic filtered for `10.0.19.14`

During analysis of NBNS traffic, the hostname was identified in the **Query Name** field. The client registered itself on the network using the name `BURNINCANDLE`.

| Attribute | Value |
|-----------|-------|
| **Hostname** | `BURNINCANDLE` |
| **IP Address** | `10.0.19.14` |
| **MAC Address** | `00:60:52:b7:33:0f` |


<img width="473" height="274" alt="GetImage (18)" src="https://github.com/user-attachments/assets/bb49d324-b96d-4b87-9475-dfef55586cde" />

---

### Step 4 — Identify the Windows User Account

**Method:** Kerberos authentication traffic filtered for `10.0.19.14`

In the Kerberos packet details, the field `cname → name-string` revealed the account used by the compromised host.

| Attribute | Value |
|-----------|-------|
| **Windows User Account** | `burnincandle` |
| **Hostname** | `BURNINCANDLE` |
| **IP Address** | `10.0.19.14` |
| **MAC Address** | `00:60:52:b7:33:0f` |

---

### Step 5 — Identify the Domain Controller

**Method:** Kerberos authentication traffic — field `sname → sname-string`

The Domain Controller associated with the infected client was identified by analyzing Kerberos authentication traffic.

| Attribute | Value |
|-----------|-------|
| **Domain Controller** | `BURNINCANDLE-DC.burnincandle.com` |
| **Realm** | `BURNINCANDLE.COM` |
| **Hostname** | `BURNINCANDLE` |
| **IP Address** | `10.0.19.14` |
| **MAC Address** | `00:60:52:b7:33:0f` |

---

## Findings Summary

| Attribute | Value |
|-----------|-------|
| **IP Address** | `10.0.19.14` |
| **MAC Address** | `00:60:52:b7:33:0f` |
| **Hostname** | `BURNINCANDLE` |
| **Windows User Account** | `burnincandle` |
| **Desktop Name** | `DESKTOP-5QS3D5D` |
| **Domain Controller** | `BURNINCANDLE-DC.burnincandle.com` |
| **Realm** | `BURNINCANDLE.COM` |

---

## Recommendations

### 1. Immediate Containment
Isolate the infected client (`DESKTOP-5QS3D5D`) from the network to prevent further lateral movement or data exfiltration.

### 2. Credential Reset
Reset machine account credentials and review domain controller logs for any unauthorized access or privilege escalation.

### 3. Patch & Update
Ensure the client and domain controller are fully patched against known vulnerabilities, particularly those exploitable via Kerberos.

### 4. Monitoring
Deploy continuous monitoring for Kerberos anomalies and suspicious authentication attempts, especially for `AS-REQ`/`AS-REP` patterns.

### 5. User Awareness
Train staff on recognizing signs of compromise and reporting unusual activity, such as unexpected authentication prompts or slow network performance.

---

## Conclusion

The analysis successfully identified the victim machine and confirmed malicious Kerberos activity involving the domain controller. The compromised host was traced to IP `10.0.19.14` with hostname `BURNINCANDLE` and account `DESKTOP-5QS3D5D`. The evidence highlights the importance of proactive monitoring and rapid incident response to mitigate domain-level compromises.

---

> **File analyzed:** `2022-03-21-traffic-analysis-exercise.pcap`  
> **Analyst:** *Pedro Barillas*  
> **Date:** *06/08/2026*
