# Your own film and TV library

This is a way to watch your own video files — the ones sitting on a computer or
NAS at your home — on your phone, your television, or in a web browser, whether
you are at home or somewhere else.

It is not a streaming service. There is no catalogue to browse and nothing to
rent. You supply the files; this gives you a decent way to watch them.

## What you need

- **A computer or NAS that holds your video files and stays switched on.** This
  is the important one. When that machine is asleep or unplugged, nothing can
  play from outside your home, because the files only ever live there.
- **An account.** Registration is invite-only, so someone has to send you a
  link. It looks like `https://api.spameri.cz/register?code=7F3KD9QAXM`.
- **A phone, an Android TV, or any modern web browser** to watch on.

A reasonable home upload speed helps but is not required for watching at home.
More on that in [faq.md](faq.md).

## How the pieces fit together

There are three parts.

**The platform** at [api.spameri.cz](https://api.spameri.cz) is a website run on
a server. It holds your account, the list of what is in your library, the
posters and descriptions, and where you got to in each episode. It does not hold
your video files.

**The agent** is a small program you install on the machine where your video
files are. It looks through the folders you point it at and tells the platform
what it found — filenames, sizes, how long each file is. When you press play, it
is the agent that sends the video.

**The apps** are what you watch in: an Android app that works on both phones and
televisions, or any web browser.

The agent makes its own connection out to the server and keeps it open. Nothing
connects inward to your home, so you do not need to open ports on your router or
set up anything on it at all. That connection is also how video reaches you when
you are away from home: it goes from your machine, out through the server, to
your phone.

When you are at home on the same network, the apps notice and fetch video
straight from your machine instead, which is faster and does not use your
internet connection.

## What it does with your files

It reads them. It does not move them, rename them, or upload them anywhere.

The one exception is a feature you have to switch on yourself for a specific
file — "optimize for streaming" — which re-encodes a file into a format that
plays more smoothly. Even then it writes a separate copy alongside the original
unless you explicitly choose to replace it. See [faq.md](faq.md).

## Getting started

Work through these in order:

1. **[install-agent.md](install-agent.md)** — put the agent on the machine with
   your files. Windows, Linux, and NAS.
2. **[install-apps.md](install-apps.md)** — get the app onto your phone and your
   television.
3. **[first-steps.md](first-steps.md)** — link the machine, point it at your
   folders, and run the first scan.

When something goes wrong, [troubleshooting.md](troubleshooting.md) covers the
problems people actually hit. [faq.md](faq.md) answers the questions people ask
before they commit to setting this up.

## When something is broken

Sign in and go to [api.spameri.cz/feedback](https://api.spameri.cz/feedback).
Describe what happened and send it. That files a report in the public issue
tracker under your display name — your email address is never included. You do
not need a GitHub account.

Say which film or episode it was and what you were watching on. A report that
says "it does not work" cannot be acted on.

## Reporting a problem

Open an issue here, or use the "Tell me about it" link inside the app, which
files one for you with your version details attached.
