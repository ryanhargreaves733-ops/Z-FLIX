# Zflix

A private Netflix-style Android app that streams the films and TV shows already stored in your Google Drive. No server, no Jellyfin, no torrents: the phone talks to Google Drive directly and plays the files itself.

**Install:** `Zflix.apk` for phones and tablets (Android 8 or newer with Google Play services) and `Zflix-TV.apk` for Fire TV / Android TV (Android 5.1 or newer, see [docs/FIRE-TV.md](docs/FIRE-TV.md)). Both are in `~/APKs`. It updates in place over earlier versions because it keeps the same package name (`uk.co.ryan.drivecinema`) and signing key.

## Project layout

| Folder | What is in it |
| --- | --- |
| `app/` | Android apps: `phone` (WebView UI) and `tv` (native remote UI) sharing one engine in `app/src/main` |
| `linux/` | Native desktop app for Linux (C++, Qt 6, libmpv). See `linux/README.md` |
| `server/` | Hosted web version in Docker, built from the same core as `linux/`. See `server/README.md` |
| `docs/` | Setup guides: Google Cloud registration, Fire TV, validation notes |
| `assets/` | Logo and artwork |
| `tests/`, `preview/` | Unit tests and the browser preview used while designing the phone UI |
| `build-pi.sh`, `signing/` | Android build on the Raspberry Pi and the release key (kept out of any share) |

Finished Android builds are kept in `~/APKs`.

## What it does

- **Library from your folders.** Pick a Movies folder and a TV Shows folder in Settings. The scan reads your whole folder tree with a handful of batched Drive calls (about 20 seconds for 160 folders). It understands `Movies / Collection / Film (Year) / file`, loose files, several rips of the same film (shown as versions), extras folders, samples, `Show / Season 01 / S01E02` layouts, specials, multi-episode files, dated episodes and absolute numbering. Subtitle files next to a video (or in a `Subs` folder) are offered in the player. Artwork files (`poster.jpg`, `fanart.jpg`, `clearlogo.png`, season posters, episode thumbs) are used when present.
- **Metadata.** Paste a free TMDB API key in Settings and Zflix fetches posters, backdrops, title logos, synopses, genres, ratings, certificates, cast, trailers, and episode names and stills. Radarr, Sonarr, Kodi and Plex metadata files (`.nfo`, `.plexmatch`) are read first, so matches are exact when those exist. Wrong match? Open the title and tap "Wrong match? Fix it".
- **Netflix-style UI.** Profiles ("Who's watching?", kids profiles that hide 15/18/R titles), billboard, Continue Watching, My List, Recently Added, Top 10, Collections, genre rows, "Because you watched", TV Shows and Movies rows, 4K & HDR, Watch it again. Detail sheets with seasons, episode stills, cast and "More like this". Search across titles, people and genres. Downloads page. Works on phones, tablets, and Android/Google TV boxes with Play services (leanback launcher entry included).
- **Player built for mobile connections.** The film is fetched by a read-ahead engine: several workers, each on its own connection to Google Drive (four by default), download into a disk cache, first what the player needs now and then up to 2 GB ahead (configurable to 16 GB). The player only reads from that cache, so the connections' speeds add up and dips are absorbed by what is already on disk. After a seek the engine drops what it was fetching and turns every connection to the new position. If the line is slower than the film's bitrate, playback pauses once and loads a stretch ahead (25 s, then 60 s, 2 min, 4 min if it keeps happening) instead of stuttering; Play overrides it. Lost connections retry on their own. FFmpeg decoders handle AC3, E-AC3, DTS, DTS-HD, TrueHD, FLAC and legacy codecs. Resume is saved every 5 s. Controls: double-tap to skip 10 s, swipe for brightness and volume, speed, audio and subtitle picker (embedded and sidecar), subtitle size, zoom, next-episode countdown, picture-in-picture, "Open in another player" through a built-in loopback relay.
- **Downloads.** Save any film or episode to the device with progress, speed and ETA; pause, resume, delete. Downloads fetch four ranges at once on separate connections (measured 1.6× faster than one). Downloads pause while something streams so they do not fight for bandwidth.
- **Keeps itself fresh.** On launch it asks Drive's change feed whether anything in the library folders changed and refreshes quietly if so.

## Folder layout it expects

```
Movies/
  Film Name (2019)/Film.Name.2019.1080p.mkv          one film per folder
  Another Film (2021).mkv                            loose files are fine too
  Fast And Furious/                                  a collection: a folder of film folders
    The Fast and the Furious (2001)/....mkv
    2 Fast 2 Furious (2003)/....mkv
TV Shows/
  Show Name (2015)/
    Season 01/Show.Name.S01E01.mkv
    Specials/Show.Name.S00E01.mkv
```

