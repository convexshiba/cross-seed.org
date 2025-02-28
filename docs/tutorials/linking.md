---
id: linking
title: Linking
sidebar_position: 2
---

By default, `cross-seed` will only look for "perfect" matches—candidate file
trees that have the exact same file tree, or a strict subset of the file tree,
of the data you already have. Then it injects the torrents into your client by
mirroring the save path of the torrent you already had.

### What is linking?

When a torrent has the same data as another torrent but a different name, you
could theoretically seed both torrents by copying the file and renaming it, but
this isn't practical—you have to store the data twice even though it's
identical. Linking is an alternative to copying that lets two file names point
to the same underlying data on disk. There are two types of links: _symlinks_
and _hardlinks_. `cross-seed` uses **hardlinks** by default, but you can
[switch to using symlinks](#hardlinks-vs-symlinks) depending on your setup.

### Should I set up linking?

You don't need to set up linking with the default settings. However, you will
need to set up linking in order to enable the more advanced/looser matching
strategies such as [partial matching](partial-matching.md) or
[data-based matching](data-based-matching.md). When linking is enabled, **all**
injected torrents will be linked, even perfect matches.

## Setting up linking

To set up linking, you need to define a directory where `cross-seed` will create
links to your existing files and set the
[`linkDir`](../basics/options.md#linkdir) option to this directory in your
config file. This directory should be accessible to your torrent
client—`cross-seed` will use the `linkDir` as the save path for the torrents it
injects.

```js
module.exports = {
	// ... other settings ...
	linkDir: "/path/to/linkDir",
};
```

**For Docker Users**: there are a few more specific requirements for linking to
work properly.

-   Your torrent client will need access to the `linkDir` you've set, seeing the
    same path `cross-seed` sees.
-   `cross-seed`'s container needs to be able to see the **original data
    files**, again at the same path that your torrent client sees.
-   If you are using **hardlinks**, these paths all need to be _within the same
    docker volume_.

In practice, this means that you should mount a **common ancestor path** of the
both the original data files _and_ your `linkDir`.

Once you have restarted `cross-seed`, new matches will have links created in
your `linkDir` pointing to the original files.

:::info

In order to prevent collisions, `cross-seed` organizes linked torrents into
subfolders of your [`linkDir`](../basics/options.md#linkdir) based on the
tracker each torrent came from. If you wish to disable this behavior, you can
set the [`flatLinking`](../basics/options.md#flatlinking) option to `true`, but
it is not recommended for new users.

:::

## Hardlinks vs Symlinks

By default, `cross-seed` uses hardlinks to link files because they are resilient
to file moves. You can switch to using symlinks by setting the
[`linkType`](../basics/options.md#linktype) option to `symlink`.

### Hardlink

A hardlink is a **direct line** to the actual data on disk, and as far as the OS
is concerned, is indistinguishable from the original file. Because of this,
hardlinks are resilient to being moved, and if you delete the original file, any
other links to the data will remain intact. The underlying data is deleted once
there are zero files left that point to it.

Because hardlinks are implemented at the filesystem layer, they only work within
the same mountpoint, so you can't hardlink a file from an internal drive to an
external drive. This is especially relevant in Docker setups - Docker volumes
are isolated from each other, so **hardlink sources and destinations must always
be within the same Docker volume**, or else linking will fail.

#### When to use hardlinks

-   Your setup moves files around and you don't want to break links
-   You want to keep cross-seeding torrents even after deleting originals
    -   Look into
        [**qbit_manage**](https://github.com/StuffAnThings/qbit_manage) to still
        be able to delete hardlinked files when originals are deleted

### Symlink

A symlink is a **shortcut** that stores a path to the original file. OSes have
special support for symlinks that allow programs (like torrent clients) to treat
them as regular files. Because a symlink itself only contains a file path, if
you move or delete the original file, the link will break and trying to open it
will throw an `ENOENT: no such file or directory` error.

#### When to use symlinks

-   Your setup doesn't move files around
-   You have separate drives (e.g. for leeching vs seeding) and therefore cannot
    use hardlinks
-   You want to stop cross-seeding torrents when originals are deleted, but
    don't use [**qbit_manage**](https://github.com/StuffAnThings/qbit_manage)
    -   This will present as cross-seeded torrents with missing files errors in
        your torrent client which you can then bulk delete
-   You prefer errors over accidentally silently copying data
