# Evidence register (private-source mapping)

This register records which findings were observed in the supplied **Fedora recording archive** and which were reported separately during the Windows troubleshooting discussion. It is designed for internal editorial verification; it does **not** contain the source files.

| Finding | Evidence source | Confidence / qualification |
|---|---|---|
| 500 GB Toshiba with BitLocker partition | `terminal.raw`, `lsblk` output | Directly observed; device identifiers redacted from public narrative |
| Short SMART test passed | `terminal.raw`, `smartctl -l selftest` | Directly observed |
| 48 reallocated sectors and 0 pending | `terminal.raw`, `smartctl -A` | Directly observed at test time; historical media issues remain |
| Extended test aborted by host, 90% remaining | `terminal.raw`, `smartctl -l selftest` | Directly observed; **not** a failed-media verdict or passed scan |
| USB disconnect(s) and read I/O error on HDD | `terminal.raw`, filtered `journalctl` | Directly observed; exact cause not determined |
| Partition table removed, GPT/NTFS recreated | `commands.log` + `terminal.raw` | Command sequence and successful output observed; not secure erase |
| Source file copy: 157,342 regular files / 321,016,499,359 bytes | `terminal.raw`, `rsync --stats` | Successful copy, exit code 0 |
| Baseline checksum comparison found no differences | `terminal.raw`, `rsync --checksum --dry-run` | Exit code 0, empty log |
| Ventoy boot metadata images and partition table saved | `commands.log` + `terminal.raw` | Commands completed; image checksum file was generated, but independent restore validation not shown |
| Fedora exFAT check clean | `terminal.raw`, `fsck.exfat -n` | Directly observed; not sufficient to dismiss Windows findings |
| Windows bitmap corruption and Dirty status | Windows PowerShell transcript supplied in conversation | Directly observed via pasted transcript; **not included in the ZIP** |
| CHKDSK `/f` corrected filesystem and follow-up check clean | Windows PowerShell transcript supplied in conversation | Directly observed via pasted transcript; **not included in the ZIP** |
| Two new `.CHK` fragments after repair | `segments/2026-10-12-190826/terminal.raw` | Only post-repair rsync additions; 3 itemized lines |
| Recovered fragment signatures and sizes | `segments/2026-10-12-190826/terminal.raw` | `file` identified one Bink Video header; other unknown |
| Both `.CHK` fragments separately copied and compared | `segments/2026-10-12-190826/terminal.raw` | `cmp`: both VERIFIED |
| Ventoy menu, ISO visibility, ISO boot | User-confirmed functional boot-test outcome in conversation | No captured BIOS screenshots or video supplied in this ZIP |
| HDD 1 GiB write/read, checksum and post-remount comparison | `segments/2026-10-13-124639/terminal.raw` | Byte match observed; limited coverage only |
| Four aborted long SMART tests, all at 90% remaining | `segments/2026-10-13-124639/terminal.raw`, `smartctl -l selftest` | Confirmed inconclusive tests; reason not determined |
| Full 500,107 MB ddrescue read assessment, zero read errors | `segments/2026-10-13-124639/terminal.raw`, `ddrescuelog --show-status` | Complete logical read coverage; diagnostic mapfile writes data to /dev/null, not an image |
| Client-backup rsync copy: 97,632 regular files / 152.10 GB | `segments/2026-10-13-124639/terminal.raw` | Transfer output observed; original laptop collection completeness not established |
| Post-copy checksum verification: exit 0, empty log | `segments/2026-10-13-124639/terminal.raw` | Original staged source matched the Toshiba destination |
| Fedora staging directory removed; available space 319 -> 582 GiB | `segments/2026-10-13-124639/terminal.raw` | Deletion target absent afterward; storage values from rounded df output |
| Temporary client backup remains on Ventoy | Fedora directory presence/size checks and user confirmation | Retention verified as present, not a new full checksum audit; client handover pending |

## Archive contents reviewed

```text
fix_disk_utility/
├── commands.log
├── terminal.raw
├── timing.log
└── segments/
    ├── 2026-10-12-190705/
    │   ├── terminal.raw
    │   └── timing.log
    ├── 2026-10-12-190826/
    │   ├── terminal.raw
    │   └── timing.log
    └── 2026-10-13-124639/
        ├── terminal.raw
        └── timing.log
```

The first continuation was a recorder-resumption check. The second covers post-CHKDSK fragment checks. The October 13 continuation covers repeated HDD tests, a full read-only scan, a client-backup transfer and verification, and Fedora backup cleanup. The raw archive also includes unrelated personal workstation maintenance; that is intentionally excluded from the public case narrative.

**Editorial note:** The raw terminal capture contains many progress-line updates and terminal escape/control sequences; treat `commands.log` as the navigation index, not as a complete proof of command outcomes. Pair it with the corresponding terminal output and cross-platform transcripts.
