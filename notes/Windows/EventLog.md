# Event Log Forensics

## Event Logging

- **Purpose:** Provides a centralized method for the OS and applications to record important events, like system changes, user actions, errors, or security events.
## Event Log System

- Events are collected by the Event Logging Service and stored in an event log.
- Logs are aggregated from multiple sources into one accessible system.
## Value of event logs

- Key Questions Event Logs Can Help Answer
  - What Happened?
    - **Event logs provide detailed info via:**
      - Event IDs
      - Event Categories
      - Event Descriptions
    - These elements reveal what occurred on the system, though logs can appear cryptic to non-experts.
  - Date/Time?
    - Timestamps establish temporal context for events.
    - Useful for narrowing focus when analyzing thousands of events.
  - Users Involved?
    - Logs tie actions to specific user accounts.
    - Includes both standard users and system-level accounts like System and NetworkService.
  - Systems Involved?
    - **Useful in networked environments:**
      - Tracks remote access to resources.
      - Initially used NetBIOS names, now logs IP addresses (post-Windows 2000).
  - Resources Accessed?
    - Event logs can show granular access info on system objects.
    - Helps identify unauthorized access to sensitive files or operations (critical for security auditing).
## Fundamental

- History of Windows Event Logs
  - Introduced in NT 3.1 (1993)
  - Original format used .evt extension.
  - Stored in binary using circular buffers (oldest entries overwritten).
  - **Logs were located in:**
    - `%systemroot%\System32\config`
- **Major Overhaul:** Vista / Server 2008 and Later
  - **Switched to .evtx format:**
    - Addresses performance issues.
    - Easier parsing and string searching.
    - Supports many more logs (300+).
  - **Logs moved to:**
    - `%systemroot%\System32\winevt\logs`
- Remote Logging
  - The new format allows logs to be forwarded to remote log collectors.
  - Be aware that external servers may also hold logs.
- Registry Paths for Log Configuration
  - **These keys define the location and configuration of event logs:**
    - `HKLM\SYSTEM\CurrentControlSet\Services\EventLog\Application`
    - `HKLM\SYSTEM\CurrentControlSet\Services\EventLog\System`
    - `HKLM\SYSTEM\CurrentControlSet\Services\EventLog\Security`
## .evtx Log format

- 1. Memory Management Overhaul
  - **Old issue:** Event logs were memory-mapped (up to 300 MB) -> high performance cost.
  - **New solution:** Logs use a small header + 64 KB chunks -> only the current chunk is in memory.
  - **Result:** Less memory usage, improved performance, and reduced admin incentive to disable logging.
- **2. File Format:** .EVTX
  - Replaces .evt with .evtx (XML-based).
  - **Benefits:**
    - Easier parsing and log deconstruction.
    - Supports XPath filtering for complex searches.
    - Structured format aids forensic investigation.
- 3. Improved Log Content
  - **Messages now:**
    - Include IP addresses alongside hostnames (also added to Win2003).
    - Use Event IDs with clarified series and message formats.
    - **Result:** Better readability and forensic value.
- 4. Granular Logging
  - System now supports hundreds of specialized logs (not just Application/System/Security).
  - **Advantages:**
    - Better visibility into specific services (e.g., Task Scheduler, Plug and Play).
    - **Supports Advanced Audit Policy Configuration:** Enables selective auditing instead of “all or nothing.”
## Categorization

- Original Logs (NT Era - Present)
  - **Started with 3 core logs:**
    - Security
    - System
    - Application
- These logs have remained vital throughout Windows versions.
- Modern Expansion (Vista+ to Win11)
  - **Introduction of custom logs:**
    - PowerShell, Task Scheduler, Windows Firewall
    - Server-specific logs like DNS, Directory Services, File Replication, etc.
  - Windows 11 now supports over 300+ logs, a 100%+ increase over Windows 7.
  - Logs are segmented, reducing turnover risk and improving long-term retention.
- **Forensic Benefit:** Segmentation and specialization improve the chance of retaining crucial data for longer, especially for logs like Security which traditionally have high turnover.
- Log Type
  - Security Log
    - Events based on auditing policies (e.g., logon attempts).
  - System Log
    - System-level events (e.g., boot failures).
  - Application Log
    - Application-specific alerts or failures (e.g., SQL DB connection loss).
  - Custom Logs
    - Specialized logs (PowerShell, Firewall, Task Scheduler, etc.). Often under "Applications and Services Logs" in Event Viewer.
## Event Types

- **Standard Event Types (found in all logs):**
  - Error
    - **Meaning:** Serious issue (e.g., data loss, service failure).
    - **Use Case:** Prioritize for troubleshooting.
  - Warning
    - **Meaning:** Potential issue (e.g., low disk space).
    - **Use Case:** Monitor to prevent future problems.
  - Information
    - **Meaning:** Successful operation of a service or application.
    - **Use Case:** Baseline behavior logging (e.g., Event Log Service started).
