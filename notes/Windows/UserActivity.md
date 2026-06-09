# Windows User Activity Artifacts

## Windows Search History

### WordWheelQuery

`WordWheelQuery` is a registry key that stores search terms entered in File Explorer or the Start menu.

**Registry path:**

```text
NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery
```

**Forensic value:**

- Reveals what users were searching for.
- May expose user interests or intentions.
- Helps identify specific file types in use, such as `.rar` or `.vmx`.
- Supports intrusion or theft investigations by revealing target data.
- Search records are often overlooked by users and rarely deleted.
- Can be manually viewed and cleared through the **Search Tools** ribbon in File Explorer.

**Other notes:**

- **Windows XP:** Used a different key: `Search Assistant\ACMru`.
- **Windows Vista:** Did not store search history.
- **Windows 8:** May store search subkeys under `WordWheelQuery` to show whether searches came from the desktop or the charm menu.

---

## Typed Paths

`TypedPaths` logs manually typed paths in the File Explorer address bar.

Examples:

```text
C:\
D:\vacation photos
\\192.168.1.3
\\STARK-FILESERVER
```

**Registry path:**

```text
NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

**What it records:**

- Local drive paths
- Remote or network paths
- UNC paths

**Forensic value:**

- Strong indicator of intentional user action.
- Helps tie activity to a specific user during forensic investigations.

### Order of Entries

Entries are automatically named in descending order of recency:

- `url1` = most recent
- `url2`, `url3`, etc. = older entries

This is not a traditional MRU list. The key names imply the order.

### Write Behavior Caveat

The `TypedPaths` key is not updated immediately when a path is typed. It is written only after the File Explorer window is closed.

Therefore, the key's last-written time reflects the closing time, not the exact typing time.

---

## RecentDocs

`RecentDocs` tracks user interactions with files and folders.

**Registry path:**

```text
NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
```

Each subkey includes:

- A list of the last 20 files of that type.
- An MRU list showing the access order.

### Investigative Insights

`RecentDocs` can help identify:

- Applications used, such as `.one` for Microsoft OneNote.
- Email access evidence, such as `.eml` files.
- Virtual machine files, such as `.vmx`.
- Word documents, such as `.docx`.
- Web activity from search queries or Chromium downloads, such as `.crdownload`.
- URLs typed in the Windows search bar that may create web-search traces.
- Human interaction, as opposed to automated malware activity.

### Why It Matters

`RecentDocs` can reveal:

- User intent.
- Usage patterns.
- Signs of malware or ransomware preparation.
- Local file usage.
- Internet-based activity.

It is also useful for correlating activity with other artifacts through MRU order and timestamps.

---

## Microsoft Office File MRU

Microsoft Office applications maintain their own MRU lists that store information about recently accessed files.

### Legacy Office Versions: Office 2003 to Office 2013

Office version numbers:

| Office Version | Version Number |
|---|---:|
| Office 2003 | `11.0` |
| Office 2007 | `12.0` |
| Office 2010 | `14.0` |
| Office 2013 | `15.0` |

**Registry path:**

```text
NTUSER\Software\Microsoft\Office\<VERSION>\<APPNAME>\File MRU
```

### Microsoft 365 / Modern Office Versions

For Office 2016, Office 2019, and Microsoft 365, version `16.0` is used.

User-based MRUs are stored under:

```text
NTUSER\Software\Microsoft\Office\16.0\<APPNAME>\User MRU\LiveID_###\File MRU
NTUSER\Software\Microsoft\Office\16.0\<APPNAME>\User MRU\ADAL_###\File MRU
```

These reflect cloud-based account identifiers, such as `LiveID` or `ADAL`.

---

## Microsoft Office Reading Locations

Microsoft Office tracks the last reading location for documents opened by a user. This allows Office to show prompts such as **“Pick up where you left off.”**

Each subkey stores:

- **File path:** Full name and location of the document.
- **Position:** Encoded scroll position showing where the user last was in the document.
- **DateTime:** Timestamp for when the document was last closed. This complements File MRU's last-opened data.

### Investigation Tip

1. Export the suspect document.
2. Open it on a test system to populate the Reading Locations key.
3. Modify the test system's `Position` registry value to match the suspect system.
4. Reopen the document. It will jump to the stored scroll position.
5. Take a screenshot as proof and include it in the forensic report.

---

## TrustRecords

Microsoft Office keeps records of documents that users have trusted, such as documents where editing was allowed or active content/macros were enabled.

The `TrustRecords` key can contain:

- Full path of the trusted document.
- Timestamp of when the file was trusted.
- Type of trust granted, such as:
  - Editing enabled
  - Macro allowed
  - Content enabled

This registry data has existed since at least Office 2010 and can go back years, making it a valuable forensic source.

### Forensic Value

`TrustRecords` helps analysts:

- Determine whether malicious Office files were opened and trusted.
- Verify whether users bypassed macro protections.
- Track suspicious or malicious document interaction.
- Identify potential security bypass attempts.
- Understand long-term document usage, especially in targeted attacks or data exfiltration cases.

---

## Common Dialog Keys

Common dialog keys store data about files and paths accessed through standard **Open** and **Save** dialog boxes across many applications.

When a user opens or saves a file, the path and filename may be recorded.

Common dialog boxes are shared by many programs, such as:

- Microsoft Word
- Microsoft Excel
- Browsers
- Encryption tools

This makes them a central source of user activity.

### Forensic Value

Common dialog keys can show:

- What files a user opened or saved.
- Which paths or folders were accessed.
- Which application activity may correlate with the file access.
- Whether the user accessed encrypted files or documents from suspicious folders.

### NTUSER.DAT Keys

| Key | Purpose |
|---|---|
| `OpenSavePidlMRU` | Tracks full paths of files opened or saved by the user. |
| `LastVisitedPidlMRU` | Records the last folder path visited when opening or saving a file. |
| `LastVisitedPidlMRULegacy` | Older version used for legacy compatibility. |

---

## Registry Redirection Hives

UWP is the modern Windows application model.

UWP apps are installed per user and are often found under:

```text
%UserProfile%\AppData\Local\Packages
```

They can also be discovered with PowerShell:

```powershell
Get-AppxPackage | Select-Object -Property Name
```

### Sandboxed Registry and File Access

UWP apps often run in sandboxed environments, which limits access to the system registry and file system.

Key points:

- Registry entries for these apps are virtualized.
- Virtualized entries may be visible only to the app.
- Entries are often removed when the app is uninstalled.
- They do not always write to traditional hives, such as `NTUSER.DAT`.
- Some apps may declare that certain data should persist after uninstall.

### Storage Location

UWP app registry writes are stored under:

```text
SystemAppData\Helium
```

UWP uses local copies of registry hives named:

| UWP Hive | Aligns With |
|---|---|
| `Registry.dat` | `SOFTWARE` |
| `User.dat` | `NTUSER.dat` |
| `UserClasses.dat` | `UsrClass.dat` |

---

## MSIX

MSIX is a Windows application packaging format introduced in 2018.

UWP apps and other applications deployed with MSIX may redirect registry writes to non-traditional, sandboxed hives.

### Registry Hive Redirection

Apps using MSIX may not store data in standard registry locations like `NTUSER.DAT`.

Instead, registry information may be redirected to hives stored within the app's folder.

Kroll Artifact Parser and Extractor, also known as KAPE, can:

- Scan the `Packages` folder for MSIX-related registry hives.
- Extract these hives for further analysis in tools such as Registry Explorer.

### Internet Metadata

MSIX redirection is not limited to registry data. Internet metadata generated by MSIX apps may also be stored internally.

This includes:

- Browser cookies
- Browser history
- Data stored in the `WebCacheV*.dat` database, even on Windows 11

These artifacts are useful for browser forensics and user activity analysis.

---

## Quick Reference Table

| Artifact | Location / Key | Main Forensic Value |
|---|---|---|
| WordWheelQuery | `NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery` | Search terms entered in File Explorer or Start menu. |
| TypedPaths | `NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths` | Manually typed local, remote, or UNC paths. |
| RecentDocs | `NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | Recently accessed files and folders by type. |
| Office File MRU | `NTUSER\Software\Microsoft\Office\<VERSION>\<APPNAME>\File MRU` | Recently opened Office documents. |
| Office Reading Locations | Office user registry data | Last reading position inside documents. |
| TrustRecords | Office user registry data | Documents trusted by the user, including macro/content enablement. |
| OpenSavePidlMRU | `NTUSER.DAT` | Full paths of files opened or saved through common dialogs. |
| LastVisitedPidlMRU | `NTUSER.DAT` | Last folders visited through open/save dialogs. |
| UWP Registry Hives | `%UserProfile%\AppData\Local\Packages` and `SystemAppData\Helium` | Sandboxed app registry and file activity. |
| MSIX Hives | App package folder | Redirected registry writes and internal app metadata. |
| WebCacheV*.dat | MSIX / app internal storage | Cookies, history, and internet metadata. |
