# Registry Forensics

## Overview

- The registry is a collection of database files that store vital system configuration data.
- It records settings for the operating system, installed applications, hardware, and user accounts.
- `Stored in C:\Windows\System32\Config`

## Main Registry Hives:

- `SAM: Stores local user and group account information.`
- `SECURITY: Contains security settings, audit policies, and security identifiers (SIDs).`
- `SYSTEM: Manages hardware configurations and system services.`
- `SOFTWARE: Holds data on installed applications and Windows OS settings.`
- `DEFAULT: Stores default environment variables, though it’s often overridden by user-specific settings in the NTUSER hive.`

## User Hives

- User Registry Hives:
  - Each user profile has its own registry hives storing personal settings, app configurations, and user-specific data (e.g., file interactions, cloud storage, and application usage).
- Primary User Hive – NTUSER.DAT:
  - `Located in C:\Users<username>\NTUSER.DAT.`
  - Contains most user-related settings and data.
  - Previously stored under Documents and Settings in Windows XP (now deprecated).
- Secondary User Hive – UsrClass.dat:
  - `Found in C:\Users<username>\AppData\Local\Microsoft\Windows\UsrClass.dat.`
  - Introduced in Windows Vista to support User Account Control (UAC) and prevent applications from requiring admin-level access to write to the main registry.
  - Stores forensic artifacts such as ShellBags (folder interaction data) and MuiCache (application execution history).

## Live and Offline Hive

- Registry hives on a live system (viewed with regedit.exe) differ from offline registry files (NTUSER.DAT, SAM, etc.).
- Live Registry Root Keys and Their Mappings:
  - `HKEY_LOCAL_MACHINE (HKLM) → Corresponds to SAM, SECURITY, SYSTEM, and SOFTWARE hives`
  - `HKEY_CURRENT_USER (HKCU) → Represents NTUSER.DAT and UsrClass.dat (for the currently logged-in user).`
  - `HKEY_CLASSES_ROOT (HKCR) → A volatile hive combining data from HKLM and HKCU, only present in memory`
  - `HKEY_USERS (HKU) → Contains user-specific registry settings for all logged-in users.`

## Keys and Values

- Registry hives consist of keys (similar to folders) and values (similar to files).
- Values store data in various formats, including strings, binary (hex), integers, and lists
- Malware and Persistence Abuse
  - Malware has increasingly leveraged registry values to store encoded scripts or binaries, helping evade traditional security measures
  - This technique enables persistence across reboots, as registry data remains intact even after system restarts.
  - Poweliks malware is an example that used this method to remain fileless and avoid detection.

## Registry Timestamps and Forensic Value

- Registry keys and subkeys in Windows store timestamps representing the last time they were modified (Last Write Time).
- These timestamps are stored as 64-bit FILETIME values, but are not visible in regedit.exe, requiring forensic tools for retrieval.
- Changes occur when a value is added, modified, or deleted within a registry key or subkey.
- Example: Finding when a sensitive file search occurred and cross-referencing with logged-in users, file transfers, or external device usage.
- Not all registry components store timestamps—only keys and subkeys do, while individual values do not.

## MRU (Most Recently Used)

- MRU lists track the most recent changes to a registry key, logging the order in which data was accessed or modified.
- Windows relies on MRU lists for dropdown menus, search dialogs, Office applications, and autocomplete features.
- How MRUListEx Works
  - MRUListEx is a common format used in modern Windows systems, storing a list of 4-byte values that reference accessed items.
  - Values are stored in little-endian format and must be converted from hex to decimal to determine their sequence.
  - Example
    - If the RecentDocs key is under investigation, its MRU list reveals the order in which .docx files were accessed.
    - Value #0 corresponds to the most recently opened document, followed by Value #16, then Value #18, etc.

## Deleted Registry Keys