Extras folders (Featurettes, Behind the Scenes, Trailers, Sample, Screens, Subs…) are recognised. The Library report in Settings lists everything the scan skipped and any folder with files but no recognised video.

## Settings worth knowing

- **Streaming buffer** Balanced / Large / Maximum: memory buffer depth.
- **Read ahead on disk** Off … 16 GB: how far ahead of playback to download. Keeps filling while paused.
- **Disk cache for streaming** 2–32 GB: must be larger than the read-ahead; applies after restart.
- **Connections** 1–6: parallel connections to Drive. On the test line one connection gave about 6 Mbps and four gave 11–15 Mbps.
- **Watch on a TV**: links a Fire TV / Android TV running Zflix TV.
- **Lights that follow the film (Home Assistant)**: sign in to Home Assistant from inside the app (its own login page, or paste a long-lived token), pick lights and give each the part of the picture it sits by (whole, left/right half, top/bottom half, or one of the four corners), switch on *Lights follow the film*. The player copies a 48×27 thumbnail of the video a few times a second, finds the dominant colour (bright, saturated pixels count most; black bars count for nothing) and measures loudness on the audio path, then drives the lights over the Home Assistant REST API: `scene.create` snapshots them first, `light.turn_on` with `rgb_color`/`brightness`/`transition` follows the picture, a loud bang flashes them, and pause/stop puts the snapshot back. Linking the TV copies the connection and the chosen lights to it. Works on Android 7+ through PixelCopy; on Fire OS 5 the picture is routed through a TextureView instead. Nothing leaves the Wi-Fi.
- **Connection check** measures what Drive delivers to the phone right now.

No setting can play a file faster than the line delivers it: a 27 GB two-hour film averages 25 Mbps, so on a line giving 12 Mbps it has to load ahead or be downloaded first.

## Build (Raspberry Pi)

Project path `/home/ryan/drive-cinema` on both the laptop and the Pi. One Gradle project, two flavors sharing the engine: `phone` (WebView UI, Play services sign-in, media3 1.11.1, Android 8+) and `tv` (native remote UI, refresh-token sign-in, media3 1.7.1 because newer releases need Android 6, Android 5.1+, 32-bit ARM only).

```bash
rsync -a --delete --exclude build --exclude .gradle --exclude preview --exclude '*.apk' --exclude '*.log' --exclude local.properties --exclude signing /home/ryan/drive-cinema/ pi:/home/ryan/drive-cinema/
```

```bash
ssh pi 'cd ~/drive-cinema && ./build-pi.sh all'
```

`./build-pi.sh phone` or `tv` builds one. Outputs `Zflix.apk` and `Zflix-TV.apk`, each compiled, linted against its minimum Android version and signed with `signing/release.keystore` (keep it: the phone's Google sign-in is tied to its fingerprint). Parser tests: `tests/NamesTest.java`. UI preview with fictional data: `python3 preview/server.py 8765`.

adb hooks for the phone app: `am start -n uk.co.ryan.drivecinema/.MainActivity --ez sync true` (refresh library), `--ez dump true` (catalogue to the app's external cache dir), `--es play <driveFileId>`, `--ez speedtest true` (1/2/4/6 connections), `--ez ptest true` (parallel downloader correctness). Ask before using them: they take over the phone's screen.

## Source map

Shared (`app/src/main`):
- `Names.java` filename/folder parsing. `Scanner.java` Drive tree to movies, shows, collections, versions, subtitles, report.
- `Drive.java` authenticated HTTP clients (one multiplexed, one that opens a TCP connection per request), listing, changes feed, speed tests. `Store.java` catalogue, metadata, profiles, watch state, settings. `Tmdb.java`, `ImageCache.java` metadata and artwork.
- `Prefetcher.java` the multi-connection read-ahead engine. `FedDataSource.java` the cache-only source the player reads. `PlaybackEvictor.java` cache eviction that protects what is ahead of the viewer. `ParallelDataSource.java` multi-range reader used by downloads. `CinemaMedia.java` caches, buffer policy sized to the device, decoders, downloads.
- `PlayerScreen.java` the player (touch and remote modes). `Playback.java` file-id to playable item, next episode. `LinkCrypto.java` sealing for phone-to-TV linking. `Diag.java` rolling log and crash capture. `LocalProxy.java`, `OfflineService.java`.

Phone (`app/src/phone`): `MainActivity.java` web UI host and bridge, `Auth.java` Play services tokens, `TvLink.java` TV discovery and linking, `assets/` the UI.

TV (`app/src/tv`): `TvActivity.java` all screens, `Catalog.java` view model, `Images.java` loader, `DeviceCaps.java` decoder detection, `LinkServer.java` link and diagnostics endpoint, `Auth.java` refresh-token sign-in.

Google Cloud registration (package, SHA-1, scope) is unchanged from before: see `docs/GOOGLE-SETUP.md`.
