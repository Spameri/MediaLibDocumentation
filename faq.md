# Questions people ask

## Are my video files uploaded to the server?

No. Your files stay on your machine and are never copied to the server or
anywhere else.

There is one thing worth being precise about. When you watch **away from home**,
the video does travel *through* the server on its way to you: your machine sends
it out, the server passes it along to your phone, and that is that. It is
relayed, not stored — nothing is written to disk on the way through and there is
no copy afterwards. When you watch **at home**, the video does not go near the
server at all; the app fetches it straight from your machine over your own
network.

## So what is on the server?

The description of your library, rather than the library itself:

- Titles, years, descriptions, genres and posters
- Filenames and full paths as they appear on your machine, plus sizes,
  durations, resolutions and codecs
- Which profile watched what, and how far into each thing
- Any subtitles that have been downloaded, since those are small text files
- Your account details

That list is deliberately complete, including the part people do not always
think about: **the server knows the names of your files and where they sit on
your disk.** It has to, because that is what makes browsing work when your
machine is asleep.

Video is the thing that never lands there.

## Who else can see my library?

Nobody. Each account sees only its own library, and there is no browsing between
accounts, no shared catalogue, and no way to search across other people's
things. Two friends with the same film have two entirely separate entries.

The person running the server has administrative access to it, in the way that
whoever runs any server does. They can see that accounts exist, how many titles
each has, and whether the machines are online. Being honest about that is more
useful than pretending otherwise. If you would not be comfortable with that,
this is not the right thing for you.

## What happens when my internet drops?

At home, things keep working. The apps notice the server is unreachable, switch
to talking to your own machine directly, and you carry on browsing and watching
from the copy of your library that your machine keeps locally. There is a
"Local mode" indication so you know where you are.

Progress is recorded while you are in local mode and handed back to the server
when the connection returns, so you do not lose your place.

Two limits worth knowing:

- **It only works on your home network.** Away from home with the server
  unreachable, there is no path to your files at all.
- **It is bounded in time.** Local mode keeps working for about three days
  without the server. After that your device needs to reach the server again
  before it can play anything. This is a security limit — a device that has been
  cut off indefinitely has no way to learn that its access was revoked.

The web player has no local mode. Browsers always go through the server.

## What if the server is down?

The same as above: at home, the apps carry on in local mode; away from home,
nothing works until it is back.

You cannot sign in on a new device while the server is down, and a television
that has never signed in cannot be set up.

## Can I use this on more than one machine?

**One machine per account, currently.** Pairing a second one is refused with
"This account already has a linked machine. Revoke it first."

If you move your files to a different machine, install the agent there, unlink
the old machine under Settings → Linked Machine, and pair the new one. Your
library, watch history and profiles are on the server, so they are not affected
— the new machine reports the same files and everything reconnects.

One machine can hold as many library folders as you like, across as many disks
as it can see, so this is only a real limit if your media is genuinely split
across two computers.

**Devices you watch on are a different matter** — sign in on as many phones,
televisions and browsers as you want.

## How good will it look when I am away from home?

It depends almost entirely on **your home upload speed**, which is the one
number that matters and the one most people have never looked at.

Video has to leave your house at the rate it plays. A 1080p file at 8 Mbit/s
needs a sustained 8 Mbit/s of upload. Many home connections upload at a tenth of
what they download, so a connection advertised as "100 Mbit" may only manage 10
Mbit going out.

Nothing about this software changes that. What it can do is make the files
smaller: the "optimize for streaming" option re-encodes a title on your own
machine into something around 8 Mbit/s that plays comfortably over a normal
connection. That takes hours of your machine's time per film, but it is done
once.

Video sent from outside your home is also capped at roughly 8 Mbit/s when it has
to be converted on the fly, so that one person watching cannot use up the whole
household's upload. At home there is no cap.

Test this early. Watch something on mobile data on day one, so you find out
where you stand before you have built a routine around it.

## How do updates work?

Three parts, three answers.

**The agent updates itself.** It checks once a day, verifies a signature on the
new build before installing it, and restarts into it. If a new version fails to
start properly three times, it puts the previous one back on its own. You do not
have to do anything. If you would rather control this, add
`"disable_auto_update": true` to its `config.json`.

**The Android app does not.** There is no store to update it and no in-app
updater yet, so when a new version comes out you download the new `.apk` and
install it over the old one. It keeps your sign-in and settings. There is no
notification when a new version appears — check the releases page occasionally.