- Security Log-Specific Audit Types
  - **Driven by the configured audit policy, these indicate access attempts:**
  - Success Audit
    - **Meaning:** Successful security event (e.g., user login).
    - **Use Case:** Track valid access.
  - Failure Audit
    - **Meaning:** Failed security event (e.g., unauthorized network drive access).
    - **Use Case:** Detect and investigate potential intrusions or misconfigurations.
## Security Log

- It captures audit events triggered by user/system activity that match defined audit policies, such as:
  - Logons/logoffs
  - Privilege use
  - File access (object auditing)
  - Remote access, runas commands
- Modifications to security settings are also recorded.
- Audit Policy Configuration
  - Policies can be set to trigger on successful, failed, or both outcomes.
  - Fine-tuning allows you to reduce unnecessary logging while capturing what matters.
  - **Examples:**
    - Successful AND failed logons -> detect lateral movement or password attacks.
    - Object access auditing -> track access to protected resources.
- Per-User Auditing
  - Audit policies can be customized per user via Group Policy (e.g., only monitor admin or suspicious accounts).
  - Ideal for identifying targeted compromises.
## Security Event Categories

- **Purpose:** Categories help identify log entries relevant to your investigation quickly.
- They are tied directly to audit policies configured on the system.
- **Each category's audit policy can be set to one of the following:**
  - No Auditing
  - Success
  - Failure
  - Success and Failure
- **Category:**
  - Account Logon
    - Logs who authorized the logon—typically at domain controllers or local systems.
  - Account Mgmt
    - Tracks account maintenance and changes (e.g., password reset, creation).
  - Directory Service
    - Logs access attempts to Active Directory objects.
  - Logon Events
    - Every logon/logoff attempt on the local system (distinct from Account Logon).
  - Object Access
    - Logs access to secured objects (e.g., files, registry keys) listed in the ACL.
  - Policy Change
    - Tracks changes to user rights, audit policies, or trust settings.
  - Privilege Use
    - Logs when a user exercises a privilege, such as SeDebugPrivilege.
  - Process Tracking
    - **Monitors process activity:** start, exit, handle usage, object access, etc.
  - System Events
    - Captures system start/shutdown, changes to the Security log itself, etc.
## Why Event Logs Often Fail Investigations

- 1. Lack of Logging Is Common
  - Most environments have poor logging configurations out of the box. Especially in residential or standalone systems, logging is minimal unless manually configured.
- 2. Event Logs Are Crucial for Investigations
  - Track employee misuse
  - Detect intrusions (e.g., after-hours logons, RDP access)
  - Support incidents like malware, privilege escalation, or data exfiltration
- 3. Poor Retention Policies
  - Even if logging is enabled, logs often aren’t kept long enough to be useful unless intentionally preserved or exported.
- 4. Defaults vs. Recommended Settings
  - The default audit policy is far from comprehensive.
  - Microsoft’s recommended auditing baseline offers much more granular insight.
  - Especially in modern Windows versions, each audit category includes many “advanced settings.”
- 5. Group Policy Overrides Matter
  - In Active Directory environments, Group Policy can enforce better audit policies across workstations and servers—often overriding weak defaults.
- 6. Server Logging Isn’t Always Better
  - There’s a misconception that Windows Servers have strong logging. Like workstations, server logging is also minimal unless configured.
## Tracking user account usage

- Why It Matters
  - Helps confirm when a user logged in and out, supporting other evidence like file access or program execution.
  - Detects credential compromise or unauthorized use by profiling logon behavior.
  - Useful for tracing lateral movement in networks (e.g., RDP, remote shell).
- Key Event IDs (Post-Win2008)
  - 4624 – Successful logon
  - 4634 – Logoff
  - 4647 – User-initiated logoff (interactive sessions)
  - 4625 – Failed logon (e.g., brute force attempts)
  - 4672 – Special privileges assigned (e.g., Admin equivalent logon)
- Event ID Consolidation
  - XP/2003 had 24+ logon-related IDs.
  - Win2008+ condensed these into a few critical ones, making analysis easier but requiring careful interpretation.
- Real-World Considerations
  - Backdoors or exploits may not trigger standard logon events, since they bypass normal APIs.
  - Still, admin account use often leaves traces (e.g., 4672), especially in early or lateral stages of an attack.
- Audit Policy Implication
  - These logon events are triggered by both Success and Failure audits.
  - Ensure your audit policy includes both under the “Logon Events” category to capture complete data.
## Using the Event Viewer for Log Analysis

