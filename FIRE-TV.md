# Zflix TV on a Fire TV Stick

`Zflix-TV.apk` is a separate, lightweight build for Fire TV and Android TV. It is 7 MB, uses plain native screens driven by the remote, and needs no Google Play services.

## What the Fire TV Stick 2nd gen can and cannot play

From Amazon's developer specifications: MediaTek 8127D quad-core 1.3 GHz, 32-bit, 1 GB RAM, 8 GB storage, Fire OS 5 (Android 5.1), output up to 1080p.

| Video | On this stick |
|---|---|
| H.264 up to 1080p | Plays |
| HEVC (x265) 8-bit up to 1080p | Plays |
| HEVC 10-bit, HDR, Dolby Vision | No decoder. Will not play |
| 4K / 2160p of any kind | No decoder. Will not play |
| Audio: AAC, AC3, E-AC3 natively; DTS, TrueHD, FLAC through the bundled FFmpeg decoder | Plays |

There is no transcoding, so a 4K or 10-bit file cannot be converted on the fly. The app reads the stick's real decoder list at start, marks titles it cannot decode "NOT ON THIS TV", and picks a playable version automatically when a film is stored twice. In your library today that means about 30 of 63 films and all but one TV episode play on the stick; 24 films are 4K-only and 9 are 10-bit.

## Install

1. On the stick: Settings › My Fire TV › Developer options › turn on **ADB debugging** and **Apps from Unknown Sources**. (If Developer options is hidden: Settings › My Fire TV › About, press Select on the device name seven times.)
2. Find the stick's address: Settings › My Fire TV › About › Network.
3. From the laptop, the Pi can install it over Wi-Fi:

```bash
ssh pi 'adb connect STICK-IP:5555 && adb -s STICK-IP:5555 install -r ~/drive-cinema/Zflix-TV.apk'
```

Accept the "Allow USB debugging?" prompt on the TV the first time. Alternative without ADB: install the Downloader app on the stick and fetch the APK from any web address you host it at.

## Link it to your Google Drive (one time)

Fire TV has no Google sign-in, so the phone app signs in once and gives the TV its own key.

1. In Google Cloud Console, same project as before: Google Auth Platform › Clients › Create client › type **Desktop app** › Create. Copy the Client ID and Client secret.
2. Recommended: Google Auth Platform › Audience › **Publish app**. While the project is in Testing, Google ends the TV's sign-in every 7 days and you would have to link again.
3. Open Zflix TV on the stick. It shows a 6-digit code.
4. On the phone: Zflix › More › **Watch on a TV**. Paste the Client ID and secret, tap **Find my TV**, type the code, tap **Link TV**. Approve access in the browser page that opens, then return to the app.
5. The TV receives the sign-in, your folders, the library and all artwork data, and opens the home screen.

## Remote control

- Home: D-pad to browse, Select to open a title, Menu (☰) for Settings, Back to go back.
- Player: Select or Play/Pause toggles playback. Left/Right skip (10 s steps that grow while you keep pressing; the seek happens when you stop). Down or Menu opens audio track, subtitles, subtitle size, next episode, speed, zoom and playback info. Back exits and saves your place.

## Tuned for a 1 GB device

- 48 MB memory buffer at most; the deep buffer is on storage (512 MB read-ahead by default, 1 GB cache), fetched over 3 parallel connections.
- Artwork is decoded at the size shown, in 16-bit colour, with a small memory cache.
- FFmpeg is used for audio only; software video decoding is disabled because it would be a slideshow on this CPU.
- No WebView, no background services, no downloads feature.

## If something goes wrong

Settings shows the last crash, and the TV serves a read-only status page while the app is open:

```bash
curl http://STICK-IP:8822/diag
```

It reports the device, decoder list, memory, cache, connection speed, recent log lines and the last crash. It contains no sign-in data.
