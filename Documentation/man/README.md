# Command manual pages

Generated from `kernel/src/shell/mod.rs` by `tools/gen-manpages.py`.

## System

| Command | Summary |
|---|---|
| [about](about.md) | what this build is and its known limits |
| [clear](clear.md) | clear the screen |
| [help](help.md) | list commands, or explain one |
| [history](history.md) | last 16 commands (Up/Down arrows recall them) |
| [lang](lang.md) | change the shell's language |
| [license](license.md) | licences of AetherOS and the third-party crypto it uses |
| [man](man.md) | full manual page for a command (same as help <command>) |
| [ps](ps.md) | list processes |
| [reboot](reboot.md) | restart the machine |
| [shutdown](shutdown.md) | power off (QEMU, VirtualBox) |
| [status](status.md) | live system summary |
| [uname](uname.md) | system name; -a adds version, arch and build id |
| [uptime](uptime.md) | time since power-on |
| [ver](ver.md) | version, build and boot-check totals |

## Files

| Command | Summary |
|---|---|
| [cat](cat.md) | print a file |
| [cd](cd.md) | change directory (no argument: /home) |
| [cp](cp.md) | copy a file into /home |
| [echo](echo.md) | print text |
| [ls](ls.md) | list IronVFS files |
| [mkdir](mkdir.md) | create a directory under /home |
| [mv](mv.md) | move a local file within /home |
| [pwd](pwd.md) | print the current directory |
| [rm](rm.md) | remove a local file or empty directory |
| [touch](touch.md) | create an empty file under /home |
| [vfs-save](vfs-save.md) | flush IronVFS to the disk device |

## Programs and receipts

| Command | Summary |
|---|---|
| [apps](apps.md) | list the signed apps shipped in this build |
| [receipt](receipt.md) | make a DEMO receipt (pid 1..999, default 1) |
| [receipts](receipts.md) | receipt summary; export prints them for a verifier |
| [run](run.md) | run an app: verify, translate, execute, receipt |
| [tamper-probe](tamper-probe.md) | flip one byte and show it is rejected |
| [verify-receipt](verify-receipt.md) | check a saved receipt's signature |

## Diagnostics

| Command | Summary |
|---|---|
| [mem](mem.md) | memory layout and live allocator state |
| [monitor](monitor.md) | AetherMon performance counters |
| [stress](stress.md) | run an n-iteration stress test (default 1000) |
