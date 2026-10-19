# Soulseek De-Duplicator for Nicotine+ — v0.1.1
A special designed and reliable tool for finding duplicated files, and removing them safely using Nicotine+ plugin system.

It runs fluently using low priority threads. Granted, you can still use Nicotine+ while it scans (a ~5TB library can take 2-3 hours depending on speed of system.)

Simply install plugin via Nicotine's Settings and install contained ZIP file. 

Send yourself a message on Nicotine+ for commands to work in private as a pro-tip. (although they should work fine in any input buffer like chat, etc.)

AI generated for this codebase, I decided to do this way for simplicity sake, and it doesn't delete files AT ALL but outputs it as CSV file once finished (`%APPDATA%\nicotine`)


OFFICIAL README.MD:

A local, report-only duplicate finder for your configured Nicotine+ shares.

## Safety

This version NEVER deletes, moves, renames, or edits your library files.
A file is only reported as a duplicate after a complete-file BLAKE hash matches.

## Pipeline

1. Enumerate configured public/buddy/trusted share roots.
2. Group files by exact byte size.
3. Partial-hash only same-size candidates:
   - beginning sample
   - middle sample
   - ending sample
4. Full-hash only matching partial fingerprints.
5. Write verified duplicate groups to a TSV report.
6. Cache hashes in SQLite using path + size + mtime_ns + hash settings.

Unchanged files are therefore very cheap on later scans.

## Hash algorithm

If Python can import `blake3`, the plugin uses BLAKE3.
Otherwise it automatically uses standard-library BLAKE2b.

The fallback requires no third-party package.

## Commands

    /dedupe scan
    /dedupe status
    /dedupe report
    /dedupe clear-cache
    /dedupe help

The scan runs on a background thread so the Nicotine+ UI remains responsive.
Progress and final results are written to the Nicotine+ log.

## Install

Nicotine+ 3.3.x:

1. Open Preferences -> Plugins.
2. Use the plugin install action and select the ZIP.
3. Enable "Soulseek De-Duplicator".
4. Run `/dedupe scan` from a command-capable Nicotine+ interface.

You can also extract the `soulseek_deduplicator` folder into the Nicotine+
user plugin directory. On Windows, Nicotine+ stores its user data beneath
`%APPDATA%\nicotine`.

## Reports

Reports are created inside this plugin's folder as:

    dedupe-report-YYYYMMDD-HHMMSS.tsv

Columns:

    group | size_bytes | full_hash | copy | path

`copy=1` is not automatically a "keeper"; it is merely the first sorted path.
Review groups manually before taking any filesystem action.

## Notes

- Hard links to the same filesystem inode are ignored as duplicate storage.
- Symlinks are ignored by default.
- Missing/unreadable files are logged and skipped.
- Partial hashes are only a filter. Full-file hashing is mandatory before
  a duplicate group is reported.

## v0.1.1

Fixed Nicotine+ 3.3.x command callback binding (`callback(args)`).

## v0.1.2

Fixed command callbacks invoked from private chat. Nicotine+ supplies command
context as keyword arguments (for example `user=...`), so `_command` now accepts
`user`, `room`, and future extra keyword arguments.
