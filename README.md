# IT Support Lab 02 – DNS Troubleshooting: Missing Internal A Record

## Overview

This repository documents a simulated IT support incident involving an internal benefits portal that became unreachable by hostname while the web application remained available at its known IP address.

The investigation identified a missing DNS A record in an Active Directory-integrated DNS zone. The record had been removed during a maintenance cleanup. The authorized record was restored, the client DNS cache was flushed, and name resolution was verified.

**Environment:** JoeLearns Labs – simulated environment  
**Incident ID:** INC-1187  
**Affected workstation:** HR-WS03 (Windows 11)  
**DNS server:** DC01 – `192.168.30.10`  
**Web server:** WEB-INT01  
**Internal DNS zone:** `northstar.local`  
**Affected hostname:** `benefits.northstar.local`  
**Expected A record:** `192.168.30.45`

> **Disclaimer:** This is a simulated training exercise completed in JoeLearns Labs. It does not represent production administration experience or access to a real corporate environment.

## 1. Incident Description

Human Resources employees could access public websites, but the internal benefits portal failed with a browser message indicating that the server IP address could not be found.

The web team reported that the application was still running. The task was to isolate the DNS issue, restore only the missing authorized record, clear obsolete client-side DNS cache data, and verify that the portal hostname resolved correctly.

## 2. Investigation

### Step 1 – Verify the workstation DNS configuration

On **HR-WS03**, the simulated `ipconfig /all` output showed:

| Parameter | Value |
|---|---|
| IPv4 address | `192.168.30.63` |
| Default gateway | `192.168.30.1` |
| DNS server | `192.168.30.10` |

The configured DNS server was DC01, the authorized internal DNS server. This ruled out an incorrectly configured client DNS server as the initial cause.

### Step 2 – Test name resolution

Executed:

```powershell
nslookup benefits.northstar.local
```

**Result:** The DNS server returned `NXDOMAIN` (non-existent domain). The internal hostname could not be found.

A public name was available for comparison, consistent with the ticket report that public websites remained reachable.

### Step 3 – Inspect the DNS zone

On **DC01**, the simulated DNS query for the `benefits` record in the `northstar.local` zone returned no matching resource record.

This confirmed that the issue was not simply an outdated client cache: the authoritative internal zone was missing the record.

### Step 4 – Review the DNS cleanup audit

The maintenance audit showed:

- `benefits.northstar.local` had been removed during change **CHG-7702**.
- The deleted address was `192.168.30.45`.
- The application inventory identified **WEB-INT01** as the owner of this hostname.
- The maintenance record did not include an application-owner review authorizing this specific deletion.

This connected the missing DNS record to the recent cleanup activity.

### Step 5 – Verify the application before restoring DNS

The application health inspection on **WEB-INT01** confirmed that the benefits portal was functioning and HTTPS was responding at:

```text
192.168.30.45:443
```

This established that the application was available at its known IP address and that restoring the hostname record was appropriate.

## 3. Root Cause

During the DNS cleanup performed under change **CHG-7702**, the A record for `benefits.northstar.local` was mistakenly deleted.

The web application itself remained operational on WEB-INT01 at `192.168.30.45`. Internal clients could not reach it by hostname because the authoritative DNS zone no longer contained the corresponding record.

## 4. Resolution

The following corrective actions were completed in the simulated environment:

1. Restored the authorized A record on **DC01**:
   `benefits.northstar.local → 192.168.30.45`.
2. Flushed the DNS resolver cache on **HR-WS03**. The simulation confirmed completion of `ipconfig /flushdns`.
3. Queried the hostname again from the client:

```powershell
nslookup benefits.northstar.local
```

**Final result:**

```text
Server:   dc01.northstar.local
Address:  192.168.30.10

Name:     benefits.northstar.local
Address:  192.168.30.45
```

The JoeLearns restoration checklist reached **3/3 completed checks**, and incident **INC-1187** was successfully closed.

## 5. Validation

| Validation | Result |
|---|---|
| Client uses authorized internal DNS server | Passed |
| Application responds at known IP and HTTPS port | Passed |
| Correct A record restored in the internal zone | Passed |
| Client DNS cache flushed | Passed |
| Hostname resolves to `192.168.30.45` | Passed |
| Incident restoration checklist | 3/3 completed |
| Unresolved disruptive changes | 0 |

## 6. Evidence

The screenshots below were captured during the simulated investigation. They document the ticket, DNS findings, audit evidence, application health check, corrective actions, successful name resolution, and incident closure.

| Evidence | Description |
|---|---|
| [01 – Incident ticket](evidence/01-incident-ticket.jpg) | Initial report and investigation objective |
| [02 – Client DNS and NXDOMAIN](evidence/02-client-dns-and-nxdomain.jpg) | Authorized DNS client configuration and failed hostname lookup |
| [03 – DNS cleanup audit](evidence/03-dns-cleanup-audit.jpg) | CHG-7702 deletion evidence and application ownership |
| [04 – Maintenance change record](evidence/04-maintenance-change-record.jpg) | Maintenance record supporting the investigation |
| [05 – Web application health](evidence/05-web-application-health.jpg) | WEB-INT01 responding at `192.168.30.45:443` |
| [06 – A record restored](evidence/06-a-record-restored.jpg) | Authorized internal A record restored |
| [07 – Client DNS cache flushed](evidence/07-client-dns-cache-flushed.jpg) | Client resolver cache cleared |
| [08 – Successful nslookup](evidence/08-successful-nslookup.jpg) | Hostname resolves to `192.168.30.45` |
| [09 – Incident closed](evidence/incident-closed.jpg) | Final incident result and root cause summary |

![Incident closed – JoeLearns Lab](evidence/incident-closed.jpg)

## 7. Skills Practiced

- Windows client network configuration review
- DNS troubleshooting and name resolution
- Interpreting `nslookup` and `NXDOMAIN`
- Active Directory-integrated DNS fundamentals
- DNS A records and internal zones
- Reviewing maintenance and audit evidence
- Verifying application availability before changing DNS
- Client DNS resolver cache management
- Root cause analysis
- Incident documentation and controlled remediation

## 8. Key Takeaways

This exercise reinforced a structured troubleshooting approach:

1. Verify the client configuration before changing it.
2. Use name-resolution tests to distinguish DNS failures from application availability issues.
3. Check the authoritative DNS zone rather than assuming that a client cache is the root cause.
4. Review change history and ownership evidence before restoring a record.
5. Confirm that the application responds at its known IP before making DNS changes.
6. Apply the smallest authorized correction, flush client cache, and validate name resolution.
7. Close the incident only after all restoration checks pass.

**Main lesson:** A DNS name-resolution failure does not necessarily mean that the application or web server is down. Verify each layer independently and use evidence to guide the change.
