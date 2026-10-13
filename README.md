# Encrypted HDD Repurposing and Ventoy exFAT Recovery

**Cross-platform IT support case study | Fedora Linux + Windows 11 | October 2026**  
**Outcome:** Filesystem repair and Ventoy boot functionality verified; separate aging HDD repurposed for non-critical use.

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

Tools: `lsblk`, `smartctl`, `journalctl`, `sgdisk`, `partprobe`, `mkfs.ntfs`, `rsync`, `sfdisk`, `dd`, `sha256sum`, `fsck.exfat`, Windows `chkdsk`, `fsutil`, and `Get-Volume`.

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

**Decision:** Reuse the HDD as an authorized, non-critical scratch disk, but do not rely on it as the only safe copy of the Ventoy files.

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

**Result:** The original utility drive was functioning as bootable Ventoy media again, with Windows filesystem checks clean and the pre-repair backup preserved. No Ventoy rebuild was necessary.

## Final status

| Workstream | Outcome |
|---|---|
| Encrypted HDD partition removal and NTFS reuse | Completed; **non-critical use only** |
| HDD long-term reliability | **Not established**; concerning SMART/USB history |
| Full Ventoy data backup | Completed and checksum-verified |
| exFAT allocation bitmap repair | Completed with Windows CHKDSK |
| Dirty flag and Windows health state | Cleared / Healthy |
| Post-repair file-content comparison | No mismatches in original files; 2 additional recovered fragments |
| Recovered `.CHK` fragments | Copied and byte-verified separately |
| Ventoy UEFI boot and ISO launch | Passed |

## Lessons

1. **Encryption and hardware usability are separate concerns.** Authorized reuse of an encrypted disk need not involve decrypting its prior files, but formatting does not erase historical hardware faults.
2. **Protect data before repair.** A complete, verified backup made subsequent exFAT metadata repair a controlled risk.
3. **Compare diagnostics across operating systems.** Linux `fsck.exfat` and Windows CHKDSK reached different conclusions on the same volume.
4. **Read-only checks and exit codes are evidence, not guarantees.** Verify the byte contents and the actual boot workflow separately.
5. **Preserve recovery artifacts without overstating them.** CHKDSK `.CHK` files may contain orphaned data, but a signature alone does not establish their original use or integrity.

## Evidence and privacy

See [Evidence register](docs/evidence.md) for a claim-by-claim map of the private sources, and [Publication checklist](docs/publication-checklist.md) before adding screenshots or command extracts. The raw terminal ZIP, client files, disk-image backups, and detailed local paths should **not** be committed to this public repository.

**Limitations:** This case study combines the recorded Fedora session with separately provided Windows command transcripts and the final user-observed boot test. It does not contain a full raw Windows recording. A passing functional test does not prove the utility disk will remain reliable indefinitely.

[Selected sanitized technical evidence](docs/selected-evidence.md)
