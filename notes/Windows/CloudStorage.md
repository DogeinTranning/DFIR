# Cloud Storage Forensics

> Converted from `CloudStorage.txt`.

## Types of Useful Forensic Artifacts from Cloud Apps

### Applications

  - Determine which cloud apps are in use.
  - Identify users and accounts involved.
  - Know where local sync folders are for further analysis.
- Files Available
  - Includes local and cloud-only files.
  - Logs may show uploaded, shared, or deleted files—even from other users.

### File Metadata

  - Timestamps (created/modified), file size, path, hashes.

### File Transfers

  - Tracks syncing between devices and cloud.
  - Can reveal exfiltration or lateral movement across devices.

### User Activity

  - Some apps offer detailed audit logs showing file interactions (open, delete, share).
  - Useful for understanding user intent and behavior.

## OneDrive

### Synced Files Folder

  - Located at: `%UserProfile%\OneDrive`
  - Contains local copies of cloud-synced files.
  - Volume Shadow Copies can reveal deleted or older versions.
  - Folder may remain even if a user disables OneDrive (but will be empty).

### Registry Check (to verify OneDrive usage)

#### Key

    - `NTUSER\Software\Microsoft\OneDrive\Accounts\Personal`
  - Confirms whether OneDrive is set up and which folder is used.

### Settings Metadata

#### Path

    - `%UserProfile%\AppData\Local\Microsoft\OneDrive\settings`
  - Contains `SyncEngineDatabase.db` or <UserCid>.dat (older versions).

#### Reveals

    - Full list of both local and cloud-only files
    - Folder structure
    - Sync status of items

### Log Files

#### Path

    - `%UserProfile%\AppData\Local\Microsoft\OneDrive\logs`
  - Up to 30 days of log history.

#### `.odl` logs show

    - File creation, rename, deletion
    - Upload/download times

### OneDrive Personal

#### Key

    - `NTUSER\Software\Microsoft\OneDrive\Accounts\Personal`

#### This registry key helps determine if OneDrive is enabled and authenticated. Key values include

    - UserFolder: Path to the local synced OneDrive folder.
    - UserCid: Microsoft unique ID for the user.
    - UserEmail: Associated Microsoft account email.
    - LastSignInTime: Last time the user signed into OneDrive (Unix time).
  - The UserCid can appear in OneDrive URLs, helping to trace cloud file activity from browser history.

### OneDrive for Business

  - Key:
  - `NTUSER\Software\Microsoft\OneDrive\Accounts\Business1`

#### Similar structure as the personal key but adds

    - UserName: Full name of the user.
    - ClientFirstSignInTimestamp: When the user first authenticated.
    - SPOResourceID: URL for the associated SharePoint site.

### Tenants Key

  - `NTUSER\Software\Microsoft\OneDrive\Accounts\Personal\Tenants`
  - `NTUSER\Software\Microsoft\OneDrive\Accounts\Business1\Tenants`
  - Tracks folders synchronized from external OneDrive accounts, referred to as “tenants.”

#### Includes

    - Files and folders shared with the user.
    - Data synced via Microsoft Teams or SharePoint.

### SyncEngines

  - `NTUSER\Software\SyncEngines\Providers\OneDrive`

#### What It Tracks

    - Files and folders requiring synchronization
    - Often more detailed than data found under Accounts\Personal or Accounts\Business1
  - Each subkey corresponds to a DriveItemId (a unique identifier for each root folder/tenant)

#### Can be cross-referenced with

    - `SyncEngineDatabase.db`
    - <UserCid>.dat
  - Key Values in Each Subkey:

#### MountPoint

    - Shows the path on the local file system where the data is stored
    - Matches entries seen in Tenants key

#### UrlNamespace

    - Indicates the source (e.g., SharePoint, OneDrive)

#### LastModifiedTime

    - Timestamp of the most recent update to the MountPoint
- Files on Demand

#### Cloud Sync Models

    - Early cloud drives simply mirrored local folders to the cloud.
    - Newer models (e.g., OneDrive, Dropbox Smart Sync) allow access to files stored only in the cloud by downloading them on demand.

#### File Availability Terms

    - Hydrated = Locally available.
    - Dehydrated = Only stored in the cloud.

