# Installing the apps

There is one Android app and it covers both phones and televisions. The same
file installs on either; the app works out which one it is running on and shows
the appropriate interface.

You can also just use a web browser, in which case there is nothing to install —
sign in at [api.spameri.cz](https://api.spameri.cz) and press play. The browser
always fetches video through the server, even at home, so the app is the better
option on a television or a phone you use at home a lot.

## Where to get the file

The app is not on Google Play. You download it from the releases page and
install it yourself:

**<https://github.com/Spameri/MediaLibDocumentation/releases/latest>**

Look for the file ending in `.apk` — it is named after the version, for example
`medialib-0.2.0.apk`.

Android 6.0 or newer is required. Anything from the last decade qualifies.

## On a phone

1. Open the releases page in your phone's browser and tap the `.apk` file.
2. Your browser will warn you that this kind of file can harm your device.
   Accept the download.
3. Open the downloaded file. Android will say the browser is not allowed to
   install unknown apps, with a button to the setting.
4. Turn on **Allow from this source** for your browser, then press back and
   install.
5. Open the app and sign in with your email address and password.

If you never see the prompt in step 3, the setting is at **Settings → Apps →
Special app access → Install unknown apps**. Find the browser you downloaded
with and allow it. You can turn it off again afterwards.

Once you are signed in, pick a profile and you are done. The app remembers you;
you will not be asked again unless you sign out.

## On an Android TV

Televisions do not have a browser you can download files with, so there are
three ways in. Pick whichever suits what you have.

First, turn on installing from unknown sources — you will need it for any of the
three. The path is usually **Settings → Device Preferences → Security &
restrictions → Unknown sources**. You enable it per app, so enable it for the
app you are about to install with (Downloader, or your file manager). Some
televisions only offer the toggle after the app has tried to install something
once and been refused, so if you do not see your app listed, try the install
first and come back.

### Option 1: the Downloader app

This is the least painful route if your television has the Google Play Store.

1. Install **Downloader** by AFTVnews from the Play Store on the television.
2. Open it and type this into the URL box:

   ```
   github.com/Spameri/MediaLibDocumentation/releases/latest
   ```

3. Downloader opens the page in its own browser. Scroll to the assets list and
   select the `.apk` file. It downloads and offers to install.
4. If the install is refused, enable unknown sources for Downloader as described
   above, then press install again.

Typing that URL on a remote is tedious but you only do it once. If whoever
invited you has set up a shorter link, use that instead.

### Option 2: ADB over the network

This is the quickest route if you have a computer to hand and do not mind a
terminal. Nothing needs to be plugged in — it works over your home network.

1. On the television, go to **Settings → Device Preferences → About** and select
   **Build** seven times. You will see "You are now a developer".
2. Go to **Settings → Device Preferences → Developer options** and turn on
   **USB debugging**. Some televisions have a separate **Network debugging** or
   **ADB debugging** switch — turn that on as well if present.
3. Find the television's address under **Settings → Network & Internet**, next to
   your Wi-Fi name.
4. On your computer, with `adb` installed (part of Android platform-tools):

   ```
   adb connect 192.168.1.60:5555
   adb install medialib-0.2.0.apk
   ```

   Use the television's own address and the filename you actually downloaded.

5. The television shows a dialog asking whether to allow debugging from this
   computer. Accept it, then run the `adb install` again if it timed out.
6. `Success` means it is on. Turn USB debugging back off when you are finished.

The app appears in the television's app list with its own banner. On some
launchers new sideloaded apps land in a separate "Apps from unknown sources" row
rather than the main one.

### Option 3: a USB stick

Works on any television, needs no network setup, but does need a file manager
app on the television — most do not have one built in.

1. Install a file manager on the television from the Play Store. **X-plore File
   Manager** and **File Commander** both work and both handle installing APKs.
2. Copy the `.apk` onto a USB stick from your computer.
3. Plug the stick into the television.
4. Open the file manager, browse to the stick, and select the `.apk`.
5. Enable unknown sources for the file manager if you are asked, then install.

## Signing in on a television

You do not type your password with a remote control. When you open the app on a
television it shows a sign-in screen with a short code on it:

> **Sign in to MediaLib**
> On your phone or computer, go to `https://api.spameri.cz/link` and enter this
> code
>
> **K7M4QX**

Take your phone, sign in at [api.spameri.cz](https://api.spameri.cz), go to
[api.spameri.cz/link](https://api.spameri.cz/link), and type the code in.

The television is checking every five seconds, so a moment after you approve it
the screen moves on by itself. You do not need to touch the remote.

Some details that save confusion:

- **The code lasts 15 minutes.** If it expires the television says so and offers
  a new one.
- **The code never contains I, L, O, 0 or 1.** If you think you are looking at
  one of those, it is a different character.
- **Each code works once.** If you approve the same code twice, the second
  attempt is rejected.
- **There is a way out if the code flow gives you trouble**: the sign-in screen
  has a **Sign in with password instead** button, which lets you type your
  credentials with the remote.

## Updating the app

The app does not update itself, and there is no store to do it for you. When a
new version is released you download the new `.apk` and install it over the old
one, exactly as you did the first time. Android recognises it as an update and
keeps your sign-in and settings.

There is no notification when a new version appears, so it is worth checking the
releases page occasionally, or asking whoever invited you to say when something
has changed.

The web player, by contrast, is always current — it is a website, so there is
nothing to update.

## Which app talks to which server

Nothing to configure. The app is built pointing at `api.spameri.cz` and there is
no server address field in it. If you are following instructions that mention
typing in a server URL, those instructions are out of date.