- The Event Viewer is a built-in Windows utility used to review and analyze event logs. It can be accessed via:
  - **Run command:** `eventvwr.exe`
  - **MMC (Microsoft Management Console):** Right-click My Computer > Manage
- Key Fields in Logon/Logoff Events (e.g., `Event ID 4624`)
- When analyzing logon usage events, particularly `Event ID 4624` (successful logon), focus on the following:
  - **Timestamp:** Time and date when the logon occurred.
  - **Computer name:** Hostname helps correlate logs across systems.
  - **Event ID:** Indicates the type of logon event (e.g., 4624 = successful logon).
  - **Event Description: Includes:**
    - **Account Name:** User account that logged in (e.g., rsydow).
    - **Logon Type:** Indicates how the logon occurred (e.g., `Type 2` = interactive logon via console).
- Always correlate logon events (4624) with their corresponding logoff events (e.g., 4634) for session validation.
- Comprehensive Event Fields (from XML metadata)
- **When expanding events in the Event Viewer, additional details are available:**
  - **Logged:** Local system time when the event occurred.
  - **Level:** Indicates severity or importance (Error, Warning, etc.).
  - **User:** Account responsible for triggering the event.
  - **Computer:** System where the event occurred.
  - **Source:** Application or service that generated the log.
  - **Task Category:** Context of the event (mapped via audit policy).
  - **Event ID:** Unique code tied to the system function (e.g., 4624).
  - **General Description:** Human-readable summary, often includes usernames, IPs, or hostnames.
  - **Details:** Optional field containing raw or error data.
## Logon Events

