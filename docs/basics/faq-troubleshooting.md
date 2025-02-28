---
id: faq-troubleshooting
title: FAQ & Troubleshooting
sidebar_position: 4
---

### Updating to cross-seed v6

If you are updating from version 5.x to version 6.x, you can visit the
[v6 migration guide](../v6-migration) for the changes and updates.

### General Suggestions

-   The log files in `cross-seed`'s `logs` folder—specifically
    `verbose.*.log`—are your friend. If you experience undesired results from
    `cross-seed`, look to these files first for indicators of why `cross-seed`
    performed the way it did.
-   In the past, `cross-seed` has been less opinionated about what settings work
    well. After a few years of experimentation, we have settled on a
    high-performing set of default settings that should work well for users and
    trackers alike. If your configuration predates v6, consider adjusting your
    settings to match the
    [defaults](https://github.com/cross-seed/cross-seed/blob/master/src/config.template.cjs).

### Does cross-seed support music torrents?

While `cross-seed` may incidentally find _some_ music matches, it is not
optimized for music _at all_. You will find better results with tools designed
for music cross-seeding like
[fertilizer](https://github.com/moleculekayak/fertilizer).

:::tip

Look in your music trackers' wikis and forums for more resources on
cross-seeding music.

:::

### What can I do about `error parsing torrent at http://…`?

This means that the Prowlarr/Jackett download link didn't resolve to a torrent
file. You may have been rate-limited, so you might want to try again later.
Otherwise, just ignore it. There's nothing `cross-seed` will be able to do to
fix it.

### Does cross-seed work on public trackers?

Don't add public trackers to your cross-seed config (i.e. as a
[torznab URL](options.md#torznab)).

Public torrents on multiple trackers usually all have the same infoHash, which
is different from private torrents. Because of this, `cross-seed` does not
consider these torrents matches because it looks like you already have them.

In theory, we could add the announce urls to the torrent already in your client,
but public torrents make heavy use of
[DHT](https://en.wikipedia.org/wiki/Distributed_hash_table) and
[PEX](https://en.wikipedia.org/wiki/Peer_exchange), which are decentralized
mechanisms for finding peers to seed to. Therefore, the role of a tracker in a
public torrent is somewhat redundant and new trackers usually won't give you new
peers.

However, it's fine to **download something from a public tracker and then
cross-seed it to private trackers**, which `cross-seed` supports automatically.

### What's the best way to add new trackers?

Just add the [`torznab` url](./options.md#torznab) to your config. `cross-seed`
will automatically queue searches for your catalog on just the one indexer.

### What should I do after updating my config?

Just restart `cross-seed`.

If you've changed your eligibility filters or your
[`matchMode`](options.md#matchmode) (e.g. enabling partial matches),
`cross-seed` will **automatically** re-evaluate previous cached rejections.

:::caution

It will **never** be necessary to delete your database or `torrent_cache` folder
to perform a "fresh search". Doing so only puts undue stress on indexers.

:::

### Can I use special characters in my URLs?

To use special characters (`!@#$%^&*`) in your URLs for user/passwords, you will
have to use URL encoding for ONLY the portion of the URL containing the special
characters.

For instance, if your user or password contains a special character `#`, you
will need to go to [URLEncoder.org](https://www.urlencoder.org/) and encode
**THIS PORTION OF YOUR URL ONLY**!

Replace the original user or password with the encoded one in your config file.

### My cross-seeds are paused at 100% after injection

This is by design and due to the way data-based searches function. This is done
to prevent automatically downloading missing files or files that failed to
recheck. Until there is a way to guarantee you won't end up downloading extra
(not cross-seeding) this is the best solution.

Alternatively, if you wish for torrents to not inject in a paused state, you can
enable `skipRecheck` in the config file, and this will instead Error on
missing/incomplete files.

:::tip

Consider using `infoHash` rather than `path` in calls to `/api/webhook` to
prevent mismatches if you are racing.

:::

### Why do I get `Unsupported: magnet link detected at…`?

`cross-seed` does not support magnet links. If your indexer supports torrent
files you will need to switch your settings.

### rtorrent injected torrents don't check (or start at all) until force rechecked

Remove any `stop_untied=` schedules from your .rtorrent.rc.

Commonly:
`schedule2 = untied_directory, 5, 5, (cat,"stop_untied=",(cfg.watch),"*.torrent")`

### `SyntaxError: Unexpected identifier` when I try and start cross-seed

`cross-seed` configuration files are formatted with commas at the end of each
identifier (option), if you are seeing this error it is most likely that you are
missing a comma at the end of the setting BEFORE the identifier specified in the
error.

Alternatively, you could also be missing quotes around the value you provided.
Check the syntax before and around the config setting given in the error.

### `RangeError: WebAssembly.instantiate(): Out of memory` error when starting

If you receive this error when trying to start `cross-seed`, usually presenting
in shared seedbox environments, it is likely caused by a limitation in the
virtual memory able to be allocated to your instance of `cross-seed`.

To fix this error, it is necessary to use at least Node v20.15 and set the
`NODE_OPTIONS` environment variable to include the
[`--disable-wasm-trap-handler`.](https://nodejs.org/en/blog/release/v20.15.0#cli-allow-running-wasm-in-limited-vmem-with---disable-wasm-trap-handler)
flag.

To ensure a successful startup of cross-seed, you can simply execute cross-seed
with the following command.

```shell
NODE_OPTIONS=--disable-wasm-trap-handler cross-seed daemon
```

### Failed to inject, saving instead.

The best way to start troubleshooting this is to check the `logs/verbose.*.log`
and find this specific event.

You will be able to see the circumstances around the failure and start
investigating why this occurred.

### My data-based search is searching torrent files!

Torrent files in [`torrentDir`](./options.md#torrentdir) take precedence over
data files with the same name and are de-duplicated.

If possible, `cross-seed` will always attempt to create links for data and
torrents being injected into a client, so the end result will generally be the
same.

If you wish to search only for data files, you can set
[**`torrentDir`**](options.md#torrentdir) to `null`, but we generally don't
recommend it as torrent-based matching is more efficient than data-based
matching. RSS/announce is also only supported with torrent-based matching.

### Error: ENOENT: no such file or directory

This is usually caused by broken symlinks in, or files being moved or deleted
from, your [`dataDirs`](./options.md#datadirs) during a data-based search. Check
the `/logs/verbose.*.log` files for the file causing this.

:::caution

If you do not link files within your `dataDirs` or have them outside of the
[`maxDataDepth`](../tutorials/data-based-matching.md#setup) visibility, this is
preventable.

:::

### My torrents are injected (qBittorrent) but show `missing files` error!

If you see injected torrents show up with a `missing files` error, it is likely
because you do not have a save path set for the original torrent's category that
`cross-seed` was cross-seeding from. You should also see a warning in the logs
when the torrents are injected telling you there is no save path set.

:::tip

Set a save path for the files in that category to fix this.

:::

### I am getting `0 torrents found` with qBittorrent and I set my `torrentDir`

If you are using qBittorrent 4.6.x, 5.x, or later, and have the option to use
`SQLite database` in the Advanced menu in preferences, you will not be able to
do torrent based searches. To be compatible with `cross-seed` you will need to
switch this to `fastresume` mode and restart qBittorrent. This will store actual
torrent files in your BT_Backup folder. `cross-seed` depends on, and indexes,
the torrent files for searching.

We have no current ETA on integration with qBittorrent's `SQLite database`
storage mode.

### I'm getting errors in cross-seed on hostingby.design seedbox

1. Get your shared instance IP address (`cat .install/subnet.lock)
2. Torznab URLs should be: `http://ip:port/prowlarr/[id]/api?apikey=[apikey]`
3. Your torrent client should be on the same subnet (if installed after dec.
   2023). If not, update the “bind ip” address in web ui settings to your ip
   from step 1 and restart the torrent client
4. The torrent client address URL in `cross-seed` config should be
   `http://user:pass@ip:port` (note: no `/qbittorrent` or `/deluge` etc at the
   end)

:::caution

The subnet/shared instances are reverse proxies so no https

:::

:::tip

Although we are providing the information given from hostingby.design, we are
not able to support this directly. This is just what was passed to us. Contact
them for details and further instructions if you have any issues

:::

### I can't reach my Prowlarr/Jackett/cross-seed/torrent client (Using Docker/VPN)

If you are using [Docker](./getting-started#with-docker), you cannot use
`localhost` as an address to communicate across containers. Consider using your
host's local IP address (usually a 192 or 10 address) or the container name (if
using a "Custom Docker Network").

If your setup is running with a VPN, you will need to either

1. Set up split tunneling
2. Try to use the Docker network addresses as your instance hostnames
3. Address the routing of local/internal traffic in your VPN configuration

Generally, you won't be able to access local instances from services utilizing a
VPN since localhost/LAN is not accessible from the VPN network by default.

:::info

There is no need to put `cross-seed` behind a VPN. All of its requests are made
to your torrent client or Jackett/Prowlarr. The only exception is when
announcements are made via the `/announce` API endpoint and are snatched during
matching or for injection.

:::

:::danger

Even with API authentication, we still recommend that you **do not expose its
port to untrusted networks (such as the Internet).**

:::

### Searching media libraries vs. torrent data (data-based searching)

You can search both your media libraries (Arr/Plex) and actual torrent data
(downloaded files). If you are using the media libraries with renamed files, you
will need to use [`matchMode: "risky"`](../basics/options.md#matchmode) in your
configuration file to allow `cross-seed` some leeway in its matching process.
[`"risky" matchMode`](./options.md#matchmode) is not recommended to be used
without [`skipRecheck`](./options.md#skiprecheck) being set to false, as it
could result in more false positives than `"safe"`.

:::caution

Due to the way data-based searching works, risky matching only matches renamed
files if they are a single-file searches. As TV libraries usually include
renamed files, data-based matching will not be able to pick up matches on
multi-file torrents (such as full-season packs matched to your season folders).

:::

### Why do I see `it has a different file tree` in my logs?

This is a result of the matching algorithm used by `cross-seed`, and is most
commonly associated with only a few scenarios. These include the presence of
additional .nfo/.srt files in a torrent, differences in the organization of
files (one torrent having a folder while the potential match does not, or vice
versa), and discrepancies in the filenames within the torrent. Setting up
[Partial Matching](../tutorials/partial-matching.md) will ignore these
extraneous small files during matching, and improve your match rate at the cost
of redownloading some of these small files.

:::tip

You can use the [`cross-seed diff`](../reference/utils#cross-seed-diff) command
to compare two torrent files and see exactly how they differ.

:::

### How can I use [**autobrr**](https://autobrr.com/) with cross-seed?

If you are using [**autobrr**](https://autobrr.com/) to cross-seed, you can use
the [`/api/announce`](../reference/api#post-apiannounce) endpoint, rather than
[`/api/webhook`](../reference/api#post-apiwebhook), to match against what
`cross-seed`
[already knows about your available media](../reference/architecture#prefiltering)
(instead of searching your indexers every time).

:::tip

If you want to filter announces even further, consider setting up more specific
filters or using [**omegabrr**](https://github.com/autobrr/omegabrr) (which
filters based on monitored items in Arrs) to minimize needless calls to
cross-seed.

:::

:::info

For more help setting this up, you can head over to the
[autobrr documentation for 3rd-party-tools](https://autobrr.com/3rd-party-tools/cross-seed#cross-seed-filter).
:::

### My tracker is mad at me for snatching too many .torrent files!

In order to prevent false positives, `cross-seed` snatches some torrent files
that may not ultimately match your owned torrents. It preserves every torrent
file that it snatches.

We have spoken to admins at several trackers and have settled on a set of
default settings that these trackers prefer. If your tracker has reached out to
you about your usage of cross-seed, consider reverting to the default settings
of [`excludeOlder`](options.md#excludeolder),
[`excludeRecentSearch`](options.md#excluderecentsearch),
[`searchCadence`](options.md#searchcadence), and
[`searchLimit`](options.md#searchLimit).

:::caution

Your tracker may have opinions on `cross-seed`'s behavior regarding snatches,
and it is important to respect these or risk your account being disabled.

:::

`cross-seed` reduces unnecessary snatches of .torrent files as much as possible,
but because it needs to compare the files inside the torrent as well as their
sizes, it is sometimes unavoidable. To improve matching efficiency, **consider
setting up [ID-based searching](../tutorials/id-searching.md)**, which searches
based on IMDb IDs and often results in less strain on trackers, more specific
search results, and less unnecessary snatches.

### My partial matches from related searches are missing the same data, how can I only download it once?

`cross-seed` is not aware of what matches will happen ahead of time, each is
performed with zero knowledge of the previous or the following. As such it
possible to have situations where a partial match when complete would become a
perfect match for another otherwise partial match. This is usually neglibile
since the missing data is small, but in cases where it is significant such as
with [seasonFromEpisodes](./options.md#seasonfromepisodes), you can use the
[inject](../reference/utils.md#cross-seed-inject) feature.

If you have not recently deleted files in your
[outputDir](./options.md#outputDir), then these torrents will still have their
.torrent file present. If so, simply pick one torrent to complete the download
on and that's it! `cross-seed` uses all possibles matches to source files with
the inject job or when using `cross-seed inject`. It will automatically detect
the newly downloaded files, link them to the other torrents, and trigger a
recheck for those torrents.

For the rare case that the .torrent files are not present in `outputDir`. First,
choose which tracker you'd like to complete the download on and start the
torrent. Then copy (or export) the .torrent files from the other related partial
matches to a safe place. Once the download is complete, follow the steps
[here](../tutorials/injection.md#manual-or-scheduled-injection) to re-inject the
copied and renamed .torrent files. `cross-seed` will automatically use the newly
completed torrent over the previous (or ensemble) that caused the partial match.

:::danger

WARNING Manual injections such as what is performed here requires the renaming
of .torrent files for proper linking.

[**Read More**](../tutorials/injection.md#manual-or-scheduled-injection)

:::

### How can I force inject a cross seed if its source is incomplete?

`cross-seed` will not inject matches if the source is incomplete in most
scenarios. The only time `cross-seed` will, is from the `cross-seed inject`
command or from the inject job. If the age of the saved .torrent file
(determined my last modified time) is older than a day, `cross-seed` will inject
and link files even from an incomplete source if
[`linkDir`](./options.md#linkdir) and [`partial`](./options.md#matchmode) is
enabled. These injections will always be treated as `partial` matches which are
paused and rechecked.

Once the torrent has finished rechecking, it's progress could be anywhere from
0% to 100%. You will then have to choose between resuming the torrent or
removing it from client with the corresponding .torrent file in
[`outputDir`](./options.md#outputdir). Of course this is no longer "cross
seeding", but it does allows the possibility of reviving the stalled torrent
used as the source. In order to do this, you may need to follow the manual steps
outlined in the
[faq entry](#my-partial-matches-from-related-searches-are-missing-the-same-data-how-can-i-only-download-it-once)
above on the stalled torrent.

:::caution

This is not really "cross seeding" anymore and likely will incur significant
download for the matched torrent. This should be rarely needed as `cross-seed`
can match from multiple sources. `cross-seed` is simply offering you the ability
to revive the stalled torrents inside your client.

:::
