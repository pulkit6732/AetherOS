# mv

## Name

mv - move a local file within /home

## Synopsis

```
mv <src> <dst>
```

## Description

Signed system files cannot be moved.

## Mechanism

Copy then delete; IronVFS has no rename. Local files only.

## See also

[cp](cp.md), [rm](rm.md)
