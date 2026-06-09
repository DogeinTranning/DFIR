# USB Device Forensics

> Converted from `USBDevice.txt`.


## Key Data to Collect

- Removable Device Information
  - Vendor/Make/Model/Type
  - Unique Serial Number(s)
  - Volume Name (assigned by the user or system)
  - Drive Capacity
- User Interaction Information
  - Drive Letter assigned during connection
  - User Accounts that accessed the device
  - First/Last Connected and Last Removal timestamps

## Registry Hives

- SYSTEM
- SOFTWARE
- NTUSER.DAT
- Used to recover:
  - Device install details
  - Mount points
  - Assigned drive letters
  - User interaction traces


## Event Logs

- Partition\Diagnostic Logs
- System Logs
- Security Logs
- Used to track:
  - Device connection and disconnection events
  - Drive errors
  - Authorization and access attempts


## Shell Items

- LNK Files
- Jump Lists
- ShellBags
- Reveal:
  - Files accessed or opened from the USB device
  - Folder navigation history
  - Timeline of interaction


## Three Major USB Classes

- HID (Human Interface Device)
  - Devices used for user interaction
    - USB keyboards, mice, Bluetooth HID devices, headsets
  - indicate keystroke capture or other malicious input devices.
- MTP (Media Transfer Protocol)
  - Specialized media transfer devices
    - Mobile phones (e.g., Pixel 4a), cameras
  - behave differently from mass storage and may need special handling to extract file access logs.
- MSC (Mass Storage Class)
  - Standard file storage devices
  - Flash drives, external hard drives (e.g., Toshiba, SanDisk)
  - most common focus — removable drives used for:
    - Data theft
    - Malware introduction
    - File transfer without network detection

## Human Interface Device

- While HIDs may not be as data-rich as storage devices, they are highly relevant in suspicious activity cases:
  - Malicious HID attacks (e.g., Hak5 Rubber Ducky) emulate keyboards to execute scripts silently (PowerShell, cmd).
  - HID keyloggers like AirDrive can capture input covertly.
  - Devices like USB Ninja embed wireless HID functions in charging cables.
- These devices often appear right before or during compromise, and their use must be tracked.
- `HKLM\`SYSTEM\CurrentControlSet\`Enum\HID```
- For each HID device, you can typically find:
- Vendor ID (VID) and Product ID (PID)
  - Example: `VID_03EB` → Atmel Corp (often used by malicious HID tools)
- Device Timestamps:
  - Tracked under the subkey:
    - <SerialNumber>\<ParentIdPrefix>\Properties\{83da6326-97a6-4088-9453-a1923f573b29}
  - With subkeys:
    - 0064 → First Connected
    - 0066 → Last Connected
    - 0067 → Last Removal
- These are 64-bit FILETIME timestamps (same as MSC/MTP devices).
- HID devices may leave limited data, but that data can be critical.
- Always investigate newly connected HID devices around incident windows.
- Use VID/PID lookup + timestamp correlation to identify possible malicious input devices.

## MTP & PTP

- MTP (Media Transfer Protocol) and PTP (Picture Transfer Protocol) are used by:
  - Smartphones (e.g., Android, iPhones)
  - Tablets, scanners, cameras
  - Music players
- They behave differently from typical USB mass storage devices (MSC), and leave fewer registry artifacts.
- PTP:
  - Older, used primarily for image and video transfer
  - Read-only – can't write files to device from PC
  - Still used by some Android devices for security reasons
- MTP:
  - Developed by Microsoft to expand beyond images
  - Allows transfer of all file types
  - Read/write access shared between OS and device, but no direct drive letter (unlike MSC)
  - Data shows under “Devices and Drives” or “Portable Devices” (no E:\ or F:\ mapping)
- MTP/PTP devices:
  - Do not mount with a volume like flash drives
  - Often require user interaction for access (e.g., unlocking phone)
  - Leave behind fewer artifacts in Windows
