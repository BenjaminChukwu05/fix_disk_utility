# Encrypted HDD Repurposing and Ventoy exFAT Recovery

**Cross-platform IT support case study | Fedora Linux + Windows 11 | October 2026**  
**Outcome:** Ventoy exFAT repaired and boot-tested; an aging HDD read-scanned in full, used for a checksum-verified client-backup copy, and retained only with a separate copy of that data. Temporary Fedora staging data was subsequently removed.

> **Scope and consent:** This case involved two distinct external drives. The client who supplied the previously BitLocker-encrypted HDD authorized its erasure and reuse. No BitLocker credentials were recovered or bypassed. Identifying information and client files are intentionally excluded from this public case study.

## Problem

A 1 TB Ventoy USB drive used for IT support repeatedly triggered Windows disk-health warnings. The tool needed to remain usable as a bootable multi-ISO utility drive, and its existing contents included client backups and other working files. The repair had to preserve that data.

Separately, a roughly 500 GB Toshiba USB HDD was BitLocker-encrypted and no longer needed for its original purpose. The recovery credentials were unavailable, but the client who supplied the drive had authorized its complete erasure and reuse. The drive also had concerning SMART history, so it was not selected as the primary destination for irreplaceable backup data.

## Systems and tools

| Component | Role |
|---|---|
| ThinkPad workstation, Fedora Linux | Disk discovery, SMART assessment, repartitioning, backups, verification |
| Windows 11 on the same workstation | exFAT diagnosis and repair with CHKDSK |
| 500 GB Toshiba 2.5-inch external HDD | Authorized encrypted-disk repurposing experiment |
| ~1 TB Ventoy external USB disk | Main IT utility drive requiring filesystem repair |
| Internal Fedora NVMe (LUKS + Btrfs) | Temporary destination for verified Ventoy backup |

Tools: `lsblk`, `smartctl`, `journalctl`, `sgdisk`, `partprobe`, `mkfs.ntfs`, `rsync`, GNU `ddrescue`/`ddrescuelog`, `sfdisk`, `dd`, `sha256sum`, `udisksctl`, `fsck.exfat`, Windows `chkdsk`, `fsutil`, and `Get-Volume`.

## Method and findings

### 1. Identify and assess the encrypted HDD

Fedora identified the external HDD as a **500 GB Toshiba MQ01ABF050** with an existing Windows-style GPT layout, including a BitLocker volume. Its capacity was initially thought to be 1 TB; device discovery corrected this assumption before any destructive operation.

SMART findings included:

- Overall attribute-based health result: `PASSED`, **not a guarantee of reliability**.
- **48 reallocated sectors**, with **0 current pending** and **0 offline uncorrectable** in the available readings.
- Approximately **17,241 power-on hours**.
- The short self-test **completed without error**.
- The extended self-test was **aborted by the host with 90% remaining**; it did **not** complete and must not be reported as a passed surface scan.
- Kernel logs also captured USB disconnects and an I/O read error involving the external HDD. The exact reason the long self-test aborted was not established.

**Initial decision:** Reuse the HDD cautiously as non-critical scratch storage, not as the sole destination for the Ventoy recovery backup. A later client-backup copy was made to this HDD only while a second copy remained on the repaired Ventoy drive.

### 2. Remove the encrypted layout without decrypting its contents

After checking device identity and authorization, the existing partition table was removed using `sgdisk --zap-all`, a new GPT with one Microsoft basic-data partition was created, and the partition was quick-formatted as NTFS with `mkfs.ntfs`. A final `lsblk`/`blkid` check confirmed the new filesystem.

This operation **discarded the encrypted partition layout**. It did **not** recover the locked data, securely overwrite the entire disk, or repair the HDD's physical media issues.

> **Destructive-command warning:** Device paths such as `/dev/sda` are assigned dynamically. Commands shown in the private case recording are historical evidence, not a script to run against another machine. Identify and independently verify the intended physical drive immediately before wiping it.

### 3. Protect the Ventoy contents before modification

The utility drive was identified as a roughly **1 TB Ventoy disk**, with an exFAT data partition and a **32 MiB `VTOYEFI`** partition. Fedora mounted the exFAT source read-only.

An available-space check showed approximately **595 GiB free** on Fedora. The source held approximately **321,016,499,359 bytes** of regular file contents. A full `rsync` transfer copied:

