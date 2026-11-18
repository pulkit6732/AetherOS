# vfs-save

## Name

vfs-save - flush IronVFS to the disk device

## Synopsis

```
vfs-save
```

## Description

Writes the RAM filesystem to virtio-blk. Without a disk it stays RAM-only.

## Mechanism

Flushes IronVFS to the virtio-blk disk. Needs a virtio disk; VirtualBox
has none, so it reports no device there.

## See also

[ls](ls.md)
