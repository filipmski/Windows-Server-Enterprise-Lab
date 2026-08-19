## Incident: File Shares Lost After Windows Server Backup Configuration

### Problem

While configuring Windows Server Backup on `SRV-FILE01`, I accidentally selected the existing `D:` data volume as the dedicated backup destination.

The `D:` volume was being used to store the company's shared folders:

D:\CompanyShares\
├── HR
├── IT
├── Sales
├── Management
└── Public

After configuring the scheduled backup, the existing data on the volume was no longer available.

As a result, users on `CLIENT01` lost access to their mapped network drives because the SMB shares hosted on `SRV-FILE01` no longer existed.

### Symptoms

- `D:\CompanyShares` was no longer available.
- Department shared folders disappeared.
- SMB shares such as `\\SRV-FILE01\HR` were unavailable.
- Group Policy mapped drives disappeared or became inaccessible on `CLIENT01`.
- The scheduled Windows Server Backup job had been configured to use the same disk that previously stored the shared company data.

### Root Cause

The existing data disk was mistakenly selected as a dedicated Windows Server Backup destination.

The same physical/virtual disk should not be used simultaneously as the production data volume and as the dedicated backup destination.

The lab originally used:

- `C:` — Operating System
- `D:` — Company shared data

A separate backup volume had not been created before configuring Windows Server Backup.

### Resolution

1. Removed the incorrectly configured scheduled backup.
2. Restored the `D:` volume for use as the File Server data volume.
3. Recreated the company folder structure:

   - `D:\CompanyShares\HR`
   - `D:\CompanyShares\IT`
   - `D:\CompanyShares\Sales`
   - `D:\CompanyShares\Management`
   - `D:\CompanyShares\Public`

4. Reconfigured NTFS permissions using the appropriate Active Directory security groups.
5. Recreated the SMB shares using the original share names.
6. Verified that the existing Group Policy drive mappings pointed to the recreated shares.
7. Refreshed Group Policy on `CLIENT01` and verified access to the mapped drives.
8. Created a separate virtual disk dedicated exclusively to Windows Server Backup.

### Final Storage Configuration

SRV-FILE01

C:  Operating System
    └── Windows Server 2022

D:  DATA
    └── CompanyShares
        ├── HR
        ├── IT
        ├── Sales
        ├── Management
        └── Public

E:  BACKUP
    └── Windows Server Backup

### Verification

After recreating the shares and permissions:

- Users could access their appropriate department shares.
- Unauthorized users were denied access.
- Group Policy drive mappings were restored on `CLIENT01`.
- File creation, modification, and deletion were successfully tested.
- The backup destination was separated from the production data volume.

### Lessons Learned

This incident demonstrated the importance of verifying the destination disk before configuring a backup job.

Production data and backup data should be stored on separate volumes. Before performing storage-related administrative operations, the existing disk layout and data should be verified to prevent accidental data loss.

The incident also provided practical experience with rebuilding SMB shares, restoring NTFS permissions, troubleshooting Group Policy drive mappings, and redesigning the server storage layout to separate production data from backups.