#### Investigative Challenges

    - Not all cloud files are downloaded locally.
    - Local systems may lack traditional forensic file artifacts.
    - However, cloud app artifacts and admin/cloud access can help fill gaps.

#### Deleted Files

    - Deleting a file affects local, synced, and online copies.
    - Recycle bins (e.g., OneDrive’s 30–93 day retention) can aid recovery.
    - Deleted but once-downloaded files may still be recoverable via local filesystem recycle bins.

#### Behind the Scenes – NTFS Reparse Points

    - Cloud services use NTFS reparse points (filesystem redirects) to fake the appearance of local files.
    - These enable seamless interaction (e.g., auto-download on click) while complicating forensic analysis.

### Settings Files

#### <UserCid>.ini (Text format)

##### Contains sync and transfer metadata

      - library: UserCid and folder location
      - lastRefreshTime: Last sync time (Unix epoch)
      - requestsSent: Number of sync requests made
      - BytesTransferred: Total bytes transferred during sync

#### <UserCid>-ProfileServiceResponse.txt (JSON format)

##### Contains identity information

      - givenName: First name
      - surname: Last name
      - userPrincipalName: Microsoft cloud email (e.g. user@domain.com)

#### `SyncEngineDatabase.db` (SQLite format)

    - The main forensic database for OneDrive
    - Stores comprehensive sync activity (covered in more detail elsewhere)

#### `healingItems.txt`

    - Logs sync errors
    - Useful for tracking deleted or missing file identifiers over time

#### `SafeDelete.db` (SQLite)

    - Tracks soft-deleted items
    - Table: filter_delete_info retains deletion metadata (including full path and process)
    - Can help identify deleted items even after sync clears them

### Metadata Database

#### Changeover Date

    - March 1, 2023 (OneDrive version 23.038.0219.0001)

#### Old Format

    - \<UserCID\>.dat (custom/homegrown format)

#### New Format

    - `SyncEngineDatabase.db` (SQLite)
  - Both databases store metadata about files and folders in OneDrive.
  - Tracks items stored locally and in the cloud (even those not synced locally).
  - Deleted items are rarely retained long—they’re often quickly deallocated.

#### Hashing algorithm change

    - Old: SHA1
    - New: quickXorHash, used in SharePoint and OneDrive for Business

#### od_ClientFile_Records Table - Key Fields

    - resourceID: Unique file identifier
    - parentResourceID: ID of the parent folder
    - eTag: Secondary identifier derived from resourceID
    - fileName: Name of the file
    - volumeID: Disk volume ID (convert to hex for shell item matching)
    - lastChange: Last modification timestamp (Unix Epoch)
    - size: File size (only for files, not folders)
    - fileStatus: Sync status (see codes below)
    - sharedItem: Was the file shared? (1 = yes)
    - localHashDigest: Binary BLOB storing the quickXorHash value

#### fileStatus Values

    - 2 = Available locally
    - 5 = Excluded
    - 6 = Not Synced
    - 7 = Not Linked
    - 8 = Available online
