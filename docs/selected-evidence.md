# Selected evidence — Ventoy exFAT recovery and encrypted HDD reuse

**Case:** Cross-platform IT support maintenance (Fedora Linux and Windows 11, October 2026)  
**Publication status:** Sanitized excerpts prepared from private session recordings and the separately supplied Windows command transcript.  
**Privacy:** Hostnames, personal usernames, disk serials, UUIDs, original BitLocker volume labels, and client file paths have been omitted. The full recordings, disk images, backup data, and recovered file contents are not published.

> **How to read this page:** These are *selected and lightly reformatted outputs*, not complete raw terminal transcripts. Command output and manual observations are distinguished. Device names and drive letters are contextual to the original session and must never be assumed correct on another computer. See the [evidence register](evidence.md) and [README](../README.md) for limits and methodology.

## 1. Encrypted HDD: SMART health and diagnostics

**Source:** Fedora terminal recording (`smartctl` and `journalctl`). The USB HDD was approximately 500 GB and initially contained a BitLocker partition.

Selected SMART attributes:

```text
Attribute                    Raw value
Reallocated_Sector_Ct        48
Current_Pending_Sector       0
Offline_Uncorrectable        0
Power_On_Hours               17241
UDMA_CRC_Error_Count         3
```

Selected self-test results:

```text
Test                    Status                         Remaining
Short offline           Completed without error        00%
Extended offline        Aborted by host                90%
```

The kernel journal also recorded a USB disconnect and a read I/O error involving this HDD. The precise cause of the extended-test interruption was not established. The HDD was repartitioned and formatted for **non-critical scratch use**, *not* selected as the sole destination for the utility-drive backup. A completed NTFS quick format is not a full-disk health test or secure erase.

## 2. Utility-drive backup: transfer and verification

**Source:** Fedora terminal recording. The Ventoy exFAT partition was mounted read-only; data was copied to the workstation's internal encrypted Fedora storage.

Selected `rsync --stats` and shell results:

```text
Number of files:                  184,797 (reg: 157,342, dir: 27,455)
Number of regular files transferred: 157,342
Total file size:                  321,016,499,359 bytes
Total transferred file size:      321,016,499,359 bytes
Rsync exit code:                 0
```

A separate checksum-based dry-run compared the original Ventoy files against the Fedora backup:

```text
Verification exit code: 0
Log lines: 0
```

**Interpretation:** The transferred files matched the readable source contents at the time of verification. This cannot establish whether any source files were damaged *before* this case.

## 3. Contradictory exFAT checks

**Sources:** Fedora terminal recording and separately supplied Windows PowerShell transcript.

Fedora read-only check:

```text
$ sudo fsck.exfat -n /dev/sda1
/dev/sda1: clean. directories 27455, files 157342
```

Windows read-only check of the same Ventoy data volume:

```text
> chkdsk D:
Corruption was found while examining the volume bitmap.
Windows has checked the file system and found problems.
Run CHKDSK with the /F (fix) option to correct these.

> fsutil dirty query D:
Volume - D: is Dirty
```

**Interpretation:** Fedora's no-write checker did not flag the allocation bitmap inconsistency that Windows reported. The discrepancy justified preserving a verified backup *before* modifying the filesystem.

## 4. Windows exFAT repair and repeat checks

**Source:** Windows PowerShell transcript provided during the case; this part was **not** captured in the Fedora recording ZIP.

Selected Windows repair result:

```text
> chkdsk D: /f
Corruption was found while examining the volume bitmap.
Windows has made corrections to the file system.
No further action is required.
```

Selected follow-up results:

```text
> chkdsk D:
Windows has scanned the file system and found no problems.
No further action is required.

> fsutil dirty query D:
Volume - D: is NOT Dirty

> Get-Volume -DriveLetter D
FileSystemLabel : Ventoy
FileSystem      : exFAT
HealthStatus   : Healthy
```

The read-only Windows check displayed a shadow-copy-storage warning but completed its filesystem examination. No Ventoy reinstallation or data-partition reformat was required.

## 5. Post-repair byte comparison and recovered fragments

**Source:** Fedora terminal continuation recording.

A checksum-based, dry-run `rsync` comparison against the *original, unchanged* backup produced only the following new entries:

```text
Comparison exit code: 0
Differences reported: 3
cd+++++++++ FOUND.001/
>f+++++++++ FOUND.001/FILE0000.CHK
>f+++++++++ FOUND.001/FILE0001.CHK
```

No changes to previously backed-up files were reported. CHKDSK had introduced two recovered `.CHK` files totaling **320,208,896 bytes**. The `file` utility detected a Bink Video header in one; the other's contents were not identified. Both were preserved separately on Fedora:

```text
FILE0000.CHK: VERIFIED
FILE0001.CHK: VERIFIED
```

**Caution:** Matching bytes and a recognizable header do not establish the original identity, usability, or ownership of the recovered fragments. Their contents have not been published.

## 6. Functional boot test

**Source:** Technician's manual test confirmation in the case discussion, **not** a captured automated or video test.

| Test | Observed outcome |
|---|---|
| Ventoy UEFI menu loaded | Pass |
| ISO files visible in Ventoy | Pass |
| Selected test ISO launched successfully | Pass |

The functional test and Windows filesystem checks support the conclusion that the utility drive was usable again **at test time**. They do not prove future hardware reliability.

## Evidence boundaries

- Raw recordings and client data remain private.
- Commands involving `/dev/sda` and `D:` are historical examples only; device assignments can change.
- BitLocker partition removal did not recover credentials or decrypt files, nor did quick formatting securely overwrite every sector.
- The Toshiba HDD's historical media/USB issues remain unresolved; it should not be relied on as the only copy of important data.
- The Windows repair transcript and manual ISO test have a different provenance from the Fedora recording archive.

[Selected sanitized command evidence](docs/selected-evidence.md)