- `SYSTEM\CurrentControlSet\`Enum\USB``
- `SOFTWARE\Microsoft\Windows` Portable Devices\Devices
- File interaction with MTP/PTP may not generate .lnk files unless:
  - A folder was accessed (files alone may not trigger LNKs)
  - The OS version supports the file type (e.g., DOCX and JPG = yes; TXT, XLS = maybe not)
- Still, these devices often appear in ShellBags and Jump Lists
  - My Computer\Pixel 4a\Internal shared storage\DCIM\Camera
  - My Computer\Apple iPhone\Internal Storage\DCIM\202201__

## Mass Storage Class

- MSC is the USB transfer protocol used when a device’s storage is mounted as read/write media, just like an external hard drive or USB stick.
- Mounted directly by Windows, allowing full filesystem access
- Appears under:
  - "Devices and drives" in Windows 10+
  - "Devices with Removable Storage" in older versions
  - Assigned a drive letter (e.g., E:, F:)
- Forensic Value
  - MSC devices:
    - Are fully mounted and browsable
    - Leave behind many forensic artifacts, including:
      - ShellBags
      - LNK files
      - Jump Lists
      - Mount point logs
  - Investigators treat them like internal volumes because:
    - Every accessed file/folder is visible
    - File timestamps, metadata, and registry entries are rich
- USB MSC Transfer Protocols: BOT vs UASP
  - BOT – Bulk-Only Transport
    - Traditional protocol used for mass storage transfers
    - Managed under the Windows registry key:
      - `SYSTEM\CurrentControlSet\`Enum\USBSTOR``
    - Still widely used by most USB 2.0/3.0 flash drives
  - UASP – USB Attached SCSI Protocol
    - Introduced with USB 3.0 to support:
      - Faster, multi-threaded transfers
      - Solid-state drives (SSDs) and Thunderbolt/USB 3.x/4.0 devices
    - Uses SCSI commands, not tracked under USBSTOR
    - Instead, forensic traces are found under:
      - `SYSTEM\CurrentControlSet\`Enum\SCSI``

## USB MSC Forensic Audit Checklist

- Step-by-Step USB MSC Analysis Tasks
1. Audit USB Devices and Types
  - → Identify all connected MSC, MTP, HID devices.
2. Document Vendor ID (VID) and Product ID (PID)
  - → Found in USBSTOR or SCSI registry keys.
3. Document Device iSerialNumber
  - → Unique serial to identify repeat connections.
4. Determine Friendly Name, Vendor, Product, Version
  - → Helps associate make/model of device.
5. Document First and Last Time Connected
  - → Timestamps from Windows registry properties.
6. Document Last Time Device Removed
  - → Useful for timeline correlation with user activity.
7. Determine Volume Name
  - → Typically visible in Explorer (e.g., "KINGSTON USB").
8. Find Last Mountpoint Drive Letter
  - → E.g., E:\, F:\, retrieved from mount logs/registry.
9. Document Volume GUID (USBSTOR only)
  - → Used to track volumes even if the drive letter changes.
10. Identify Related User Accounts (USBSTOR only)
  - → Who accessed or mounted the device.
11. Determine Volume Serial Number
  - → Allows correlation with file system entries.


## Audit USB Devices and Types

- Identify all USB devices previously plugged into a system.
- USB Device Classes Auditable
  - MSC → Mass Storage Class (USBSTOR or SCSI)
  - UASP → USB Attached SCSI Protocol
  - MTP / PTP → Media & Picture Transfer Protocol (phones, cameras)
  - HID → Human Interface Device (keyboards, mice, etc.)
- Registry Key Location
  - `SYSTEM\CurrentControlSet\`Enum\USB``
- Provides:
  - Device Type (USBSTOR, HID, MTP, etc.)
  - Vendor ID (VID)
  - Product ID (PID)
  - iSerialNumber
  - ParentIdPrefix (used especially for UASP devices)
- On Windows 10/11, this data may be reset after major OS updates

1. Vendor ID (VID) & Product ID (PID)
  - Found in the subkey name (e.g., `VID_XXXX`&`PID_YYYY`)
  - Identify manufacturer and model
  - Cross-reference with online USB VID/PID databases
2. iSerialNumber
  - Found under each VID_XXX&PID_YYY key
  - Unique per device instance
  - Use to track a device across multiple registry hives or machines
  - Caveat: Some devices spoof serials or use the same serial across multiple units
3. Service & DeviceDesc Fields
  - Located in the iSerialNumber subkey
  - Tell you what type of device it is
  - Helps differentiate between hubs, drives, webcams, etc

## VID & PID

- VID (Vendor ID): Identifies the manufacturer
- PID (Product ID): Identifies the specific device model
- Assigned by the USB Implementers Forum (usb.org)
- These IDs are found in the Windows Registry under:
  - `HKEY_LOCAL_MACHINE\`SYSTEM\CurrentControlSet\`Enum\USB```
