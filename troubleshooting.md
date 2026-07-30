# Troubleshooting

Problems are grouped by what you see. Each one explains what is actually going
on before suggesting what to do, because the fix usually depends on which of
several causes you have.

If none of this helps, report it at
[api.spameri.cz/feedback](https://api.spameri.cz/feedback) — say what you were
doing, which film or episode, and what you were watching on.

---

## The agent shows Offline

**Where you see it:** Settings → Linked Machine shows a red **Offline** badge
next to your machine.

**What it means:** the server has not got an open connection from your machine.
The badge reflects the connection as it stands right now, not when the machine
was last heard from — the "last seen" timestamp next to it tells you that.

The agent keeps one long-lived connection out to the server. If it drops, the
agent reconnects on its own, waiting a little longer after each failure up to a
minute between attempts. So a brief network hiccup fixes itself within seconds
and you would never notice. A persistent Offline means something is genuinely
wrong.

**Work through these in order:**

1. **Is the machine on and awake?** Sleep counts as off. A machine that suspends
   after 30 minutes of inactivity will be offline every time you want to watch
   something. Turn sleep off, or at least disable it when on mains power.

2. **Is the agent actually running?**

   ```
   sudo systemctl status medialib-agent          # Linux
   ```

   ```
   schtasks /Query /TN "MediaLib Agent"          # Windows
   ```

   If it is not running, start it and look at why it stopped.

3. **Read the log.** This is where the answer usually is.

   ```
   sudo journalctl -u medialib-agent -n 50       # Linux
   ```

   On Windows, the console window the task opened.

   | What you see | What it means |
   | --- | --- |
   | `tunnel closed: … — reconnecting in 30s` repeatedly | It cannot reach the server. Check the machine's own internet connection, and whether anything is filtering outbound traffic. |
   | `dial … 401` or the tunnel closing immediately after connecting | The credential is not accepted. The machine was probably unlinked in the web UI. Pair it again. |
   | `config incomplete — run 'agent pair' first` | It cannot find its configuration. Almost always because it is running as a different account than the one you paired with — see below. |
   | `cannot determine owning user (re-pair?)` | Same cause; pair again as the account the service runs as. |

4. **Check it is running as the account you paired with.** This catches a lot of
   people. The credential is stored in the running account's own profile folder.
   Pair as yourself and then run the service as `SYSTEM`, or as a `medialib`
   system user, and it will not find anything.

   The fix is to pair again as the account that runs the service:

   ```
   sudo -u medialib env HOME=/var/lib/medialib-agent \
     /opt/medialib-agent/medialib-agent pair --server https://api.spameri.cz --code XXXX-XXXX
   ```

5. **Outbound traffic.** The agent needs to reach `api.spameri.cz` on port 443,
   including a WebSocket connection that stays open. Corporate networks and some
   security software interfere with long-lived connections. Test the basics:

   ```
   curl -sS https://api.spameri.cz/health
   ```

   If that fails, the problem is the machine's networking, not this software.

---

## The scan finishes but nothing appears

**What it means:** the scan ran and found no files it recognised. It is almost
never a bug — it is one of four things.

1. **The folder path is wrong.** The path you typed on the website is never
   checked there; it is sent to your machine to look for. A typo, or a path that
   is right on one machine and not another, silently finds nothing. Check the
   path on the machine itself:

   ```
   ls /srv/media/movies          # Linux, NAS
   ```

   ```
   dir D:\Movies                 # Windows
   ```

   Whatever that command needs to work is exactly what belongs in the folder
   box.

2. **The agent cannot read the folder.** If you set it up as its own system
   user, that user needs read access to your files — and it very often does not
   have it by default, particularly for files under `/home/you`. Check as that
   user, not as yourself:

   ```
   sudo -u medialib ls /srv/media/movies
   ```

   `Permission denied` is your answer. Add the account to the owning group, or
   loosen the permissions on the media folders.

3. **The file extensions are not ones that get scanned.** Only these are looked
   at: `mp4`, `mkv`, `avi`, `mov`, `wmv`, `flv`, `webm`, `m4v`, `mpg`, `mpeg`.
   A library of `.ts` recordings, Blu-ray `.m2ts`, DVD `.vob` or disc images
   will scan as empty and it will look exactly like a broken scan. There is no
   message telling you the files were skipped.

4. **A network share was not mounted.** If the media lives on a NAS mounted on
   another machine, and the mount was not there when the agent looked, the
   folder appears empty. Files that were there before are marked missing and
   disappear from the library; they come back on the next scan once the mount is
   restored.

**Everything is under a show called "Unknown Show"** is a different problem with
a specific cause: episodes that are not inside a folder named after the show.
The show name comes from the folder, not the filename. See
[first-steps.md](first-steps.md#how-your-files-need-to-be-arranged).

**Files appear but have no duration, resolution or quality label** means ffmpeg
is missing on that machine. The files were found and listed but could not be
inspected. Install `ffmpeg` and `ffprobe`, then scan again.

---

## "Your library's machine is offline" while trying to play

**Where you see it:** a black overlay in the player headed **Playback Error**,
with that sentence and a **Try Again** button.

**What it means:** exactly what it says. The server tried to fetch video from
your machine and there was no connection to fetch it through. The metadata you
are looking at is stored on the server, which is why you can browse a library
you cannot play from.

The player retries three times on its own, waiting one, then two, then four
seconds, before giving up and showing this.

**What to do:** this is the same problem as
[the agent showing Offline](#the-agent-shows-offline) — work through that
section. Once the machine is back, press **Try Again** rather than reloading;
it picks up where it was.

The thing to understand is that there is no cached copy anywhere. Your video
files exist in exactly one place. If that machine is off, there is nothing to
play, and no amount of retrying will produce it.

---

## Playback stutters or buffers when I am away from home

**What it means:** video is travelling from your home machine, out through your
home internet connection, to the server, and back down to you. The limit is
almost always **your home upload speed**, which on most connections is a
fraction of the download speed.

A 1080p file at 8 Mbit/s needs 8 Mbit/s of upload, sustained, to play without
pausing. If your connection uploads at 5 Mbit/s, that file cannot be watched
away from home at full quality no matter what. This is a physical limit, not a
setting.

**Find out what you have.** Run a speed test on the home connection and look at
the *upload* number. Compare it with the bitrate of what you are trying to
watch — a rough figure is the file size in gigabytes divided by the runtime in
hours, times 2.2, to get Mbit/s.

**What actually helps:**

- **"Optimize for streaming".** On a file's page there is an option to
  re-encode it into a form that streams well: 1080p, H.264, sized to stay within
  about 8 Mbit/s. It runs on your own machine and takes a while, but it turns an
  unwatchable 40 GB remux into something that plays over a normal connection.
  Files already in the right format are repackaged in minutes rather than
  re-encoded over hours.
- **Watch the smaller copy.** If you have both a 4K and a 1080p version, the
  1080p one is the one to take on the road.
- **Wired, not Wi-Fi, at the home end.** An agent on Wi-Fi is sharing capacity
  with everything else and adds its own instability.

**What does not help:** upgrading the phone's connection. If your home upload is
the bottleneck, a faster connection at your end changes nothing.

Two things happen automatically that you should know about. Re-encoded video
sent from outside your home is capped at about 8 Mbit/s, deliberately, so that
one stream cannot saturate your whole connection. And no more than two files can
be converted on the fly at once — a third attempt is refused with "Too many
transcodes in progress" rather than making all three stutter.

---

## Subtitles are missing

**The main thing to know: subtitles are not fetched automatically.** Nothing is
searched for on your behalf when files are scanned. This is the most common
reason for "there are no subtitles" and it is working as built, not broken.

**To get them:** open the title, use the subtitle search in the player, and pick
what you want from the results. It searches OpenSubtitles by the file's own
fingerprint first, which is the most accurate way to match a specific release,
then falls back to identifiers and finally to title and year. Downloaded
subtitles are stored on the server and stay there — you only do this once per
file.

You can also upload your own `.srt` if you have it.

**Other reasons subtitles do not show:**

- **Subtitles inside the video file itself are not extracted.** A `.mkv` with
  embedded subtitle tracks will not offer them in the player. Search for an
  external one instead.
- **The language you want was not searched for.** Which languages get looked for
  comes from the profiles' preferences. Set yours under **Settings → Subtitles**
  using two-letter codes such as `cs`, `en`, `sk`.
- **Nothing exists for that release.** Obscure films and non-English titles
  often have nothing available, particularly for a specific release. Try
  searching by title rather than accepting the automatic match.
- **The player picks one for you and it is the wrong one.** When several
  languages are available it selects the first alphabetically by language code,
  not your preferred one. Change the track in the player.

---

## The app cannot sign in

**"Invalid credentials" or the login is rejected.** Your email and password are
the ones you registered with on the website. Confirm they work by signing in at
[api.spameri.cz](https://api.spameri.cz) in a browser. If they do not work
there either, use the password reset on the sign-in page.

**The app cannot reach anything at all.** Check that the phone has working
internet, then that `https://api.spameri.cz` loads in the phone's browser. If
the website loads but the app does not connect, the app build may be older than
the current server — reinstall the newest `.apk` from the releases page.

**On a television, the code is not accepted.** Several possibilities:

- **It expired.** Codes last 15 minutes. The television offers a new one; take
  it.
- **You already used it.** Each code works exactly once. Get a fresh one.
- **You misread a character.** Codes never contain I, L, O, 0 or 1, precisely so
  that they cannot be confused. If you think you see one, it is something else.
- **You were signed out on the phone.** You must be signed in at
  [api.spameri.cz](https://api.spameri.cz) before the `/link` page will do
  anything.

If the code flow is not working at all, the television's sign-in screen has a
**Sign in with password instead** button.

**Signed in yesterday, signed out today.** Sessions last a long time and renew
themselves. Being asked to sign in again usually means the device was revoked,
or the app was cleared. Just sign in again.

---

## It is not using the fast home connection

**What should happen:** when your phone is on the same network as your machine,
the app fetches video straight from it rather than routing through the server.
That is faster, does not touch your internet connection, and is not subject to
the remote quality cap.

**How the app decides:** it asks the server where your machine is on the local
network, then tries to reach it directly, giving it about a second and a half to
answer. If it does, home mode is used. If not, everything goes through the
server. This check runs each time you start playing something.

**Why it might not kick in:**

- **The machine has not reported a local address yet.** It sends one every
  minute once running. A machine that just started may not have been heard from.
- **Your network isolates devices from each other.** Guest networks and the
  "AP isolation" or "client isolation" setting on many routers block devices
  from talking to each other. Your phone can reach the internet and your
  machine can reach the internet, but they cannot reach each other. Take the
  phone off the guest network.
- **Separate 2.4 GHz and 5 GHz networks, or a mesh with separate subnets.** If
  the phone and the machine end up on different subnets, the direct connection
  does not work. They need to be on the same one.
- **A firewall on the machine is blocking port 8484.** That is the port the
  agent listens on at home. See the firewall commands in
  [install-agent.md](install-agent.md).
- **You are using a browser.** Web browsers always go through the server, even
  at home. This is a limitation of browser security — a page served over HTTPS
  is not permitted to fetch from a device on your local network. Use the app if
  you want home-network playback.

**To check the machine is reachable**, from another device on the same network:

```
curl -k https://192.168.1.50:8484/health
```

Use your machine's own address. A small piece of JSON back means it is working
and the problem is on the phone's side. A timeout means the network or a
firewall is in the way. The `-k` is expected — the agent uses its own
certificate, which the app verifies by fingerprint rather than through a
certificate authority.

---

## Wrong title, wrong year, wrong film

Matching works from the filename, and some filenames are ambiguous in ways no
amount of cleverness fixes.

**A year inside the title gets taken as the release year.** *Blade Runner 2049
(2017)* is read as the film "Blade Runner" from the year 2049, which then
matches nothing sensible. Renaming the file so the real year is the only
four-digit number in it fixes it, or you can correct the match by hand.

**Common titles match the wrong entry.** Remakes and films sharing a name are
resolved by the year, so a file without one is a coin toss.

**To fix any of these:** open the title, edit it, and search for the correct
entry. The correction sticks and is not undone by a later scan.

**Multi-episode files lose the second episode.** A file named `S01E01E02` is
recorded as episode 1 only. Split the file, or accept that it will be filed
under the first of the two.

**No posters at all, for anything.** That is a server-side configuration
problem, not something on your machine. Report it.

---

## The player scrub bar has no preview images

Working as built — preview thumbnails are not generated for files served from
your own machine. Dragging the scrub bar moves the position and the timestamp
updates, but the small preview picture never appears.

---

## The library lost files that are still there

Files that could not be seen during a scan are marked as missing and removed
from the library. The usual cause is a network share that was not mounted, or a
disk that was not spun up when the scan ran.

Reconnect the storage and run a scan. The files come back, along with their
watch progress — the entries are hidden rather than destroyed.

Scans run automatically every six hours, so if a share is intermittently
available, expect things to come and go. Making the mount reliable is the real
fix.
