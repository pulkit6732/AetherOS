# uname

## Name

uname - system name; -a adds version, arch and build id

## Synopsis

```
uname [-a]
```

## Description

The build id is the SHA-256 of this kernel's source tree (first 6 bytes).

## Mechanism

Version constant, target arch and the build id from build.rs.

## See also

[ver](ver.md), [about trust](about.md)