- Like file systems, registry hives have allocated and unallocated areas.
- Deleted registry keys are marked as unallocated, making them recoverable using forensic tools.
- Deletions may occur due to:
  - Privacy cleaners (e.g., deleting browser history).
  - Uninstalling programs or system maintenance.
  - Manual anti-forensic actions to hide activity.
- Specialized tools can identify and extract deleted registry keys, such as:
  - Eric Zimmerman's Registry Explorer
  - TZWorks YARU
  - Arsenal Recon's Registry Recon
- Clues that indicate registry tampering:
  - Missing critical keys (e.g., OpenSavePidMRU, LastVisitedPidMRU).
  - Unusual software activity (traces in Prefetch, UserAssist, or event logs).
  - Tools capable of detecting unallocated registry keys reveal deleted evidence.

## Hive Transaction Logs

- Registry changes are cached in two locations before being permanently written:
  - `System memory`
  - Transaction log files (<HIVENAME>.LOG1 and <HIVENAME>.LOG2)
- When a registry key or value is modified, it first updates in memory.
- Later, a hive flush writes the changes from transaction logs to the main registry hive.
- Windows 8 and later, registry changes stay in transaction logs longer before being written to the primary hive.
- The flushing process:
  - Occurs only during system idle time or shutdown.
  - Takes approximately an hour if the system remains active.
- When collecting forensic data, always acquire both registry hives and their transaction logs to ensure no recent modifications are overlooked.

## User/Group Info Analysis

- Key information to analyze includes:
  - Login frequency and last login time
  - Failed login attempts
  - Group memberships
- Relative Identifier (RID) and the SAM Hive
  - Many forensic artifacts (e.g., Recycle Bin, Event Logs) use a user’s RID instead of their username
  - The SAM hive provides an easy way to map RIDs to local user accounts.
  - Investigating the SAM hive can uncover:
    - Previously overlooked accounts
    - Use of built-in administrator accounts, which may indicate unauthorized activity.
- Account Profiling
  - The last login date helps determine if an account is relevant to the investigation.
  - The number of logins shows account usage trends (e.g., frequent use vs. rare logins).
  - Unusual login patterns (e.g., only logging in at odd hours) can indicate suspicious behavior.
  - Excessive failed logins may suggest brute-force attacks or unauthorized access attempts.
- Cloud Account Activity
  - Microsoft cloud accounts are tied to email addresses and enable cross-device synchronization for browsers and Windows settings
  - Cloud account usage is recorded in the SAM, but logon count values are not updated for these accounts (reason unknown).
- Domain Accounts vs. Local Accounts
  - The SAM hive provides extensive details about local accounts, but domain accounts (common in enterprises) are primarily managed by network domain controllers.
  - Domain accounts authenticate across multiple systems, making them important for tracking potential abuse, particularly in cases of computer intrusions.
  - Although domain account details reside on a domain controller, endpoint systems keep a ProfileList registry key, which:
    - Logs both local and domain accounts that have interactively logged into the system
    - Indicates that a profile was created and that the account had a GUI desktop session.
    - Helps investigators identify unauthorized domain account usage on systems where they should not be present.


## System Configuration

- OS Version
  - `SOFTWARE\Microsoft\Windows NT\CurrentVersion`
  - Stores information about the latest Windows update, including:
    - OS version and type
    - Build number
    - Date/time of the last update
  - `SYSTEM\Setup\Source OS`
    - Tracks major Windows updates from Windows 7 onwar
  - If no major updates occurred, this represents the original installation date.
- Account for potential time discrepancies due to Windows' update process

## Current Control Sets

- Control sets store system configuration settings (drivers, services) essential for booting Windows.
- Modern Windows versions (post-Windows 7) usually have only one control set (ControlSet001), possibly due to improved system stability
- Identifying the Active Control Set
  - `SYSTEM\Select holds a value named Current, which tracks the control set in use.`
  - On a live system, CurrentControlSet is a volatile key pointing to the active control set
- The LastKnownGood value identifies a backup control set used for system recovery if a crash occurs.

