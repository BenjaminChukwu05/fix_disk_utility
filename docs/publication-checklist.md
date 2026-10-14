# Publication checklist

## Do not publish directly

- [ ] The original `fix_disk_utility.zip` archive (raw terminal recordings and timings)
- [ ] Raw `terminal.raw`, `commands.log`, or unreviewed excerpts
- [ ] Original client or personal backup contents
- [ ] Ventoy `VTOYEFI` disk image, first-MiB image, or partition-table image/dump without an explicit need and review
- [ ] Recovery fragment `.CHK` files: unknown content may include personal or copyrighted data
- [ ] Unredacted screenshots containing file names, paths, login names, disk serials, or recovered content

## Sanitize before sharing any log excerpt

- [ ] Usernames, home directories, hostnames, and client names
- [ ] HDD serial numbers, WWNs, filesystem UUIDs, partition UUIDs, and other device identifiers
- [ ] Original BitLocker volume labels and timestamps that could identify its previous owner
- [ ] File/folder names belonging to clients, family, or third parties
- [ ] Terminal command history for passwords, access tokens, cloud endpoints, private addresses, and shell environment values
- [ ] Internal drive details not relevant to the repair
- [ ] Full timing/session metadata if it reveals private operational patterns

## Verify factual wording

- [ ] The short SMART self-test passed; the extended test was **aborted**, not passed
- [ ] Format/repartition did **not** decrypt old contents or securely overwrite the entire HDD
- [ ] The Toshiba HDD remains unsuitable as the sole trusted backup due to SMART and connection history
- [ ] The Fedora check said clean but Windows identified bitmap corruption
- [ ] A successful `rsync` checksum comparison verifies compared bytes, not their preexisting semantic correctness
- [ ] Bink Video detection means recognizable header, not proven playable recovered video
- [ ] Windows `chkdsk` result is from a separate chat transcript, not this Fedora ZIP
- [ ] Ventoy boot functionality is based on the confirmed menu/ISO test, not a captured automated test
- [ ] The HDD's full ddrescue **read** pass did not establish long-term reliability or complete write integrity
- [ ] Four extended SMART tests were **aborted by host**, not passed or confirmed media failures
- [ ] The copied client backup was checksum-verified, but completeness against the original laptop was not established
- [ ] Client handover and a dependable additional backup remain outstanding; the Ventoy copy is temporary
- [ ] The Fedora backup was removed only after the other copies were checked; the released-space figure comes from rounded filesystem accounting

## Suggested repository layout

```text
fix_disk_utility/
├── README.md
├── .gitignore
└── docs/
    ├── evidence.md
    ├── selected-evidence.md
    └── publication-checklist.md
```

Keep the unredacted recordings and backup content outside the Git working tree. If publishing supporting proof, build carefully selected, manually verified, sanitized excerpts rather than uploading the original ZIP.