| Transfer metric | Result |
|---|---:|
| Regular files | 157,342 |
| Directories | 27,455 |
| Total file/directory entries | 184,797 |
| File data transferred | 321,016,499,359 bytes |
| `rsync` transfer return code | `0` |
| Fedora free space afterward | ~332 GiB |

A second `rsync` pass using `--checksum --dry-run` completed with **return code 0 and an empty difference log**. This supports byte-for-byte agreement between the backup and readable source files at that time; it does not prove files were undamaged before the original incident.

The partition table, first 1 MiB of the USB disk, and full `VTOYEFI` partition were also saved separately, with SHA-256 sums for the image files.

### 4. Reconcile contradictory filesystem diagnostics

Fedora `fsck.exfat -n` reported the exFAT data volume **clean**, counting **157,342 files** and **27,455 directories**. Windows subsequently reported:

- exFAT **allocation bitmap corruption** during read-only `chkdsk`.
- Volume flagged **Dirty** by `fsutil dirty query`.
- `Get-Volume` reporting **Warning**.

Thus, the earlier Linux result could not be treated as final evidence that Windows would accept the filesystem as healthy.

With the verified backup available, Windows `chkdsk /f` corrected the detected exFAT metadata inconsistency. Follow-up checks reported:

- `chkdsk`: **no filesystem problems**.
- `fsutil dirty query`: **NOT Dirty**.
- `Get-Volume`: **Healthy**.

The read-only Windows check also displayed a shadow-copy-storage diagnostic message; despite it, CHKDSK completed the filesystem verification.

### 5. Examine the changes introduced by CHKDSK

Windows reported **two additional files** and **one additional directory index** after repair. A post-repair checksum comparison from Fedora yielded **return code 0** and only three added entries:

```text
cd+++++++++ FOUND.001/
>f+++++++++ FOUND.001/FILE0000.CHK
>f+++++++++ FOUND.001/FILE0001.CHK
```

No content differences were reported for previously backed-up files. The two recovered `.CHK` fragments totaled **320,208,896 bytes (~305 MiB)**. One had a recognizable **Bink Video** file signature; the other was classified as generic binary data. Neither fragment's original identity or completeness was established.

Both fragments were copied separately to Fedora and independently verified with `cmp`.

### 6. Verify the utility drive's actual purpose

A real UEFI boot test on the ThinkPad established that:

- The Ventoy boot menu loaded successfully.
- ISO files were visible.
- A test ISO started successfully.

**Result:** The utility drive was functioning as bootable Ventoy media again, with Windows filesystem checks clean. The pre-repair Fedora backup was retained through further verification, then removed in a later cleanup phase. No Ventoy rebuild was necessary.

### 7. Investigate the HDD further and complete a full surface-read assessment

In a follow-up Fedora session, an initial **1 GiB file write, SHA-256 comparison, and post-remount byte comparison passed**. The drive's existing SMART defects persisted: **48 reallocated sectors**, **6 reallocation events**, **0 pending sectors**, **0 offline uncorrectable sectors**, and **3 historical UDMA CRC errors**.

An alternative USB-to-SATA adapter was tried after earlier connection difficulties. Across the case, **four extended SMART self-tests were recorded as “Aborted by host,” each with 90% remaining**. These were inconclusive tests, **not** completed surface scans or reported unreadable-sector failures. The precise cause of the aborts was not isolated.

To test actual readability independently, GNU `ddrescue` read the HDD in resumable, read-only segments, sending data to `/dev/null` and recording coverage in a *diagnostic-only* mapfile on Fedora. The completed mapfile reported:

```text
Mapfile extent: 500107 MB
Rescued/read:   500107 MB (100%)
Non-tried:      0 B
Bad-sector:     0 B
Read errors:    0
```

The full logical address range was readable during this assessment. This does **not** reverse the drive's historical remapped sectors, establish its future reliability, or constitute a full-disk write verification.

### 8. Transfer and checksum-verify the client backup

A **pre-existing laptop backup folder**, staged inside the verified Fedora recovery copy, was copied to the Toshiba's NTFS volume. This was a transfer of the available *backup dataset*, not proof that every file from the original laptop had been captured.

| Transfer metric | Recorded result |
|---|---:|
| Regular files copied | 97,632 |
| Directories counted | 22,329 |
| File data | 152.10 GB (about 142 GiB apparent size) |
| Copy elapsed time | 2 h 19 min |
| Post-copy `rsync -rci --dry-run --delete` exit code | `0` |
| Verification difference-log lines | `0` |