- **Logon Event Details:**
  - Logon events reveal date, time, username, hostname, and success/failure status.
  - The Logon Type (found in the event's Description) indicates how the user logged on (e.g., locally at console, via RDP, etc.).
  - This helps differentiate between interactive user logons and system-level or automated logons (like scheduled tasks).
- **Common Logon Types:**
  - Interactive
    - Logged on via console or direct interface
  - Network
    - SMB, drive mapping, or RDP with NLA
  - Batch
    - Scheduled tasks
  - Service
    - Windows service login
  - Unlock
    - Reconnect or unlock
  - Cleartext
    - Network logon with cleartext (e.g., downgrade attack)
  - New credentials
    - “RunAs” scenarios
  - Remote interactive
    - RDP/Terminal Services
  - Cached credentials
    - For offline or domain authentication fallback
## Logon ID

- 1. Logon ID Tracks a Unique Session
  - Logon ID is assigned when a user logs in (e.g., `Event ID 4624`).
  - Stays consistent throughout the session.
  - Can be used to match with a logoff event (like 4647 or 4634) to determine the session duration.
  - **For example:**
    - **Logon at 1:**11:38 PM (4624)
    - **Logoff at 8:**12:10 AM (4647)
    - = 19-hour session
- 2. Session Length Analysis
  - **Most useful for interactive logons:** Types 2, 10, 11, 12
  - Less meaningful for Types 3 and 5 (brief connections, e.g., SMB).
  - **Important for investigations involving:**
    - Timecard fraud
    - Privileged access reviews
    - Remote access session analysis
- 3. Linked Logon ID and Additional Activity
  - Windows 10+ introduced Linked Logon ID (e.g., in 4624/4647/4800/4801), which maps related logon events (e.g., unlocking screen).
  - Correlates screen locking/unlocking, privileged vs non-privileged actions, etc.
  - Helpful for reconstructing full user activity.
- 4. Microsoft and Standard Accounts
  - Microsoft cloud accounts (e.g., fred.rocba@outlook.com) can appear alongside standard usernames (e.g., fred).
  - Both are tied to the same session.
  - Filtering and correlation must account for this dual representation.
## Remote Desktop Protocol (RDP)

- `Event ID 4778` – Session Reconnect
  - Indicates an RDP session was successfully reconnected.
  - Triggered when a user re-establishes a previously disconnected session.
  - Does not show brand-new RDP connections, only reconnections.
- `Event ID 4779` – Session Disconnect
  - Triggered when an RDP session disconnects.
  - Does not necessarily mean the user logged off — only that the session was interrupted (manually or due to network drop).
- Advantages of 4778/4779 Over Standard Logon (4624)
  - Unlike standard logons (`Event ID 4624`, especially Logon `Type 10` for RDP), 4778/4779 give insight into mid-session activity like reconnects.
  - **Include valuable data such as:**
    - IP address of the client
    - Hostname of the connecting system
  - Useful for detecting persistent lateral mov
- Practical Notes
  - Console sessions must be logged out before RDP sessions can initiate.
  - Logon events via RDP typically show Logon `Type 10`.
  - “Fast User Switching” can rename session names (e.g., from RDP to “Console”).
## Microsoft Office

- `OAlerts.evtx` – Microsoft Office Event Log Summary
  - Purpose and Triggering Condition
    - `OAlerts.evtx` logs Office dialog alerts shown to users.
    - **Most common trigger:** attempting to close an Office document with unsaved changes.
    - **Captures:**
      - File name
      - Application name
      - Duration the file was open
      - Modification actions, such as edits and deletes
  - Event Details
    - All Office apps use `Event ID 300`.
    - Includes the application name in the description.
    - Triggers regardless of file location (local, USB, network share).
    - Also logs file access and permission issues.
## System time changes

- **Anti-Forensic Technique:** Time Backdating
  - **Tactic:** Changing the system clock to falsify file timestamps (e.g., post-dating a signed document).
  - **How it's done:** Copy the document to a new volume, change the system time, edit/save the file.
  - Windows 10+ restricts this action to administrators only (via "Change the system time" rights).
  - This action triggers timestamp changes across files, leading to forensic inconsistencies.
- Detection and EDR Alerts
  - **Detection: Can be identified through:**
    - Anomalous metadata changes
    - System event log anomalies
    - Tools like EDR (Endpoint Detection and Response)
  - Setting the clock backward can evade EDR detection unless timestamp behavior is monitored.
- System Time Logging – Key Points
  - Normal time adjustments are expected (e.g., via NTP updates) and are usually logged with:
    - `Event ID 1` in the System Log
    - NTP changes are performed by SYSTEM or LOCAL SYSTEM accounts
  - User-initiated changes are tied to the user account, especially relevant in Kerberos-based networks (authentication relies on time).
## USB/removable device-related

- Plug and Play (PnP) Device Events
  - `Event ID 20001` (System Log)
    - Triggered when a driver is installed for a Plug and Play device.
    - **Logs:**
      - Device vendor name
      - Device name
      - iSerialNumber (if exists)
  - `Event ID 20003` (System Log)
    - Recorded at the completion of device installation.
    - **NOTE:** Recent Windows 10/11 versions do not store user account info by default for this event, requiring correlation with logon events or registry parsing.
  - **Applies to a variety of devices:** USB, MTP, FireWire, PCI, PCMCIA, Thunderbolt, etc.
- Removable Storage Audit Events (Windows 8+)
  - `Event ID 4663` (Security Log)
    - Marks removable storage usage if Audit Removable Storage is enabled via Advanced Audit Policy Configuration.
    - Often used to detect BYOD (Bring Your Own Device) scenarios.
  - `Event ID 4656` (Security Log)
    - Triggered by a failed attempt to access a removable device.
  - `Event ID 6416` (Security Log)
    - Added in Windows 10
    - Logs detailed Plug and Play tracking every time a device is connected.
## Wireless network Geo

- Key Event IDs
- **`Event ID 8001`:**
  - Successful connection to a wireless network.
  - Records SSID and BSSID (MAC address of the access point).
  - Useful for identifying the wireless network and potentially geolocating its physical location.
- **`Event ID 8002`:**
  - Indicates a failed connection attempt to a wireless network.
- **`Event ID 8003`:**
  - Used with 8001 to determine session duration on the network.
- **`Event ID 11005`:**
  - Wireless security succeeded.
- **`Event ID 11004`:**
  - Wireless security stopped (e.g., after sleep mode).
- These can help trace automatic re-connections to previously used Wi-Fi networks.
- This log maintains a detailed record of each connection attempt.
- **Useful for:**
  - Identifying suspicious Wi-Fi activity
  - Investigating data exfiltration
  - Confirming physical device presence (based on Wi-Fi use)
  - Reconstructing historical access patterns
  - Even if Windows Registry is altered, this log persists independently and can retain data going back months.
## Windows Event IDs

- Why You Don’t Need to Memorize Event IDs
  - **There are too many:** Thousands of Event IDs with varying meanings across Windows versions and components.
  - **Descriptions vary:** Especially for Application and System logs, which are often poorly documented or vague.
- Recommended Online Resources
  - Ultimate Windows Security
    - Now rebranded as Ultimate IT Security.
    - Considered “THE” go-to site for researching Security Event IDs.
    - Regularly updated and crowdsourced.
    - Highly recommended for quick lookups and unfamiliar events.
  - Microsoft’s Official Documentation
    - **Useful for Application/System log IDs, which are often:**
      - Overlooked or under-documented elsewhere.
      - Accompanied by cryptic or repetitive descriptions.
  - **Recently, Microsoft released the:**
    - Windows 10 Security Auditing and Monitoring Reference
      - Very detailed auditing guide.
      - Highly recommended to include in your digital forensic reference library.
