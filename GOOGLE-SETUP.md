# Connect Drive Cinema to your Google Drive

The APK is built and signed. Google Drive access needs a one-time registration in **your Google Cloud project** before the login button can work. No APK rebuild or client secret is needed.

## Exact values for this APK

- Application type: **Android**
- App name: **Drive Cinema**
- Package name: `uk.co.ryan.drivecinema`
- SHA-1 signing certificate fingerprint:

```text
ED:43:F1:5A:80:21:13:5D:B6:71:CE:C4:CD:23:32:56:AC:7F:DC:CA
```

## Setup

1. Open [Google Cloud Console](https://console.cloud.google.com/) using the account that owns your collection. Create a project, for example **Drive Cinema**.
2. Open **APIs & Services → Library**, search **Google Drive API**, and enable it for that project.
3. Open **Google Auth Platform** and configure the app's **Branding**: name, your support email, and developer contact email. Complete the fields Google requires for your account and chosen audience.
4. Under **Audience**, use **External** for a personal Gmail account. For initial personal testing, keep the project in **Testing** and add your own Google email under **Test users**.
5. Under **Data Access**, add this scope:

   ```text
   https://www.googleapis.com/auth/drive.readonly
   ```

   It is needed to read your existing movie library. This permission grants read access across Drive; the app itself scans only your selected folders. It cannot modify your files.
6. Under **Clients → Create client**, select **Android**, then paste the package name and SHA-1 above. Save. There is no web client, redirect URL, API key or client secret to paste into this app.
7. Install **Drive-Cinema.apk** on an Android 8+ phone/tablet with Google Play Services. Open it and tap **Continue with Google**. Choose the account you added as a test user and grant read-only Drive permission.
8. In settings, choose **Movies** and **TV Shows** folders separately. Navigate inside the desired folder, then press **Use this folder**. Press **Refresh library**.
9. Leave the app open for its first scan. Large collections with many folders can take several minutes. The scan downloads file information, not the full videos.

Suggested layout:

```text
My Drive/
  Movies/
    Film Name (2024)/Film.Name.2024.1080p.mkv
  TV Shows/
    Show Name/
      Season 01/
        Show.Name.S01E01.mkv
        Show.Name.S01E02.mkv
```

Movie files can also sit directly inside Movies. `S01E02` and `1x02` episode names are recognised. Episodes without numbering are grouped by their show folder when possible. Folder shortcuts and shared-drive root browsing are not included.

## Connection help

- **Developer error / error 10 / invalid client:** check the enabled API, exact package name, and exact SHA-1 above. Confirm the OAuth client type is Android.
- **Access blocked:** check the selected Google account is a test user. Workspace administrators can also restrict Drive access.
- **Unverified app warning:** a personally registered app requesting restricted Drive access can show a warning. Check the project name and account match your own registration. Wider distribution requires considering Google's verification requirements.
- **Connection expires in testing:** Google test-user authorizations expire after seven days. Reconnect through Settings. Review Google's audience and verification documentation before changing to a public production audience.
- **Player cannot play a file:** check your connection and file permissions. Drive download quotas and the phone's supported audio/video codecs apply. There is no server transcoding. Embedded subtitles and audio-track selection are available in the player's controls.
- **No posters / old previews:** the app uses Drive thumbnails where available, with generated title cards otherwise. Refresh your library to renew temporary thumbnail links. It does not retrieve film posters, genres or synopses from a movie database.
- **Disconnect:** clears this app's local catalogue and watch history, leaving Drive unchanged. To revoke Google's stored permission too, visit [Google account connections](https://myaccount.google.com/connections).

Google's references: [Android authorization setup](https://developer.android.com/identity/authorization), [Drive media downloads](https://developers.google.com/workspace/drive/api/guides/manage-downloads), [test-user authorization expiry](https://support.google.com/cloud/answer/15549945), [restricted-scope verification and exceptions](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification).