**The website is always current.** It is a website; there is nothing to update.

## Does my machine have to be on all the time?

To watch from outside your home, yes. There is no cached copy anywhere else —
your files exist in one place, and if that machine is off, there is nothing to
play.

Browsing still works with the machine off, because the library description lives
on the server. You can add things to your list and read descriptions. Pressing
play gets you "Your library's machine is offline."

Sleep counts as off. A machine that suspends after 30 minutes will be
unavailable exactly when you want it. Turn sleep off on any machine you intend
to use this way.

## Do I need to configure my router?

No, and this is deliberate. The agent makes its own connection out to the server
and keeps it open; everything reaches you back down that connection. There are
no ports to forward, no dynamic DNS to set up, and it works behind the shared
addressing that many providers now use, where port forwarding would not be
possible at all.

The only networking you might touch is your machine's own firewall, and only for
home-network playback on port 8484.

## What does it cost to run?

Nothing to you, beyond your own electricity and internet. The server is paid for
by whoever invited you.

Worth knowing: every minute you watch away from home uses your home connection's
upload allowance and the server's bandwidth. If your internet has a data cap,
watching remotely counts against it.

## Will it play my files, or does it convert everything?

It plays the file as-is wherever it can, which is the fast path — nothing is
re-encoded and your machine barely notices.

When it cannot, it converts on the fly. In a web browser that happens for most
`.mkv` and `.avi` files, because browsers cannot read those containers, and for
anything in a codec the browser does not support such as HEVC. The app is far
less fussy and plays most things directly.

Conversion uses real processing power. Two at once is the limit; a third is
refused rather than making all three unwatchable. If your machine is modest and
your library is full of HEVC remuxes, "optimize for streaming" is worth setting
up so the conversion happens once, overnight, instead of every time someone
presses play.

## What does "optimize for streaming" actually do to my files?

It makes a version of a file that streams well: 1080p at most, H.264, aimed at
about 8 Mbit/s, with the index at the front so playback starts immediately. If a
file is already in the right shape it is repackaged rather than re-encoded,
which takes minutes instead of hours.

You choose what happens to the original:

- **Keep** — the new version is written to a hidden `.medialib/optimized/`
  folder next to the original. Nothing you have is touched. This uses more disk.
- **Replace** — the original is moved to a `.medialib/trash/` folder next to it
  and the new version takes its place. The original is not deleted, so you can
  get it back, but nothing removes it for you either. That folder grows until
  you empty it yourself.

It refuses to start if there is not enough free space, and it checks that the
new file is the same length as the original before it replaces anything.

## Does it change or delete my files?

No, apart from "optimize for streaming" in replace mode, described above, which
you have to choose per file.

Scanning only reads. Deleting a title in the web interface removes it from the
library listing; the file on your disk stays where it is. If you want a file
gone, delete it yourself and it will disappear from the library at the next
scan.

## Will it work on my television?

If it runs Android TV or Google TV, yes — the app installs on both and has a
proper remote-control interface. See
[install-apps.md](install-apps.md#on-an-android-tv).

If it is a Samsung, LG or other non-Android television, there is no app for it.
The usual answer is a cheap Android TV box or streaming stick plugged into it.

## Can I stop using this and take my files?

Yes. They never left your machine, in the folders they were always in, with the
names they always had. Delete the agent and you are back where you started. The
only thing you lose is the watch history, which lives on the server.

## Something is broken. Where do I report it?

Sign in and go to [api.spameri.cz/feedback](https://api.spameri.cz/feedback).
Pick whether it is a fault or a request, describe it, and send.

That files a report in the public issue tracker under your display name and the
versions you are running. Your email address is never included, and neither is
anything about what is in your library. You do not need a GitHub account.

Include which film or episode it was, what you were watching on, and whether you
were at home or away. Those three facts distinguish most causes from each other.

## Does anything I do reach a third party?

Two things, both only when you ask for them.

Pressing **Trailer** on a title opens a YouTube player. That embed is the one
third-party frame in the product; it loads from youtube-nocookie.com, YouTube's
reduced-tracking domain, and nothing loads until you press the button. If you
never press it, YouTube never hears from you.

Submitting a scene to the **guessing game** uploads that single image to
spameri.cz under your linked account, where it waits for a moderator. Nothing
else about your library travels with it, and nothing is ever submitted
automatically — every submission is a button you pressed on a scene you chose.