## Computer Name

- Many Windows artifacts tag events using the hostname rather than an IP address
- Verifying the computer name early in an investigation ensures:
  - You are examining the correct system.
  - You stay within authorization limits (especially in restricted environments).
  - You don’t waste time investigating the wrong machine
- `SYSTEM\<CurrentControlSet>\Control\ComputerName\ComputerName`

## System Time Zone

- Most Windows timestamps (e.g., NTFS timestamps, registry key last write times, event logs) are stored in UTC format.
- Using UTC simplifies forensic analysis by:
  - Ensuring consistent timestamps across systems in different time zones.
  - Avoiding confusion with daylight saving time changes.
  - Preventing misinterpretation when analyzing events across multiple systems.
- TimeZoneInformation
  - The TimeZoneInformation registry key tracks the last known system time zone setting.
  - `System Event Log (Event ID 1) can sometimes help track past time zone change`
- Avoiding Time Zone Conversion Errors
  - Some forensic tools automatically convert timestamps to the local system time zone, which can lead to misinterpretation.
- Best Practice:
  - Set the analysis machine time zone to UTC.
  - This prevents accidental bias adjustments and ensures accurate forensic reporting.
  - You can convert timestamps to local time later in the final report if needed.

## Network Interface

- Tracking network activity helps determine:
  - Where a device has been used.
  - Whether it connected to suspicious networks (e.g., rogue access points).
  - VPN usage, indicating possible obfuscation of location.
  - Unexpected geographic locations, suggesting unauthorized device movement.
- `SYSTEM\<CurrentControlSet>\Services\Tcpip\Parameters\Interfaces`
  - IP addresses (private and sometimes public).
  - DHCP settings.
  - Last known domain connections.
  - Geolocation Clues:
    - Domain names in IP settings can reveal ISP, region, or company ownership.
    - Example: hsdl.ut.comcast.net suggests a home/small business connection in Utah.
- `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkCards`
  - Physical vs. Virtual adapters (VPNs, virtual machines).
  - GUIDs (Globally Unique Identifiers) for network interface

## Historical Network Connections

- NLA allows Windows to classify networks as public or private
- Different networks receive specific firewall policies, ensuring secure connectivity.
- `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures`
- Profiles are categorized as:
  - Managed Networks → Typically corporate or enterprise environments
  - Unmanaged Networks → Includes public networks (e.g., coffee shops, hotels, home Wi-Fi).
- Stored network information includes:
  - DNS suffix (useful for identifying corporate domains).
  - SSID (identifies wireless networks).
  - Gateway MAC Address (crucial for geolocation and tracking network access points).
- ProfileGUID:
  - Each network is assigned a Globally Unique Identifier (GUID).
  - This links network data across different registry keys for deeper investigation.

## Network Profiles

- Windows remembers previously connected networks to allow automatic reconnection
- Wireless Zero Configuration (XP) or WLAN AutoConfig (modern Windows), stores a historical record of network connections.
- `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profile`
- NameType Values (Network Connection Type)
  - 6 (0x06): Wired (Ethernet) (802.1x authentication)
  - 23 (0x17): VPN
  - 53 (0x35): VPN
  - 71 (0x47): Wireless (Wi-Fi)
  - 243 (0xF3): Mobile Broadband (Cellular Network)