- Logs
  - `%UserProfile%\AppData\Local\Microsoft\OneDrive\logs\`
  - Stores up to 30 days of logs in binary format

#### Common file extensions (based on version/type)

    - `.odl`, `.odl`sent, `.odl`gz, `.aodl`
  - Record detailed interactions between the local system and cloud storage

#### Can be large and complex, but crucial for tracing

    - File uploads/downloads
    - File creation/deletion/renaming
    - Sync and telemetry activity

#### Tools & Parsing Support

##### Yogesh Khatri

      - Created a Python parser for these logs
      - Accounts for both plaintext (`ObfuscationStringMap.txt`) and Bcrypt-encrypted (`general.keystore`) file name schemes

##### Brian Maloney – OneDriveExplorer

      - Parses `.odl` log entries and ties them to specific files
      - Can export data to CSV
      - Offers GUI and command-line versions
      - Has parsed over 3,000 log entry types and continues to develop support
      - Added a feature called "CStructs" to define entry structures for parsing
  - Tip: `.odl` log entries often correlate with file deletions (especially those logged in the Recycle Bin).

### OneDriveExplorer

#### OneDriveExplorer Tool

    - Parses both new and old OneDrive database formats.

### Unified Audit Logs (UAL)

  - UAL provides detailed per-user activity logging for OneDrive for Business and SharePoint Online.
  - Enabled by default since 2019, but requires initial activation by an admin.
  - Retention period: 90 days (non-extendable by default).
  - Logs can be exported in CSV or JSON.
  - Logs appear with a delay of 15–30 minutes after the event.

#### File Events (per user)

    - FileAccessed, FileAccessedExtended
    - FileModified, FileModifiedExtended
    - FileDownloaded
    - FileDeleted, FileDeletedFirstStageRecycleBin, FileDeletedSecondStageRecycleBin
    - FileCopied (only audits copying within OneDrive/SharePoint, not to local storage)

#### Anonymous & External Sharing

    - Tracks shared file access and if shared links were created or used.

#### Audit Search Tools

    - Supports setting search criteria and alerts for real-time notification of matching events.

#### Synchronization Events

##### Help identify file syncing between local filesystem and cloud

      - ManagedSyncClientAllowed
      - FileSyncDownloadedFull
      - FileSyncDownloadedPartial
      - FileSyncUploadedFull
      - FileSyncUploadedPartial
    - Site Permissions Events:

##### Manage group permissions in shared OneDrive/SharePoint folders

      - SiteCollectionAdminAdded
      - AddedToGroup / GroupAdded
      - GroupRemoved / RemovedFromGroup
      - GroupUpdated

## Google Drive

- Replaced Google Backup and Sync in October 2021.
- Provides on-demand access to cloud files via a virtual FAT32 mountpoint (shows as a new drive).
- Files are only visible while the user is logged in.
- Forensics must deal with its ephemeral nature (volatile file system structure).
- Registry Key:

### `NTUSER\Software\Google\DriveFS\Share\SyncTargets`

  - Tracks assigned drive letters and account IDs in hex.
- File Metadata Storage:

### Located in

#### `%UserProfile%\AppData\Local\Google\DriveFS\<account` identifier>\

    - <account identifier> = encoded Google account ID

##### `metadata_sqlite_db`: Main database of file metadata

      - Includes file info (cloud-only, offline, trash, “Shared with me”)

### File Recovery & Caching

  - `content_cache` folder: Contains local file cache, recoverable for original cloud-stored files.
- Migration Data (Legacy Support):

### Path

  - `%UserProfile%\AppData\Local\Google\DriveFS\<account` identifier>\migration
  - Might include old Backup and Sync databases from users who upgraded

### Desktop database

#### Stores metadata for

    - Offline files
    - Cloud-only files
    - "Shared with me" items
    - Deleted files (in Trash)
  - Includes filename, size, timestamps, ownership, and status
  - Backup database: mirror_`metadata_sqlite_db` (can have duplicate data)
  - Table: items
    - stable_id: Unique file identifier
    - id: Cloud identifier (useful for correlating with audit logs or URL traces)
    - trashed: Whether the file is in Trash (0 = No, 1 = Yes)
    - is_owner: Whether the current user is the file owner
    - is_folder: Indicates if entry is a folder (1) or file (0)
    - local_title: Filename or folder name
    - file_size: File size in bytes
    - modified_date: Last modified time (Unix epoch, starts from when file added to Google Drive)
    - viewed_by_me_date: Last interaction timestamp (e.g., viewed, copied, moved)
    - shared_with_me_date: Marks files shared from other users
    - proto: Binary blob with MD5 hash, stored in protobuf format
  - Table: properties
    - account: Stores Google account info (username, email, ID – in protobuf)
    - account_settings: Full configuration of application/user settings
  - Table: item_properties
    - pinned: Marks files stored offline (1 = offline)
    - local-title: File/folder name
    - modified-date: File modification time (from local filesystem)
    - drivefs.Zone.Identifier: Origin info (e.g., browser zone ID)
    - trashed-locally: If 1, file was deleted from the local system (could still be in local Recycle Bin)
    - trashed-locally-name: Name of deleted file (as stored in $Recycle.Bin)
    - content-entry: Indicates file is cached locally
    - version-counter: Number of cloud-side revisions of the file

### Protocol Buffers (protobuf)

  - Google uses protobuf instead of XML/JSON for performance.
  - Protobuf is efficient but hard to interpret without tools—looks like binary with scattered readable strings.

#### Found in

    - content-entry (from item_properties)
    - account and account_settings (from properties table)
    - All stored in the `metadata_sqlite_db` database

### Local File Cache

  - `C:\Users\<username>\AppData\Local\Google\DriveFS\<identifier>\content_cache`

#### Contains cached files, often including

    - Cloud-only files
    - Trashed files
  - Files have renamed filenames with no extensions.

#### File types must be identified via

    - Header analysis (e.g., JPG signature)
    - Filename hash matching
    - Protobuf decoding

#### Mapping Cached Files to Original Metadata

    - Use the `metadata_sqlite_db` database

##### Focus on the item_properties table

      - Look for content-entry, which contains protobuf data mapping the cached file back to its filename

##### Cross-check with items table to validate

      - Filename
      - File size (as a secondary confirmation)

### Automate Cached File Analysis

#### gMetaParse (by Olaf Schwarz)

    - Parses `metadata_sqlite_db` and `content_cache` to identify locally cached files.
    - Works with items and item_properties tables.
    - Exports results in CSV or JSON.

##### Has both

      - Command-line mode (for rich metadata output)
      - GUI mode (launched with -g, useful for visual file tree exploration)

#### google-fs-recover (by Net Protect, LLC)

    - Python-based tool
    - Focuses only on cached files

##### Finds

      - Full path
      - Filename
      - MD5 hash
    - Exports a list of all cached files for fast filtering and review

### DriveFS Sleuth

  - A Python-based tool from a GitHub project focused on extracting and analyzing metadata from Google Drive for Desktop.
  - Uses the ProtoDeep library to decode and parse Protocol Buffer (protobuf) data.

### Google Workspace Logging

    - Logs retained for 180 days, or indefinitely if exported to BigQuery.
    - Logs can be exported in CSV format (up to 500,000 rows).
    - Logs may lag by 1–3 days, depending on Google’s systems.
    - API and SIEM support are available for real-time log forwarding.

#### Drive Audit Log Capabilities

    - Tracks viewed, created, previewed, modified, deleted, downloaded, and shared content.
    - Covers native Google files (Docs, Sheets, etc.) and uploaded files (PDFs, Word docs).

##### Key actions

      - Shared externally
      - Locally trashed
      - Downloaded (note: may include multiple events per action, even views)
    - Helps reconstruct user activity, though it’s often ambiguous whether an action was taken locally or online.

#### Limitations

    - `metadata_sqlite_db` has limited data for these audit actions.
    - Logs may contain redundancies (e.g., multiple "download" logs).
    - Link sharing logs the sharer, not the viewer/downloader.
    - Local file viewing via the virtual drive may not be logged.
    - IP addresses are not directly available in the log interface, but can be found in exported data.

## Timestamp

- Modification Time is preserved across all apps. This is consistent and reliable.

### Creation Time is

  - Preserved only in Google Drive for Desktop
  - Set to sync time in all others (i.e., overwritten)

### Access Time is

  - Only reliably preserved in Google Drive for Desktop
  - In Box Drive, it's tied to the file’s modification time
  - Overwritten during sync for Dropbox and OneDrive

### When syncing files to new devices

  - File modification time can be trusted.
  - Creation/access times may be unreliable unless using Google Drive for Desktop.

### Keep in mind

  - Filesystem timestamps differ from cloud metadata timestamps stored in app databases/logs.
  - Some apps may update timestamps based on how the file is accessed (e.g., web vs. local app).

## Virtualized filesystems

### Acts like a shortcut or pointer

  - Redirects filesystem activity (e.g., reads/writes) to cached cloud files.
  - Often uses NTFS reparse points (Box Drive) or FAT32 mountpoints (Google Drive Desktop).
- If the app is not running or the user is logged out, the virtual volume may appear empty.
- Acquiring a disk image won’t capture cloud-only files unless they were locally cached.
- FTK Imager’s “Contents of a Folder” or tools like KAPE are useful to acquire live content while the app is active

### If cloud files are auto-retrieved during acquisition, they

  - May overwrite unallocated space
  - May go beyond legal authority—legal review is advised
