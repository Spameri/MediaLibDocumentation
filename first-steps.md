# First steps

This assumes the agent is installed and running on the machine with your video
files. If it is not, start with [install-agent.md](install-agent.md).

## 1. Sign in

Go to [media.knivy.xyz](https://media.knivy.xyz) and press **Sign in**. Knivy uses your
**knivy.xyz account** — there is no separate Knivy password to invent or
remember. If you have a spameri account, approve the sign-in there and you are
through; if you do not, the person who invited you will have had one made for
you and told you the details.

After signing in you land on the profile screen. There is one profile already,
named after you. Select it to carry on.

## 2. Link your machine

Go to the machine that holds your files and start the agent — on Windows and
macOS, double-click it. It prints a short code, something like `H4KP-2QRW`,
and opens the website in your browser with that code already filled in:

```
Knivy agent — first-time setup

This machine is not linked to an account yet.
On any device, go to https://media.knivy.xyz/settings/machine
and type this code:

    H4KP-2QRW
```

Sign in if the website asks you to, and press **Link this machine**. A few
seconds later the machine is listed under **Settings → Linked Machine** with a
green **Online** badge. The code is good for 15 minutes; the agent shows a fresh
one after that.

You do not have to use the browser the agent opened. The same code can be typed
on your phone, on a television, or on any other computer — wherever you are
signed in, at **Settings → Linked Machine**.

If the machine says Offline, see
[troubleshooting.md](troubleshooting.md#the-agent-shows-offline).

### If there is no browser on that machine

On a NAS or a server you reached over SSH, the agent prints the code in the
terminal and waits. Type it at
[media.knivy.xyz/settings/machine](https://media.knivy.xyz/settings/machine) on
whatever you are reading this on. `knivy-agent link` does the same thing and
exits once the machine is linked, which is what the
[service recipes](install-agent.md) use.

There is also the other way round, for scripts: press **Generate pairing code**
under *Other ways to link a machine* on that page and carry the code to the
machine with
`knivy-agent pair --server https://media.knivy.xyz --code H4KP-2QRW`.

## 3. Tell it where your files are

Do this on the website — **Settings → Linked Machine** — or in the app. Press
**Add folder** and you get a list of the machine's own disks to walk through.

The question goes to your machine over the connection it already has open, so
what you see is what is actually there. Nothing to type, and nothing to get
wrong. Windows has no single root, so the list starts at drive letters.

You do not tell Knivy what a folder holds. Films and programmes can sit in the
same folder, in any arrangement; what each file is comes from the file.

There is still a **Type a path instead** box. It is the only route for a machine
you can reach only over SSH, and the one to use when the machine is switched off
or its agent is too old to be asked. Nothing checks a typed path — it is sent to
your machine, which looks for it — so a typo is accepted and simply finds
nothing.

Some real examples of what the paths look like:

| Machine | Path |
| --- | --- |
| Windows | `D:\Movies` |
| Windows, network share | `\\tower\media\Films` |
| Linux | `/srv/media/movies` |
| Synology | `/volume1/video/movies` |
| QNAP | `/share/Multimedia/Movies` |

### How your files need to be arranged

This is the part that decides whether you get proper titles and posters or a
list of filenames.

**Only these extensions are looked at:** `mp4`, `mkv`, `avi`, `mov`, `wmv`,
`flv`, `webm`, `m4v`, `mpg`, `mpeg`. Anything else — including `.ts`, `.m2ts`,
`.iso` and `.vob` — is ignored completely.

**Films** are read from the filename, or from the folder when the filename says
nothing useful. A title and a year is what matching wants:

```
D:\Movies\Inception.2010.1080p.BluRay.x264.mkv
D:\Movies\The Matrix (1999).mkv
D:\Movies\The_Godfather_1972_1080p_BluRay_x264.mkv
```

Dots, underscores and spaces all work as separators. Quality and codec tags are
recognised and stripped from the title rather than becoming part of it. A film
with no year in the name still gets imported, but matching it to the right entry
is guesswork from the title alone, so include the year where you can.

**Programmes take their name from the folder, not the file.** This surprises
people. The layout you want is:

```
/srv/media/shows/Breaking Bad/Season 01/Breaking.Bad.S01E01.Pilot.1080p.mkv
/srv/media/shows/Breaking Bad/Season 01/Breaking.Bad.S01E02.mkv
/srv/media/shows/Friends/Season 01/Friends.1x01.mkv
```

The show name comes from the first folder inside your library folder that is not
a season folder. The filename supplies the season and episode number, in either
the `S01E01` or the `1x01` form. Both are case-insensitive.

The filename supplies the season and episode number in any of the usual forms —
`S01E01`, `1x01`, `Season 1 Episode 1`, `3. série 12. díl`, a bare `EP07`. A file
with none of those is still read as an episode when the shape of the folder says
so: a folder of its own holding several files is a series, and a file lying
loose beside season folders is one of that show's specials.

Episodes sitting loose in the *library* folder, with no show folder above them,
end up under a show called **Unknown Show**. If you see that after a scan, this
is why.

## 4. Run the first scan

Press **Scan now** on the website or in the app.

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
genres, ratings and per-episode information come from the spameri catalogue; posters and
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

Go to [media.knivy.xyz/profiles](https://media.knivy.xyz/profiles) and add one per
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

## A few things worth knowing up front

- **Subtitles are looked for automatically.** Every scan checks new files and
  fetches Czech and English subtitles where it finds them. When nothing was
  found, the player's subtitle menu can search Webshare and OpenSubtitles by
  hand — downloads are stored on the server and stay available afterwards.
- **Scans repeat every six hours.** New files you drop in are picked up within
  that window, or immediately if you press Scan now.
