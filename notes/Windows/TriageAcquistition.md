# Triage Acquisition

## Memory Acquisition

- **Do not pull the plug** on a computer while it is still running.
- **Volatile data** disappears or is destroyed once the system is powered off, including:
  - RAM
  - Current active connections
  - Running applications
  - Open/listening network connections
  - Configuration parameters
  - Encryption keys and passwords
  - Memory-only exploit techniques

### How Memory Acquisition Tools Access Data

- Before **Windows 2003 SP1**, tools could access memory via the handle:

  ```text
  \Device\PhysicalMemory
  ```

- Modern tools usually load a `.sys` driver to gain raw access to memory.
- The tool then dumps the memory contents into a raw file.
- This technique can also be abused by malware to access memory.
- Drivers generally must be Microsoft-signed to access memory on modern systems.

### Acquisition Footprints

Memory acquisition tools may leave artifacts behind, such as:

- Temporary files dropped in:

  ```text
  AppData\Local\Temp
  ```

- Deleted temporary files, which may look similar to malware activity.

### `hiberfil.sys`

- `hiberfil.sys` is created when the system enters sleep/power-save/hibernation mode.
- It may contain a compressed copy of RAM from the time of hibernation.

### Memory Acquisition Tools

- **F-Response**
  - Images RAM as if it were a physical drive.
- **WinPMEM**
- **Magnet Forensics RAM Capture**
- **DumpIt**
- **Belkasoft Live RAM Capturer**
- **FTK Imager**

### On a Dead System

If the system is no longer running, collect available memory-related files:

#### Hibernation File

- Contains a compressed RAM image.

```text
%SystemDrive%\hiberfil.sys
```

#### Page and Swap Files

```text
%SystemDrive%\pagefile.sys
%SystemDrive%\swapfile.sys
```

> `swapfile.sys` exists on Windows 8 / Windows Server 2012 and later.

#### Kernel-Mode Memory Dump

```text
%WINDIR%\MEMORY.DMP
```

---

## Analyzing Memory Images

### Stream Carving / String Searching

Used to extract and recover investigative artifacts from memory images.

Methods include:

- Data stream extraction
- String searching
- File carving based on:
  - File headers
  - File signatures
  - Keywords

Tools can examine a memory image similarly to a disk image.

### Example Tool: AXIOM

AXIOM can help recover artifacts such as:

- Chat sessions
- Internet history
- Web email

### Deep Memory Analysis

Deep memory analysis examines memory data structures.

Common tools:

- **Volatility**
- **MemProcFS**

These tools are pre-installed in the downloadable **Linux SIFT Workstation**.

### Recovering Encryption Keys

Memory analysis may help recover keys for encryption products such as:

- BitLocker
- TrueCrypt
- VeraCrypt

Tools:

- **Passware Kit**
- **Elcomsoft Disk Decryptor**

---

## Check for Disk Encryption

- Always check for disk encryption.
- If an encrypted volume is mounted, this may be the only chance to capture readable data from the live system.
- Some cloud storage data, DPAPI credentials, and browser data may only be available while the user is logged in.

### Live Imaging Guidance

If performing a live image because of encryption:

- Image the **logical drive**, not the physical drive.
- The logical drive is seen by the local machine as unencrypted.
- The physical disk may still be encrypted at the disk level.

---

## Create a Quick Triage Image

Collect the following artifacts during quick triage.

### Registry Hives

- `SAM`
- `SYSTEM`
- `SOFTWARE`
- `DEFAULT`
- `NTUSER.DAT`
- `USRCLASS.DAT`

### LNK Files

- `*.lnk`
- Provide evidence of file and folder opening.

### Event Logs

Located under:

```text
Windows\System32\winevt\Logs
```

### Other Log Files

Examples include:

- `setupapi.dev.log` — Plug and Play log file
- Antivirus/security logs
- IIS logs

### AppData Folder

- Many applications store valuable forensic artifacts here.
- Common artifacts include:
  - Logs
  - Databases
  - Application usage traces

### `$MFT`

- Master File Table.
- Metadata database for every file and folder on an NTFS volume.

### `pagefile.sys`

- Windows page file.
- May contain memory fragments.

### `hiberfil.sys`

- Compressed image of RAM from the last hibernation event.

### NTFS `$LogFile` and `$UsnJrnl:$J`

- File system journal and change log.
- Can record granular file activity such as:
  - File open
  - File close
  - File creation
  - File deletion

### Prefetch Files

- `*.pf`
- Created during application execution.

### Jump Lists

- Contain shell items showing:
  - File usage
  - Folder usage
  - Application usage

### Recommended Tool

Use **KAPE** for quick triage collection and processing.
