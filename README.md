# Endpoint Forensics and Threat Hunting Lab
 
Endpoint detection and log analysis using Velociraptor and Splunk Enterprise. A Windows Server 2022 endpoint on AWS EC2 was enrolled into Velociraptor for live forensics, then wired into Splunk via the Universal Forwarder for SPL-based threat hunting.
 
**Stack:** Velociraptor v0.75.6 · Splunk Enterprise v10.2.2 · Splunk Universal Forwarder · AWS EC2 (Windows Server 2022) · VQL · SPL
 
**Full write-up:** [Velociraptor_Splunk_Writeup.pdf](Velociraptor_Splunk_Writeup.pdf) covers the full deployment, VQL queries, and SPL threat hunting in depth.
 
## Lab Environment
 
- Local Velociraptor server (Windows 10) with a remote AWS EC2 Windows Server 2022 endpoint enrolled over mutual TLS
- Splunk Enterprise instance receiving forwarded Windows event logs via the Universal Forwarder
## Endpoint Enrollment and Artifact Collection
 
The EC2 endpoint enrolled successfully with full host metadata populated (OS release, architecture, MAC address, first/last seen). A hunt using the `Windows.System.Pslist` artifact returned 73 process rows in 1 second, and a live VQL query in the Velociraptor Notebook mapped the full 75-row process tree by parent-child relationship:
 
```
SELECT Pid, Name, ParentName FROM pslist()
```
 
![Endpoint Enrollment](images/velociraptor-client-enrolled.png)
 
![Pslist Hunt](images/velociraptor-pslist-hunt.png)
 
![VQL Process Hierarchy](images/vql-process-hierarchy.png)
 
## PowerShell and Process Threat Hunting in Splunk
 
Two SPL queries were built against the forwarded logs:
 
- **Event ID 4104 (script block logging):** extracted the full text of executed PowerShell commands, matching 49 events and 10 distinct commands, confirming script block logging captures real administrative activity even when invoked through automation.
- **Event ID 4688 (process creation):** a timechart across 110 events and 91 unique process names surfaced a clear burst of Splunk worker processes in a four-minute window, reconstructing a precise timeline of when the forwarder was configured on the endpoint.
![PowerShell Event ID 4104](images/splunk-powershell-4104.png)
 
## Key Takeaway
 
Velociraptor and Splunk answer different questions: Velociraptor gives point-in-time forensic detail on a single endpoint, Splunk gives a searchable, aggregated view across time. Working with both together, rather than either alone, is what a real detection stack looks like.
 
## Repository Contents
 
- `README.md`
- `Velociraptor_Splunk_Writeup.pdf`
- `images/`