- Lookup Tools
  - USB IDs Repository: Frequently updated database
  - DeviceHunt.com: Graphical front-end for USB IDs and PCI IDs
- Special Case: USB Adaptors
  - If you’re analyzing a USB adapter, the attached USB device’s iSerialNumber, volume name, and other metadata are still captured.
  - So the profiling process remains the same, even when a device is connected via an adapter.

## Profiling USBSTOR Devices

- Registry Path:
  - `HKEY_LOCAL_MACHINE\`SYSTEM\CurrentControlSet\`Enum\USBSTOR```
- This key stores detailed records of MSC (mass storage class) USB devices like flash drives and external hard drives
- How to Identify the Device
  1. Locate the device in:
    - `SYSTEM\CurrentControlSet\`Enum\USB``
  2. Use the iSerialNumber found there to match against:
    - `SYSTEM\CurrentControlSet\`Enum\USBSTOR\<DeviceID>\<iSerialNumber>``
- DiskId and Correlation with Event Logs
  - Each USBSTOR device includes a DiskId located at:
    - ...USBSTOR\<DeviceID>\<SerialNumber>\Device Parameters\Partmgr
  - You can correlate this DiskId with:
    - `Microsoft-Windows-Partition/`Diagnostic.evtx``
  - → to find volume-level metadata and activity logs.
- Information Available in USBSTOR
  - Device ID: Appears in the subkey name
  - FriendlyName: Human-readable name
  - First Time Connected
  - Last Time Connected
  - Last Removal Time
- These timestamps are critical for building a timeline of USB usage.


## USB device timestamps

1. First Time Device Connected
  - When the USB device was first plugged into the system
  - Added in Windows 7
  - Useful for identifying the initial presence of a new device
2. Last Time Device Connected
  - When the device was most recently reconnected
  - Added in Windows 8+
  - Helps correlate suspicious activity windows
3. Last Removal Time
  - When the device was last unplugged or disconnected
  - Also added in Windows 8+
- USB Timestamp Profiling – Key Concepts
  - When auditing USB devices (especially under USBSTOR), forensic analysts extract timestamps from the Properties subkey within each iSerialNumber entry.
  - Registry Path Format:
    - `Enum\USBSTOR\<DeviceID>\<iSerialNumber>\Properties\{83da6326-97a6-4088-9453-a1923f573b29}`
- Three Critical Timestamp Subkeys

- Each key stores an 8-byte Windows FILETIME value and maps to a device activity event:
  - Subkey
  - 0064
    - First Install Date (initial connection)    Windows 7+
  - 0066
    - Last Connected Date    Windows 8+
  - 0067
    - Last Removal Date (ejected or unplugged)    Windows 8+
  - These are considered standard markers for profiling USB activity.
  - Note on 0065
    - Contains a valid timestamp, usually identical to 0064
    - Represents last driver install date
    - If different, may indicate a reinstallation or updated driver – rarely used in most investigations
- These subkeys and timestamping techniques are valid across:
  - USBSTOR (MSC)
  - `Enum\USB`
  - `Enum\SCSI` (UASP)
  - `Enum\HID` (input devices)
  - UASP and SCSI entries use ParentIdPrefix instead of iSerialNumber as the <Identifier>.

## First connected USB

- The primary and more modern method uses the registry key:
  - `SYSTEM\CurrentControlSet\`Enum\USBSTOR\<DeviceID>\<iSerialNumber>\Properties\{83da6326-97a6-4088-9453-a1923f573b29}\0064``
- The 0064 subkey holds a hexadecimal FILETIME timestamp (e.g., `24-CA-94-B6-56-DF-D7-01`)
- This converts to a First Connected Time, such as:
  - 2021-11-22 04:09:15 UTC
- Legacy Method – Setup Log (Pre-Win8)
  - Before this key was standardized, forensic analysts used the log file:
    - `C:\Windows\INF\`setupapi.dev.log`` (or `setupapi.log` in Windows XP)
  - This log records:
    - Hotfix installs
    - Driver installations
    - USB plug and play events
    - Analysts search for the device’s iSerialNumber to find a nearby timestamp
  - Caution:
    - Log times are in local system time
    - Registry timestamps use UTC
    - Small discrepancies may exist due to install process timestamping


## Last Connected Time

- Key Registry Path for Last Connected Time
  - `SYSTEM\CurrentControlSet\`Enum\USBSTOR\<Device`` ID>\<iSerialNumber>\Properties\{83da6326-97a6-4088-9453-a1923f573b29}\0066
- The key 0066 holds a Windows FILETIME timestamp representing the last connection time.
- In the example shown:
  - Hex value: `86-2C-11-C2-11-0B-D8-01`
  - Human-readable time: 2022-01-16 19:46:30 UTC
- This key is not present on Windows 7 or earlier.
- Timestamp is stored in UTC, and tools like Registry Explorer will automatically convert it for you.
- This value is updated every time the device is connected.


## Last Removal Time

- Registry Path for Last Removal Time
  - `SYSTEM\CurrentControlSet\`Enum\USBSTOR\<Device`` ID>\<iSerialNumber>\Properties\{83da6326-97a6-4088-9453-a1923f573b29}\0067
- The 0067 key contains the removal time, even if it was a safe eject or just unplugged.
- In the example:
  - Hex value: `F6-F2-E1-D9-1A-0B-D8-01`
  - Converted Time: 2022-01-16 20:51:35 UTC


## Determine Volume Name

- Volume Name from the SOFTWARE Hive
- Location:
  - `SOFTWARE\Microsoft\Windows` Portable Devices\Devices
- Each sub-key is named according to the Device ID (e.g., Disk&Ven_HP&Prod_v100w&Rev_1024) and stores:
  - FriendlyName: The last known volume name assigned to the device (or the last drive letter if no name exists).
- Why It Matters:
  - Volume names help correlate devices across registry hives like `Enum\USB` and `Enum\USBSTOR.`
  - This key helps track USB devices with the same device ID and iSerialNumber, even if seen at different times or after OS updates.
  - Persistence: This SOFTWARE key is less likely to be wiped during OS updates, unlike USBSTOR or USB keys.


## Last Mountpoint Drive

- Volume name and drive letter can link USB devices to:
  - LNK files
  - Prefetch
  - RecentDocs
  - OpenSavePidlMRU
  - ShellBags
  - Jump Lists
- These links help reconstruct user activity involving a specific removable device.
- Drive letter data:
  - Only reflects the last device connected with that letter.
  - A single letter might be reused across multiple devices.
- This limits reliability for precise device tracking over time.
- VolumeInfoCache
  - Location:
    - `SOFTWARE\Microsoft\Windows` Search\VolumeInfoCache
  - Purpose:
    - Stores mappings of drive letters (C: to H:) to their last connected volume label.
  - Key Field:
    - VolumeLabel – shows the name of the last volume connected to each drive letter.

- Registry Key: `SYSTEM\MountedDevices`
  - Used For: Identifying the last drive letter (mount point) assigned to a device.
  - Best For: MSC devices (like USB thumb drives profiled under USBSTOR
  - How it works:
    - Each drive letter (e.g., \DosDevices\E:) has a binary value containing data including the device’s iSerialNumber.
    - Search for the iSerialNumber within these binary values.
    - If matched, you can conclude that the last drive letter used by that device was E: in this example.

## Identify Related User Accounts

- Determine which user account was logged in when a USB device was used by:
  - Cross-referencing `SYSTEM\MountedDevices` with
  - Each user’s `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2.`
- Key Concepts:
  - NTUSER.DAT
    - Loaded per user session; contains user-specific registry data
  - Volume GUID
    - Acts as the identifier that links USB use between SYSTEM and NTUSER hives
  - MountPoints2
    - Stores mount info (drive letters, share paths) for devices seen in File Explorer
- MountPoints2
  - `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2`


## Diagnostic.evtx

- Log Name:
  - `Microsoft-Windows-Partition/`Diagnostic.evtx``
- `Event ID 1006`:
  - Triggered each time a MSC device is connected/disconnected.

### 📋 Information Captured per Event:

  - Model & Manufacturer
  - Vendor ID (VID) & Product ID (PID)
  - Disk capacity
  - Two Serial Numbers:
    - SCSI Serial Number
    - iSerialNumber (from the ParentIdPrefix, more important for correlation)
  - MBR and up to 3 Volume Boot Records (VBRs) – helps identify partition structure
- Connect Event: Capacity field shows disk size
- Disconnect Event: Capacity is 0
- Detects:
  - Normal insert/removal
  - Sleep/hibernation plug-ins (may cause back-to-back connection logs)

## Volume Serial Numbers

- Purpose: VSNs are used to correlate external devices with accessed files or folders on a system.
- Source of VSNs: VSNs are embedded in FAT, exFAT, and NTFS partition boot records.
- Matching Technique: Investigators can match VSNs from LNK or Jump List shell items to identify file/folder access from removable media.
- Value: Helps prove that a specific external device was used to open or access particular files on a system.
- Where to Find VSN Data Without the Device
  - `Microsoft-Windows-Partition/`Diagnostic.evtx``: Logs detailed volume and partition data, including VSNs, partition types, and boot records.
  - `SOFTWARE\Microsoft\Windows` NT\CurrentVersion\EMDMgmt: Registry key that stores metadata about USB media like iSerialNumber and Volume Name. However, it's legacy and not available on newer systems (post-Vista).
  - Data Format: VSNs are stored in hex and can be parsed from logs or registry manually or using tools like Partition-4DiagnosticParser.

## iSerialNumber

- A unique identifier typically stored in the hardware/firmware of a USB device.
- Assigned per USB specification and follows the device even if plugged into different computers.
- Helps track specific device usage across multiple systems and over time.
- Devices that go through Windows Hardware Quality Labs (WHQL) are required to have this field populated.
- How to Extract iSerialNumber:
  - Use a USB write blocker or hardware that reads descriptors without plugging into a live system.
  - Use Microsoft USBView/UVCView tool (from Windows SDK) to scan ports and list descriptors including iSerialNumber.
  - Look at the device casing, but verify against actual reported value—some print fake ones.
- USB devices can have two serial numbers:
  - iSerialNumber: Used in the registry (USBSTOR).
  - SCSI Serial Number: Found via PowerShell or diagnostic logs.
- Mapping SCSI Serial Numbers to iSerialNumbers
  - SCSI Serial Number is not stored in the Windows Registry.
  - To map it, use:
    - The ParentId field in the `Microsoft-Windows-Partition/`Diagnostic.evtx`` `Event ID 1006` (contains iSerialNumber)
    - The SerialNumber field in the same log (contains SCSI Serial Number)
  - Use the RegistryId value from the log to correlate with the DiskId in the Registry:
    - Path: `SYSTEM\CurrentControlSet\`Enum\USBSTOR\<DeviceID>\<SerialNumber>\Device`` Parameters\Partmgr
  - This DiskId and RegistryId linkage allows you to tie together SCSI Serial and iSerialNumber, even if only one is present in each data source.

## UASP Device Profiling

- UASP (USB Attached SCSI) devices differ from USBSTOR devices and require additional keys for identification.
- Uses ParentIdPrefix instead of iSerialNumber under the registry key:
  - `SYSTEM\CurrentControlSet\`Enum\SCSI``
- Devices still have iSerialNumber, but Windows prepends it with “MSFT30”, which should be removed for accurate tracking.
- Uses Device Parameters\Partmgr\DiskId for identifying volume info.
- No Volume GUIDs for UASP devices, making NTUSER.DAT correlation not possible.
- DiskId becomes essential for correlating with logs (especially Partition/Diagnostic logs

## USB Device Cleanup and Artifact Retention

- Windows Cleanup Behavior:
  - Automated cleanup of USB-related data was introduced with Windows 8 and continued in Windows 10, removing data about devices not recently connected.
  - Initial cleanup tasks ran on a 30-day cycle, but recent behavior is to clean only on major updates.
  - Affected artifacts:
    - USBSTOR, USB, SCSI, HID registry keys
    - `Microsoft-Windows-Partition/`Diagnostic.evtx`` log
- Alternative Artifact Sources:
1. DeviceMigration Key:
  - Path: `SYSTEM\Setup\Upgrade\PnP\CurrentControlSet\Control\DeviceMigration`
  - Survives cleanup and contains:
    - VID, PID, iSerialNumber, ParentIdPrefix, DiskId, LastPresentDate
2. `C:\Windows.old` Folder:
  - Created during major updates
  - May contain previous registry hives
  - Useful for recovering deleted profiling data
3. Volume Shadow Copies (if enabled):
  - Allow rollback to earlier versions of the registry and logs
  - Especially helpful if data was recently removed
4. Setupapi.dev.log:
  - Path: `C:\Windows\INF\`setupapi.dev.log``
  - Not affected by cleanup
  - Records installation/removal of USB devices
  - May cover the past month or two

## DeviceMigration

- Purpose of DeviceMigration Keys
  - They archive device data during system cleanup operations (e.g., major OS updates).
  - Allow investigators to retrieve historical USB information, even after data has been purged from standard locations
- Two Key Registry Paths
  - `SYSTEM\CurrentControlSet\Control\DeviceMigration`
  - `SYSTEM\Setup\Upgrade\PnP\CurrentControlSet\Control\DeviceMigration`
  - Both may exist; the second is more comprehensive and preferred.

## Event Logs

- Plug and Play Logs
  - System Log → `Event ID 20001`
    - Triggered when Plug and Play installs a driver.
    - Contains: Device name, vendor, and iSerialNumber (if available).
  - System Log → `Event ID 20003`
    - Indicates completion of installation.
- Removable Storage Event Logs (Windows 8+)
  - `Event ID 4663` – Tracks access to removable storage.
    - Requires enabling “Audit Removable Storage” in the Advanced Audit Policy Configuration.
  - `Event ID 4656` – Access attempt to a removable device.
  - `Event ID 6416` – Identifies a new device connection (Security log).
  - `Microsoft-Windows-Partition/`Diagnostic.evtx`` – Detailed logging of connects/disconnects and partition data.
- `Event ID 20001` / 20003 (Plug and Play Events)
  - Source: System Log
  - Purpose: Detect devices plugged in and recognized by Plug and Play Manager.
  - How it helps:
    - Event 20001 = driver install start
    - Event 20003 = driver install complete
  - Usage:
    - Cross-reference timestamps with logon events to identify which user was active when a device was connected.
  - Tracks all Plug and Play-capable devices, not just USB.
- `Event ID 6416` (Security Log)
  - Purpose: Logs when a USB device is attached to the system.
  - Example Details:
    - Timestamp: 7 Feb 2018 11:58:31
    - VID/PID: 0930 / 6545
    - USB iSerialNumber: 001D0F0C0801B92103DC04DA
  - Often paired with another 6416 event that includes the volume name.
  - Also correlates with `Event ID 4663`, which logs access to the device.
- `Event ID 4663` (Security Log – Object Access Audit)
  - Purpose: Captures file-level interaction on the removable media.
  - Example:
    - Operation: File STRATPLAN_2018Q4.docx was appended to folder BusinessPlans.
    - User: Chad
    - Process: Explorer.exe (suggesting GUI usage)
  - Captures:
    - File creation, read/write, modification
    - Deletion and attribute changes
  - Precise user account and process name


## Windows 8/2012+ and Removable Storage Logging

- Windows 8/2012+ introduced object auditing for removable storage via `Event ID 4663`, helping track BYOD (Bring Your Own Device) usage.
- This is configurable through Audit Removable Storage under Object Access in Advanced Audit Policy.
- `Event ID 4663` Features:
  - Captures file-level activities (create, delete, modify, etc.) on removable storage.
  - Shows user logon ID, object name (e.g., Device\HardDiskVolume4), and action.
  - Limitation: Lacks unique identifiers (e.g., iSerialNumber), so attribution relies on correlation with other logs or registry keys.
- Additional Insight:
  - `Event ID 4656` (failure to access) can indicate blocked attempts.
  - Event correlation is necessary for precise attribution, especially when identifiers like iSerialNumber aren’t logged.
  - Useful context is available from logs like:
    - `Microsoft-Windows-Partition/`Diagnostic.evtx``
    - `setupapi.dev.log`
    - Registry keys like MountedDevices, MountPoints2.
- Contextual Tracking & Logon Mapping
  - Usage can be tracked further by mapping logon IDs with Object Name in event logs.
  - Backtracking through 4663 events may expose prior usage patterns.
  - `Event ID 4663` may not be triggered if access is denied (e.g., Group Policy blocks access). Instead, `Event ID 4656` logs the failure.
  - Compatibility Note:
    - These auditing capabilities are not available in Windows XP/2003


## Windows 10 – Security Log Audit via Event ID 6416

- Feature: Logs each time a Plug and Play (PnP) device is added.
- Policy Path: Found under Advanced Audit Policy Configuration → Detailed Tracking → “Audit PNP Activity”
- Default: Disabled by default. Must be explicitly enabled.
- Key Benefits:
  - Logs are stored in the Security Log, which is more centralized and persistent than System logs.
  - Each plug-in attempt is recorded (unlike `Event ID 20001`, which only logs the first installation).
  - Captures detailed hardware info: VID, PID, iSerialNumber, and volume name.
- MBAM/Operational Log – For Enterprises
  - MBAM = Microsoft BitLocker Administration and Monitoring.
  - MBAM/Operational log records:
    - Mount/dismount events for removable devices.
    - Event IDs 39/40.
    - Assigned volume GUID – allows cross-referencing with registry artifacts.


## Checklist

1. Audit USB Devices and Types
  - `Enum\USB` (SYSTEM)
2. Document Vendor ID and Product ID
  - `Enum\USB` (SYSTEM)
3. Document Device iSerialNumber
  - `Enum\USB` (SYSTEM)
4. Get Friendly Name, Vendor, Product, Version
  - `Enum\USBSTOR` (SYSTEM) — Also HID & SCSI
5. Document First and Last Time Connected
  - `Enum\USBSTOR` (SYSTEM) — Also HID & SCSI
6. Document Last Time Device Removed
  - `Enum\USBSTOR` (SYSTEM) — Also HID & SCSI
7. Determine Volume Name
  - Windows Portable Devices (SOFTWARE)
8. Find Last Mountpoint Drive Letter
  - VolumeInfoCache (SOFTWARE)MountedDevices (SYSTEM)
9. Document Volume GUID (USBSTOR)
  - MountedDevices (SYSTEM)
10. Identify Related User Accounts (USBSTOR)
  - MountPoints2 (NTUSER.DAT)
11. Determine Volume Serial Number
  - Partition/`Diagnostic.evtx` (Log)
