# ls

## Name

ls - list IronVFS files

## Synopsis

```
ls
```

## Description

Lists files in the RAM filesystem with sizes. Use 'vfs-save' to persist them.

## Mechanism

Lists one directory of IronVFS (directories are path prefixes). Marks each
file as local (user, unsigned) or signed system file.

## See also

[cd](cd.md), [cat](cat.md), [mkdir](mkdir.md)
