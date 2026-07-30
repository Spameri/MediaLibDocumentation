# First steps

This assumes the agent is installed and running on the machine with your video
files. If it is not, start with [install-agent.md](install-agent.md).

## 1. Sign in

Open the invite link you were sent. It looks like:

```
https://api.spameri.cz/register?code=7F3KD9QAXM
```

The code is filled in for you. Choose a name and a password and register.
Registration is invite-only, so this link is the only way in — without it the
registration form will not accept you.

After signing in you land on the profile screen. There is one profile already,
named after you. Select it to carry on.

## 2. Link your machine

Go to **Settings → Linked Machine**
([api.spameri.cz/settings/machine](https://api.spameri.cz/settings/machine)) and
press **Generate pairing code**. You get something like `H4KP-2QRW`, good for
15 minutes.

On the machine with your files, run:

```
/opt/medialib-agent/medialib-agent pair --server https://api.spameri.cz --code H4KP-2QRW
```

Adjust the path for where you installed it, and use the code you were just
given. On Windows that is `C:\medialib\medialib-agent.exe`; on Linux, pair as
the service account as shown in [install-agent.md](install-agent.md).

Start the agent (or restart the service, if you set one up), then reload the
settings page. Your machine should now be listed with a green **Online** badge.

If it says Offline, see
[troubleshooting.md](troubleshooting.md#the-agent-shows-offline).

## 3. Tell it where your files are

Still on the Linked Machine page, there is a **Library folders** box. Type a
folder path exactly as it appears on that machine, choose whether it holds
**Movies** or **Shows**, and press **Add**.

Some real examples:

| Machine | Path |
| --- | --- |
| Windows | `D:\Movies` |
| Windows, network share | `\\tower\media\Films` |
| Linux | `/srv/media/movies` |
| Synology | `/volume1/video/movies` |
| QNAP | `/share/Multimedia/Movies` |

The path is not checked by the website — it is sent to your machine, which looks
for it and reports back. So a typo will be accepted here and simply find nothing
later. Add movies and shows as separate folders; the type you pick decides how
files inside are interpreted.

### How your files need to be arranged

This is the part that decides whether you get proper titles and posters or a
list of filenames.

**Only these extensions are looked at:** `mp4`, `mkv`, `avi`, `mov`, `wmv`,
`flv`, `webm`, `m4v`, `mpg`, `mpeg`. Anything else — including `.ts`, `.m2ts`,
`.iso` and `.vob` — is ignored completely.

**Movies** are read from the filename. It needs a title and a year:

```
D:\Movies\Inception.2010.1080p.BluRay.x264.mkv
D:\Movies\The Matrix (1999).mkv
D:\Movies\The_Godfather_1972_1080p_BluRay_x264.mkv
```

Dots, underscores and spaces all work as separators. Quality and codec tags are
recognised and stripped from the title rather than becoming part of it. A film
with no year in the name still gets imported, but matching it to the right entry
is guesswork from the title alone, so include the year where you can.

**Shows take their name from the folder, not the file.** This surprises people.
The layout you want is:

```
/srv/media/shows/Breaking Bad/Season 01/Breaking.Bad.S01E01.Pilot.1080p.mkv
/srv/media/shows/Breaking Bad/Season 01/Breaking.Bad.S01E02.mkv
/srv/media/shows/Friends/Season 01/Friends.1x01.mkv
```

The show name comes from the first folder inside your library folder that is not
a season folder. The filename supplies the season and episode number, in either
the `S01E01` or the `1x01` form. Both are case-insensitive.

Episodes sitting loose in the library folder, or in a bare `Season 1` folder
with no show folder above it, end up under a show called **Unknown Show**. If
you see that after a scan, this is why.

## 4. Run the first scan

Press **Scan now**.

The machine walks your folders, then inspects each video file with `ffprobe` to
learn its length, resolution and codecs. The **Recent activity** list on the same
page shows progress as a percentage. Roughly what the numbers mean:

| Progress | What is happening |
| --- | --- |
| 5% | Walking the folders, listing candidate files |
| 10–70% | Inspecting each file — the slow part |
| 75% | Sending the list to the server |
| 90% | Refreshing the machine's own copy of your library |
| 100% | Done |

The inspection stage is what takes the time, and it depends on how many files
you have and how fast the disk is — a few hundred files on a local SSD is a
minute or two, several thousand on a network share can take a good while. Files
that have not changed since the last scan are not inspected again, so later
scans are much faster.

You can leave the page. The scan continues on its own.

If you press Scan now while one is already running, you get "A scan is already
queued or running" — that is not an error, just a refusal to queue a second one.

## 5. Wait for the descriptions and posters

Your library appears in two waves.

**First**, as soon as the scan reports in, every file shows up under **Browse**
with whatever could be worked out from its name. Titles may look raw and there
will be no artwork. Everything is playable at this point.

**Then** each newly discovered title is looked up. Titles, years, descriptions,
genres, ratings and per-episode information come from Trakt.tv; posters and
backdrops come from TMDB. This happens in the background, one title at a time,
and the results appear as they arrive without you needing to reload.

How long depends on how much you added. A handful of films fill in within a
minute. A library of several hundred titles takes considerably longer, because
the lookups are deliberately paced — if the metadata services start refusing
requests, the queue backs off and retries rather than pushing through.

Two things worth knowing about this stage:

- **Only newly discovered titles are looked up.** Rescanning does not refresh
  descriptions for things you already have — there is a separate refresh action
  for that.
- **Episodes are only filled in for episodes you actually have.** The lookup
  will not create entries for the rest of a season you have not got.

If something ends up matched to the wrong film — which happens with common
titles and with films whose names contain a year, such as *Blade Runner 2049* —
you can correct it. Open the title, edit it, and search for the right entry.

## 6. Set up profiles

Profiles work like they do on a streaming service: separate watch history and
separate continue-watching lists for each person in the house, all sharing one
account and one library.

Go to [api.spameri.cz/profiles](https://api.spameri.cz/profiles) and add one per
person. Each profile can have:

- **Parental controls** — a content rating limit and a PIN, under
  **Settings → Parental Controls**. A profile restricted to a rating simply does
  not see anything above it.
- **Subtitle languages** — under **Settings → Subtitles**, as two-letter codes
  such as `cs`, `en`, `sk`. This is what gets searched for when subtitles are
  fetched.

Profiles are chosen after signing in, on the web and in the app both.

## What to do next

- Play something. Pick a film and press play from the browser to confirm the
  whole path works end to end.
- Install the app on your phone and television — [install-apps.md](install-apps.md).
- Try it from outside your home network, on mobile data, so you find out now
  rather than later whether your home upload speed is up to it. See
  [faq.md](faq.md#how-good-will-it-look-when-i-am-away-from-home).

## A few things that are not automatic

Worth knowing up front so you are not waiting for something that is not coming:

- **Subtitles are not fetched automatically.** Open a title, use the subtitle
  search in the player, and download the ones you want. They are stored on the
  server and stay available afterwards.
- **Seek preview images are not generated.** Dragging the scrub bar moves the
  position but does not show a thumbnail of what is there.
- **Scans repeat every six hours.** New files you drop in are picked up within
  that window, or immediately if you press Scan now.
