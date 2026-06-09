# Data Recovery

## Volume Shadow Copies

### Purpose
Windows Volume Shadow Copies create differential, block-level backups of changed data on NTFS volumes.

### Default Behavior
- Introduced in Windows Vista for client operating systems.
- Snapshots are typically taken approximately once per week.

### Forensic Benefits
- Revert files or entire volumes to previous points in time.
- Compare multiple versions of files.
- Recover deleted data if it existed during a snapshot.

### Limitations
- Only specific point-in-time versions are available, depending on when snapshots occurred.
- Typically uses about `3–5%` of disk space, so older snapshots may be overwritten.

### ScopeSnapshots on Windows 8+
- Limits backups to “system restore only.”
- May omit full user data.
- Can be disabled through a registry tweak to make snapshots more complete for forensic use.

### Tools
- X-Ways
- Magnet AXIOM
- VSCMount
- VSC Toolset
- ShadowExplorer
- KAPE

KAPE can collect artifacts from Volume Shadow Copies during triage and supports deduplication to reduce redundant data collection.

### Cross-Platform Approach
`libvshadow`, integrated into the SIFT Linux workstation, can emulate the Windows shadow service and enable VSC analysis outside Windows.

---

## NTFS Clusters

### Allocated Clusters
- Currently used by an active file.
- Contain live data that has not been deleted.

### Unallocated Clusters
- Not assigned to a specific active file.
- May contain remnants of deleted data or file fragments.
- Can be used for deleted-data recovery until overwritten.

### Cluster Basics
- A cluster is the smallest file-allocation unit in NTFS.
- The typical NTFS cluster size is `4096 bytes`.

### File Fragments
Even when a complete file cannot be recovered, partial data fragments may still exist in unallocated clusters.

### Forensic Recovery
Tools can analyze clusters, check allocation status, and recover deleted or partially deleted files using:

1. Existing filesystem metadata.
2. Deeper carving methods.

### Overwriting and Erasure
- One-pass overwriting is generally sufficient to block most forensic recovery on modern spinning hard drives.
- SSD garbage collection can automatically erase or consolidate unallocated areas, effectively wiping deleted data.

### NIST Guidance
NIST guidance updated in December 2014 recommends:

- One-pass overwrite for non-sensitive data.
- Physical destruction for high-sensitivity data.
- Proper software-based or hardware-based erasure/destruction procedures for SSDs.

---

## Recovery Methods

### Metadata Method
The metadata method relies on filesystem records, such as NTFS MFT entries, to identify exactly which clusters belong to a deleted file.

### Data Layer / File Carving Method
File carving searches the disk for known file signatures, such as an executable file’s `MZ` header.

This method relies on:

- A recognizable file footer, or
- An estimated file size when no footer is available.

File carving often produces partial or fragmented recoveries when intact metadata is unavailable.

---

## File Carving

### Tool
- PhotoRec

---

## Curse of SSD

### Wear Leveling
SSDs have a limited number of writes per memory cell. To extend lifespan, SSD controllers continually relocate data to different physical blocks.

This behavior can rapidly overwrite deleted data or slack space, making traditional recovery techniques less effective.

### TRIM
Operating systems send the TRIM command to SSDs to indicate which blocks are no longer valid after a file is deleted.

The SSD can then erase those blocks in advance, which results in:

- Fewer remnants of deleted files.
- Less useful unallocated space for recovery.

### Forensic Implications
- SSD behavior varies by drive model, firmware, and operating-system settings.
- Some SSDs may retain useful unallocated data, while others may retain almost none.
- Deleted-data recovery from SSDs should not be assumed to be stable or reliable over time.
- Windows forensic artifacts such as registry hives and event logs usually remain available because they are stored in allocated space.

### Key Point
SSD technology complicates file and slack-space recovery, but it does not eliminate standard forensic artifacts stored in allocated areas.

---

## Stream Carving

### Purpose
Stream carving focuses on recovering fragments of data rather than complete files.

### When It Is Useful
- Large files such as databases are heavily fragmented.
- RAM contents are fragmented.
- Classic file carving is impractical.

### Forensic Value
Even partial fragments can preserve valuable information such as:

- Timestamps
- Source information
- User activity
- Browser database fragments
- Registry hive fragments

### Tools
- Magnet AXIOM can automatically identify and extract fragments from memory or disk images.

---

## String Searching

### Purpose
String searching is widely used in:

- Memory analysis
- Malware reverse engineering
- Unallocated-space searches on disk images

Investigators commonly search for:

- IP addresses
- Filenames
- Domains
- User credentials
- Registry paths
- File paths
- URLs

### `bstrings` Utility

`bstrings` combines ASCII/Unicode string extraction with advanced search options, including regular expressions.

### Example Commands

```bash
bstrings -f file -m 8
```

Find strings with a minimum length of 8 characters.

```bash
bstrings -f file --ls search_term
```

Locate a specific search term.

```bash
bstrings -f file --lr ipv4
```

Match IPv4 addresses using a built-in regex pattern.

### Other Tools
Autopsy can index evidence sources for rapid keyword searching. Indexing can be time-consuming and may omit some raw data.

### Two Approaches to String Searching

#### Direct Bit-by-Bit Searches
- More comprehensive.
- Can be slower.
- May still miss data hidden inside certain compressed formats.

#### Indexed Searches
- Faster for repeated queries.
- Can find text in partially supported compressed formats.
- Depends on how the indexer defines searchable text.

### Practical Example
A common use case is searching for a BitLocker recovery key.

### Key Takeaways
- Direct searching is comprehensive but slower.
- Indexed searching is faster but depends heavily on the indexer’s capabilities and limitations.

---

## File Metadata

Many file formats embed internal metadata that can reveal information beyond normal filesystem timestamps.

This metadata often travels with the file when it is:

- Moved between drives.
- Uploaded to cloud services.
- Recovered from unallocated space.

### Examples
- Microsoft Word documents may contain embedded company details that identify file origin.
- Photos may store GPS coordinates showing location and time.
- Criminal investigations may use metadata to identify authors, locations, or source systems.

### Tool: ExifTool
ExifTool by Phil Harvey is a widely respected open-source utility that supports approximately 180 metadata formats.

It is useful for extracting and interpreting embedded metadata during forensic analysis.
