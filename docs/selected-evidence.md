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

## 7. Follow-up HDD testing and full read-only scan

**Source:** Fedora continuation recording, October 13–14. A 1 GiB test file was copied to the Toshiba, with matching SHA-256 and `cmp` results both initially and after remounting the NTFS volume.

SMART self-test log, summarized without device identifiers:

```text
Extended offline    Aborted by host    90% remaining
Extended offline    Aborted by host    90% remaining
Extended offline    Aborted by host    90% remaining
Extended offline    Aborted by host    90% remaining
```

A replacement USB-to-SATA adapter was used during troubleshooting. These are **inconclusive internal self-tests**, not confirmed media-read failures. The cause of the host-aborted tests was not established.

GNU `ddrescue` then read the *entire device* in successive resumable segments using a mapfile. The output was `/dev/null`, so **no recoverable disk image was created**. The final cumulative mapfile status was:

```text
Mapfile extent: 500107 MB
non-tried:            0 B  (0%)
rescued:      500107 MB  (100%)
non-trimmed:          0 B
non-scraped:          0 B
bad-sector:           0 B
```

The final segment ended with `read errors: 0`. SMART still reported **48 reallocated sectors**, **6 reallocation events**, **0 pending**, **0 offline uncorrectable**, and **3 UDMA CRC errors**. A full read pass does not certify the drive for long-term unique storage.

## 8. Client backup copied to the aging HDD

**Source:** Fedora continuation recording. The source was a previously staged laptop-backup folder, not the entire set of files that could have existed on the original client computer.

Selected `rsync` results:

```text
Number of files:             119,961 (reg: 97,632, dir: 22,329)
Number of regular files transferred: 97,632
Total file size:             152.10G bytes
Total transferred file size: 152.10G bytes
Elapsed transfer:            2h 19m (approx.)
```

After synchronizing and remounting the HDD, a full checksum dry-run compared the staged source and destination:

```text
Verification exit code: 0
Difference log lines:   0
```

The comparison verifies the transferred dataset's contents at that time. It does **not** guarantee that the aging HDD will remain reliable or that earlier backup collection was complete.

## 9. Post-verification staging cleanup

**Source:** Fedora continuation recording and technician authorization. After confirming the repaired Ventoy drive still held the client backup and checking the new HDD copy, the temporary Fedora recovery staging directory was removed.

```text
Fedora storage       Before      After
Used                 611 GiB     347 GiB
Available            319 GiB     582 GiB
Filesystem usage       66%        38%

Target absence check: PASS
```

The rounded `df` values show approximately **263 GiB more available space**. A separate metadata backup, recovered fragments, and diagnostic files were outside the removed directory.

**Handover boundary:** At the time of this case update, the client's HDD had not yet been confirmed handed over, and a reliable independent backup had not been arranged. The repaired Ventoy drive retained a temporary additional copy of the client dataset.

## Evidence boundaries

- Raw recordings and client data remain private.
- Commands involving `/dev/sda` and `D:` are historical examples only; device assignments can change.
- BitLocker partition removal did not recover credentials or decrypt files, nor did quick formatting securely overwrite every sector.
- The Toshiba HDD's historical reallocated sectors and repeatedly host-aborted SMART tests remain unresolved; despite a complete successful read pass, it should not be the only copy of important data.
- The client-backup folder was verified as transferred; completeness against the original client laptop was not established. Final handover and a dependable independent backup are still pending.
- The Windows repair transcript and manual ISO test have a different provenance from the Fedora recording archive.

[Return to the case study](../README.md)