After `sync` and unmount/remount, the checksum-based comparison found no differences between the staged source and HDD copy. Subsequent SMART readings were unchanged and the filtered kernel log contained no new matching disk errors.

**Retention and handover:** The Toshiba HDD was subsequently handed over to the client, as confirmed by the technician in an October 2026 follow-up. The repaired Ventoy drive was confirmed to hold the client's backup as a temporary second copy at the last inspection. The older Toshiba is **not** considered a reliable sole long-term copy. Independent backup arrangements and client-side verification of file access remain **unconfirmed**.

### 9. Remove verified temporary Fedora staging data

Once the Ventoy copy was confirmed present and the HDD copy had passed checksum verification, the technician explicitly authorized removal of the complete Fedora `Ventoy-Recovery` staging directory. The deletion was guarded by exact-path and filesystem checks; an initial interactive removal was interrupted, then the remainder was removed. The directory's absence was verified afterward.

| Fedora storage | Before | After |
|---|---:|---:|
| Used (rounded `df -h`) | 611 GiB | 347 GiB |
| Available (rounded `df -h`) | 319 GiB | 582 GiB |
| Usage | 66% | 38% |

**Observed available-space increase: approximately 263 GiB.** The separate recovery metadata, recovered fragments, and diagnostic logs were not part of the deleted directory. The working Ventoy disk and its client backup were not deleted.

## Final status

| Workstream | Outcome |
|---|---|
| Encrypted HDD partition removal and NTFS reuse | Completed; prior encrypted layout discarded, not decrypted |
| HDD full read-only assessment | **500.1 GB read; zero errors** in the ddrescue diagnostic mapfile |
| Extended HDD SMART self-tests | Four **host-aborted**, not completed |
| HDD long-term reliability | **Not established**; 48 historical reallocated sectors |
| Full Ventoy data backup | Completed and checksum-verified |
| exFAT allocation bitmap repair | Completed with Windows CHKDSK |
| Dirty flag and Windows health state | Cleared / Healthy |
| Post-repair file-content comparison | No mismatches in original files; 2 additional recovered fragments |
| Recovered `.CHK` fragments | Copied and byte-verified separately |
| Ventoy UEFI boot and ISO launch | Passed |
| Client backup copied to Toshiba | 97,632 files; checksum verification found **zero differences** |
| Temporary Fedora recovery backup | Deleted after checks; about **263 GiB additional available space** |
| Client HDD handover | **Completed**, as reported by the technician after the recorded tests |
| Client-side file access and dependable backup | **Not confirmed**; additional copy was retained on Ventoy at last check |

## Lessons

1. **Encryption and hardware usability are separate concerns.** Authorized reuse of an encrypted disk need not involve decrypting its prior files, but formatting does not erase historical hardware faults.
2. **Protect data before repair.** A complete, verified backup made subsequent exFAT metadata repair a controlled risk.
3. **Compare diagnostics across operating systems.** Linux `fsck.exfat` and Windows CHKDSK reached different conclusions on the same volume.
4. **Read-only checks and exit codes are evidence, not guarantees.** Verify the byte contents and the actual boot workflow separately.
5. **Preserve recovery artifacts without overstating them.** CHKDSK `.CHK` files may contain orphaned data, but a signature alone does not establish their original use or integrity.
6. **Separate internal self-test failures from real read coverage.** A host-aborted SMART test is inconclusive; a completed read-only scan establishes current readability but not future reliability.
7. **Close the data-retention loop.** Verify transfers before deleting staging copies, and retain an independent copy when the client destination is an aging drive.

## Evidence and privacy

See [Evidence register](docs/evidence.md) for a claim-by-claim map of the private sources, and [Publication checklist](docs/publication-checklist.md) before adding screenshots or command extracts. The raw terminal ZIP, client files, disk-image backups, and detailed local paths should **not** be committed to this public repository.

**Limitations:** This case study combines Fedora recordings, separately provided Windows command transcripts, and a user-observed boot test. It does not contain a full raw Windows recording. The Toshiba's complete read test establishes readability only at assessment time. The pre-existing client backup may have collection gaps, and independent client-side file access and a dependable backup arrangement have not been verified.

[Selected sanitized technical evidence](docs/selected-evidence.md)
