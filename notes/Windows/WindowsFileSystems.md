# Windows File Systems

## NTFS

- Uses a log file to record metadata changes and help track the state and integrity of the filesystem. This is called **transaction logging**.
- Can track file changes through the **USN (Update Sequence Number) Journal**.
- Supports POSIX-style features such as **hard links** and **soft links**.
- Uses **Object IDs** to track files.
- Supports file-level encryption through **Encrypting File System (EFS)**.

### Tool

#### MFTECmd

- Takes the NTFS **Master File Table (MFT)** as input.
- Parses each record in the database into human-readable information.

---

## Master File Table (MFT)

- Every object gets a **FILE record** in the MFT.
- Each record has attributes that contain both data and metadata.
  - Files get records.
  - Directories get records.
  - Even the volume itself gets a record.
- Each record is **1024 bytes** long.
  - If a file is small enough, its data can be stored inside the MFT record together with its metadata.
- NTFS reserves an **MFT Zone** so the MFT can grow.
  - If the rest of the disk becomes full, the MFT Zone can be cut in half and made available for regular file storage.

### MFT Timestamp Fields

| Timestamp | Meaning |
|---|---|
| Modified | Time the content of a file was last modified |
| Accessed | Time when the file content was last accessed |
| Metadata | Time when the MFT record or file attribute changed |
| Created | Time the file was created in a volume or directory |

---

## Alternate Data Stream (ADS)

### Legitimate Uses

Windows uses ADS to store metadata, such as marking files downloaded from the internet with a security tag like **Zone.Identifier**.

When a file is downloaded from the internet, Windows adds a `Zone.Identifier` ADS to indicate its origin.

Example:

```text
C:\Users\user\Downloads\document.pdf:Zone.Identifier
```

#### Metadata Storage

- Some applications store metadata in ADS instead of modifying the main file.
- Example: Cloud storage services like Dropbox may tag downloaded files using ADS.

#### File Associations

- Windows can use ADS to store additional information about file properties, such as indexing data.

### Forensic Importance

Investigators can use ADS to track file origins, execution history, and potential hidden data that may not be visible through standard file listings.

- **Tracking file execution:** ADS records can show whether a suspicious file was downloaded or executed.
- **Evidence of persistence:** Malware often hides in ADS to evade detection.
- **Data hiding techniques:** Attackers may use ADS to store stolen data before exfiltrating it.

### Security Risks

Threat actors can abuse ADS to hide malware or other malicious content inside seemingly innocent files.

- Hiding malicious code
- Fileless malware execution
  - ADS can be used to execute malware while bypassing traditional file-based detection.
- Evasion of forensic analysis
  - Standard file explorers and many security tools do not display ADS, so attackers may use it to store and execute files discreetly.

Example ADS execution command:

```cmd
wmic process call create "cmd.exe /c start C:\legitfile.txt:hiddenmalware.exe"
```

### Detection and Analysis

Tools such as **PowerShell**, **FTK Imager**, and **MFTECmd** can help identify and extract ADS for forensic review.

#### Using Command-Line Tools

List ADS in a file:

```cmd
dir /r
```

Extract ADS:

```cmd
type C:\path\to\file.txt:hidestream > extracted.txt
```

Delete ADS:

```cmd
fsutil alternateDataStream delete C:\path\to\file.txt:hidden
```

#### Using Forensic Tools

- **FTK Imager**: Detects ADS in files.
- **MFTECmd**: Part of Eric Zimmerman's forensic suite; can extract metadata, including ADS.
- **PowerShell and Sysinternals Streams.exe**: Useful for advanced ADS analysis.

---

## Windows System File Layout

- **Program Files / Program Files (x86)**
  - Locations where 64-bit and 32-bit applications typically reside.
- **Windows**
  - Core operating system directory containing many important subfolders.
- **System32**
  - Stores critical system executables, libraries such as DLLs, logs, registry hives, and databases like SRUM.
- **Windows.old**
  - Backup directory created during OS upgrades.
  - Used for rollback if needed.
  - Contains backups of key folders from the previous installation, including:
    - Registry hives, both user and system
    - Program Files folders
    - Main Windows directory
    - Event and USB logs
    - Prefetch data
- **Users**
  - Contains user profiles and associated folders such as Desktop and Documents.
  - Also includes each user's registry hives, such as `NTUSER.dat`, and configuration data.
- **AppData**
  - Hidden per-user folder containing application-specific data.
  - Examples include shell items, databases, and browser folders.
- **OneDrive**
  - Example of cloud storage integration.
  - Stores and syncs files per user account.

---

## Universal Windows Platform (UWP)

- Many core Windows applications, such as Notepad in Windows 11, and many third-party apps now use this format.
- Each UWP app maintains its own data in:

```text
%UserProfile%\AppData\Local\Packages\[AppName]\
```

- This folder stores:
  - Settings
  - Logs
  - Caches, such as internet history and cookies
  - Registry hives that capture application usage
- UWP apps run in an isolated container and do not have full Windows registry or filesystem access.
- This design can spread related forensic artifacts across multiple locations, making investigations more complex.
- Standard forensic artifacts can still show UWP app usage and execution, including:
  - SRUM
  - UserAssist
  - Prefetch
  - BAM
- Jump Lists and special **MSIX** registry keys can track usage of these apps.
- Internet cache data for UWP apps is found in the `Packages` folder instead of shared system locations.