- Category Values (Network Profile Type)
  - 0 = Public network
  - 1 = Private network (Home)
  - 2 = Domain (Corporate Network

## GeoLocation

- Wireless access points (WAPs) and signal strength allow for precise geolocation, often comparable to GPS.
- Devices constantly scan and connect to known networks, leaving trails in registry entries.
- Companies like Apple & Google maintain private databases mapping wireless access points, but Wigle.net provides an open-source alternative.

## Installed Apps

- Windows stores detailed information on installed applications across multiple registry locations. The Uninstall keys are the best starting point for forensic audits, storing data on:
  - Application Name
  - Version
  - `Software Publisher`
  - File Size
  - Install Date
  - Location on Disk
- `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`
- `SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall  (for 32-bit apps on 64-bit systems)`
- `NTUSER\Software\Microsoft\Windows\CurrentVersion\Uninstall`
- `NTUSER\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall`
- InstallProperties and MSI Information
  - The InstallProperties key contains additional metadata, often duplicating information from Uninstall keys.
  - It is mainly for Microsoft Software Installer (.msi) packages.
  - MSI installations include a GUID (a unique identifier for tracking software versions).
  - Example uninstall string:
    - `MsiExec.exe /I{9BE0AC23-0551-4755-94A3-F4D377E3CF16}`
- Microsoft Store (UWP) Applications
  - Universal Windows Platform (UWP) apps (Microsoft Store apps) are spread across multiple registry locations.
  - `SOFTWARE\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore`
  - Even after an application is uninstalled, traces often remain in the registry. Forensic searches should include:
    - Most Recently Used (MRU) Lists (records of recently opened apps).
    - Configuration settings tied to specific applications.
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths`
    - `NTUSER\Software\Microsoft\Windows\CurrentVersion\App Paths`
    - `NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts`
    - `NTUSER\Software\Microsoft\IntelliType Pro\AppSpecific`
- InstallDate Value
  - Tracks the last update date for the application
  - Not all applications store InstallDate
    - If InstallDate is missing, the last write time of the subkey can sometimes estimate installation time.
      - `System updates, patches, and major registry modifications can overwrite timestamps and create inconsistencies.`

## Capability Access Manager

- Windows 11 and Windows 10 (Build 1903 and later) introduced keys auditing the use of microphones, cameras, and location data.
- This feature helps users and administrators determine which applications have accessed these resources, making it useful for security and forensic investigations.
- `SOFTWARE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore`
- `NTUSER\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore`
- `Security Implications`
  - Malware or unauthorized applications using a microphone or camera can be detected through this key.

## AutoStart Program

- Windows has many autorun locations, making it challenging to secure against malware.
- `NTUSER\Software\Microsoft\Windows\CurrentVersion\Run`
- `NTUSER\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
- Execution Timing: Items in these keys run when a user logs in, not during boot.
- Services run in the background without user interaction, making them ideal for stealthy attacks.
- Services and device driver configurations are tracked in:
  - `SYSTEM\CurrentControlSet\Services`
- Start Values and Persistence:
  - 0x02 = Automatic (runs at boot)
  - 0x00 = Boot start (for device drivers)
  - Malware often abuses these settings to maintain persistence.

## Malware Hunt Registry

- Malware often abuses the Windows registry to persist long-term on a system and evade detection.
- Indicators of Compromise (IoCs):
  - Strange registry key names (randomly generated keys and values).
  - Unusual service names pointing to suspicious file locations.
  - Run and Service keys are commonly exploited for malware persistence.
- Advanced Evasion Tactics:
  - Attackers now use Microsoft utilities (e.g., regsvr32.exe) to avoid detection.
  - Malware is increasingly encoded into registry keys using base64 scripts and executable code to avoid being stored on disk.
- Recent malware samples:
  - Emotet: Uses randomly named services (e.g., "1A345B7").
  - Qakbot: Uses RunOnce keys to execute malicious DLLs using regsvr32.exe.

## Shutdown Information

- Knowing the last shutdown time helps identify user behavior and detect system anomalies.
- Some artifacts (e.g., Shimcache/Application Compatibility Cache) are only written to disk during a shutdown.
- If the system hasn’t shut down recently, critical evidence might be missing.
- Knowing how long the system has been running helps estimate the retention window for memory-resident data.
- `SYSTEM\<CurrentControlSet>\Control\Windows`
- For Windows XP (also includes shutdown count):
  - `SYSTEM\<CurrentControlSet>\Control\Watchdog\Display`
