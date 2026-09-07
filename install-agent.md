# Installing the agent

The agent is one small program. It has no installer, no dependencies to
manage, and no Docker. You download a file, run it once to pair it with your
account, then arrange for it to keep running.

You do not need anything from the website before you start. The agent shows
an eight-character code like `H4KP-2QRW` when it first runs; you type that code
at [media.knivy.xyz](https://media.knivy.xyz) under **Settings → Linked
Machine** — from any device, signed in as you. Codes last 15 minutes and the
agent shows a fresh one when one runs out.

## The short version

On a machine with a desktop, you do not need the rest of this document:

1. Download the agent for your platform.
2. Double-click it. It prints a code and opens the website with the code
   filled in.
3. Sign in if asked and press **Link this machine**.

That is all the machine itself is ever asked. Everything after it — which
folders to watch, when to scan, what to convert ahead of time — is on the
website under **Settings → Linked Machine**, or in the app.

There is nothing else to install.

The commands further down are for machines with no desktop — a NAS, a
headless server — and for keeping the agent running after a reboot.

## ffmpeg comes with it

Nothing to install. The agent carries its own copies of `ffmpeg` and `ffprobe`
and unpacks them next to its own data the first time it runs — which is why the
download is around 90 MB for what is otherwise a small program.

That matters because everything the agent does to a video needs them: reading
how long a file is and what it was encoded with, making the thumbnails, and
converting anything your browser will not play as-is. An agent without them
scans happily, fills your library with titles, and then refuses to play any of
them.

If you would rather it used an ffmpeg you already have, put the paths in the
agent's config file as `ffmpeg_path` and `ffprobe_path` and it will prefer
those.

## Which file to download

All builds are on the releases page:

**<https://github.com/Spameri/MediaLibDocumentation/releases/latest>**

| Your machine | File |
| --- | --- |
| Windows, 64-bit | `knivy-agent-windows-amd64.exe` |
| Linux, Intel or AMD | `knivy-agent-linux-amd64` |
| Linux or NAS, ARM (most modern NAS boxes, Raspberry Pi 4/5) | `knivy-agent-linux-arm64` |

If you are not sure whether your NAS is ARM or Intel, run `uname -m` over SSH.
`x86_64` means the amd64 build; `aarch64` or `arm64` means the arm64 build.

---

## Windows

### 2. Put the agent somewhere permanent

Create a folder and download the binary into it. `C:\knivy` is a good choice
because it stays out of `Program Files`, which matters: the agent replaces its
own file when it updates itself, and it needs write access to its own folder to
do that.

```
mkdir C:\knivy
curl.exe -L -o C:\knivy\knivy-agent.exe https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/knivy-agent-windows-amd64.exe
```

### 3. Link it

```
C:\knivy\knivy-agent.exe link
```

It prints a code and opens the website. Sign in, press **Link this machine**,
and the command finishes on its own:

```
Paired as agent 01K3F7Q2XVJ8R5NBWZ4CDMHT9E.
Credential stored in C:\Users\you\AppData\Roaming\knivy\config.json.
Start serving with: agent run
```

Link as the same Windows account that will run the agent later. The credential
is written into that account's profile, and another account will not find it.

For a scripted install there is the other direction: generate a code on the
website (*Other ways to link a machine*) and pass it in —
`knivy-agent.exe pair --server https://media.knivy.xyz --code H4KP-2QRW`. The
hyphen is optional.

### 4. Try it once by hand

```
C:\knivy\knivy-agent.exe run
```

Leave it for a few seconds. You are looking for a line like
`tunnel established to wss://media.knivy.xyz/tunnel`. Windows Firewall may ask
about incoming connections — allow it on private networks so that watching at
home works. Press `Ctrl+C` to stop it.

### 5. Make it start on its own

The agent has no Windows service of its own, so use Task Scheduler. These
commands use Command Prompt syntax — open **Command Prompt as Administrator**,
not PowerShell, because the quoting below is not the same in both.

The straightforward version, which starts the agent when you log in:

```
schtasks /Create /TN "Knivy Agent" /TR "\"C:\knivy\knivy-agent.exe\" run" /SC ONLOGON /RL HIGHEST /F
```

This leaves a console window open while it runs. That is normal, and closing it
stops the agent. The task only has to start it once: the agent restarts itself
after an update or a crash, in the same window.

If the machine reboots without anyone logging in — which is what you want for a
machine that just sits there — use a start-up task with your account's password
stored instead:

```
schtasks /Create /TN "Knivy Agent" /TR "\"C:\knivy\knivy-agent.exe\" run" /SC ONSTART /RU "%USERNAME%" /RP * /RL HIGHEST /F
```

`/RP *` prompts for your Windows password and stores it so the task can run
without you. It must be the same account you paired with. Do not use `SYSTEM`
here: it has a different profile folder and would not find the credential you
just created.

Start it now without rebooting:

```
schtasks /Run /TN "Knivy Agent"
```

To allow the home-network connection through the firewall explicitly:

```
netsh advfirewall firewall add rule name="Knivy Agent" dir=in action=allow protocol=TCP localport=8484
```

### Where things live on Windows

| What | Where |
| --- | --- |
| Configuration and credential | `%AppData%\knivy\config.json` |
| Everything else the agent keeps | `%AppData%\knivy\` |

That folder also holds `state.db` (its record of your files), `lan-cert.pem` and
`lan-key.pem` (the certificate used for watching at home), and cached poster
images.

---

## Linux

### 2. Create a user and install the binary

Running the agent as its own account keeps it away from the rest of your system.

```
sudo useradd --system --create-home --home-dir /var/lib/knivy-agent --shell /usr/sbin/nologin knivy
sudo mkdir -p /opt/knivy-agent
sudo curl -L -o /opt/knivy-agent/knivy-agent https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/knivy-agent-linux-amd64
sudo chmod 0755 /opt/knivy-agent/knivy-agent
sudo chown -R knivy:knivy /opt/knivy-agent
```

On ARM, swap `knivy-agent-linux-amd64` for `knivy-agent-linux-arm64`.

The `chown` on the whole directory is deliberate. When the agent updates itself
it writes the new binary next to the old one and swaps them, which it cannot do
in a directory it does not own. If you would rather install to a root-owned
location such as `/usr/local/bin`, that works too — you just have to install
updates by hand from then on.

### 3. Give it access to your media

The `knivy` account needs to read your video folders. Read access is enough
for scanning and playback. If you plan to use "optimize for streaming" or
Webshare downloads, it needs write access to those folders too.

The simplest approach on a single-user machine is to add it to the group that
already owns the files:

```
sudo usermod -aG "$(stat -c '%G' /srv/media)" knivy
```

Check it worked, using one of your own paths:

```
sudo -u knivy test -r /srv/media && echo readable || echo "not readable"
```

### 4. Link it

Link as the `knivy` user, so the credential lands where the service will look
for it:

```
sudo -u knivy env HOME=/var/lib/knivy-agent \
  /opt/knivy-agent/knivy-agent link
```

It prints a code and waits. Type the code at
[media.knivy.xyz/settings/machine](https://media.knivy.xyz/settings/machine)
on any device; the command finishes once the machine is linked. (With a code
generated on the website instead, the same command is
`… knivy-agent pair --server https://media.knivy.xyz --code H4KP-2QRW`.)

The credential is written to
`/var/lib/knivy-agent/.config/knivy/config.json` with permissions
`0600`.

### 5. Install the service

Save this as `/etc/systemd/system/knivy-agent.service`:

```ini
[Unit]
Description=Knivy agent
Documentation=https://github.com/Spameri/MediaLibDocumentation
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=knivy
Group=knivy
# The agent stores its configuration under $HOME and refuses to start
# without one, so set it explicitly rather than relying on the account.
Environment=HOME=/var/lib/knivy-agent
ExecStart=/opt/knivy-agent/knivy-agent run
# The agent restarts itself after an update, so this is here for the case it
# cannot handle: the whole thing being killed. Restart=always rather than
# on-failure, because a clean exit here is still an agent that is not serving.
Restart=always
RestartSec=10
# Transcoding is CPU-hungry; keep the rest of the machine usable.
Nice=5
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

Then:

```
sudo systemctl daemon-reload
sudo systemctl enable --now knivy-agent
```

Watch it start:

```
sudo journalctl -u knivy-agent -f
```

Within a few seconds you should see `tunnel established to
wss://media.knivy.xyz/tunnel`.

If you run a firewall, allow the home-network port:

```
sudo ufw allow from 192.168.0.0/16 to any port 8484 proto tcp
```

### Where things live on Linux

| What | Where |
| --- | --- |
| Configuration and credential | `$HOME/.config/knivy/config.json` |
| Everything else the agent keeps | `$HOME/.config/knivy/` |

With the service set up as above, `$HOME` is `/var/lib/knivy-agent`.

---

## NAS

A NAS is just Linux with a friendlier front end, so the Linux instructions apply
once you have a shell. The awkward part is getting something to start at boot;
ffmpeg is not a problem, because the agent brings its own.

Enable SSH first — on Synology under **Control Panel → Terminal & SNMP**, on
QNAP under **Control Panel → Telnet / SSH** — then connect with
`ssh you@your-nas`.

### ffmpeg on a NAS

Nothing to do — the agent unpacks its own alongside its data directory. This is
worth knowing on a NAS in particular, where the usual answer was Entware or
borrowing the copy inside a Plex or Video Station package, and where those
vendor builds are often missing encoders the agent needs.

It does mean the agent wants roughly 300 MB of space in its data directory. On
a NAS whose system volume is small, point `HOME` at the data volume when you
run it — the systemd unit further up already does this.

If you would rather it used an ffmpeg you already have, name it explicitly in
`config.json` after pairing:

```json
{
  "ffmpeg_path": "/var/packages/VideoStation/target/bin/ffmpeg",
  "ffprobe_path": "/var/packages/VideoStation/target/bin/ffprobe"
}
```

### Installing and pairing

Most NAS boxes are ARM, so start with the arm64 build. Put it somewhere on the
data volume — the system partition is often small and gets wiped by firmware
updates.

```
mkdir -p /volume1/knivy
curl -L -o /volume1/knivy/knivy-agent https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/knivy-agent-linux-arm64
chmod +x /volume1/knivy/knivy-agent
/volume1/knivy/knivy-agent link
```

It prints a code and waits; type the code at
[media.knivy.xyz/settings/machine](https://media.knivy.xyz/settings/machine) on
your phone or computer, and the command finishes once the NAS is linked.

If it fails with `cannot execute binary file`, you have an Intel NAS — use
`knivy-agent-linux-amd64` instead.

Your library paths on a NAS look like `/volume1/movies` and `/volume1/tv`
(Synology) or `/share/Multimedia/Movies` (QNAP). You should not need them: the
website lists the machine's own disks and you pick from them. Note them down
anyway if this NAS is one you reach only over SSH — a typed path is the fallback
for exactly that case. See [first-steps.md](first-steps.md).

### Starting it at boot

**If your NAS has systemd** — check with `systemctl --version` — use the unit
file from the Linux section above, adjusting `ExecStart` to
`/volume1/knivy/knivy-agent run` and setting `Environment=HOME=` to a
directory on the data volume, for example `/volume1/knivy/home`. Pair as that
same user with the same `HOME` so the credential lands in the right place.

**On Synology DSM**, the supported route is the built-in scheduler: **Control
Panel → Task Scheduler → Create → Triggered Task → User-defined script**, with
the user set to `root`, the event set to **Boot-up**, and the script:

```
/volume1/knivy/knivy-agent run &
```

**On QNAP**, use **Control Panel → System → Hardware → Autorun** (which runs
`autorun.sh` from the system partition) or a cron entry via **Task Scheduler**
with the same command.

Both of these are less robust than systemd: if the agent stops, nothing brings
it back until the next reboot. If your NAS has systemd, prefer it.

---

## Checking that it is working

Four things to look at, in order of usefulness.

**1. The web UI.** Go to
[media.knivy.xyz/settings/machine](https://media.knivy.xyz/settings/machine). Your
machine should be listed with a green **Online** badge, its platform, and the
agent version. This is the check that matters — it means the machine reached the
server and the server can reach it back.

**2. The logs.** The line you want is:

```
tunnel established to wss://media.knivy.xyz/tunnel
```

`journalctl -u knivy-agent -f` on Linux; the console window on Windows.

Repeated `tunnel closed: … — reconnecting in 2s` lines mean it cannot get out to
the server. The delay doubles on each failed attempt up to a minute, so it keeps
trying without hammering anything.

**3. The version.** Confirms the binary itself runs:

```
/opt/knivy-agent/knivy-agent version
```

**4. The home-network endpoint.** From another machine on the same network:

```
curl -k https://192.168.1.50:8484/health
```

Use your machine's own address. You should get back a small piece of JSON with
the agent id and version. The `-k` is expected: the agent uses a certificate it
generated itself, which the apps verify by fingerprint rather than through a
certificate authority.

## Keeping it up to date

The agent checks for a new version once a day and installs it on its own,
verifying a signature before it does. The first check happens a few minutes
after it starts, at a randomised moment, so a batch of machines restarted
together do not all ask at once.

Installing an update means the running program has to be replaced by the one
just downloaded, and the agent does that itself: the thing you start is a small
supervisor that runs the agent and starts it again — after an update, after a
crash, after the machine kills it for using too much memory. Nothing on your
side has to arrange that. The restart waits for anything you are watching to
finish first, so an update never stops a film halfway.

Before installing, the new build is asked for its version, which it can only
answer by starting. A download this machine cannot run is thrown away and the
working agent keeps going. If a new build does install and then fails three
times in a row — or comes up but never reaches Knivy — the agent puts the
previous one back by itself.

**Once, if you are updating from 0.9.2 or older:** that version exits after
installing an update and does not come back, because the code doing the
installing is the old code. Start the agent on the machine one more time after
the update lands. From 0.9.3 on it comes back on its own.

To turn this off, add `"disable_auto_update": true` to `config.json` and restart
the agent. You are then responsible for replacing the binary yourself. One
exception the setting does not cover: a build old enough that Knivy no longer
supports it is installed anyway, because the alternative is a machine that
quietly stops serving.

## Things worth knowing

- **One machine per account.** Linking a second machine is refused with "This
  account already has a linked machine." Unlink the old one under Settings →
  Linked Machine first.
- **The agent only ever connects outward** — to `media.knivy.xyz` over HTTPS and
  one long-lived WebSocket. Nothing needs to reach in from the internet, so
  there is nothing to configure on your router.
- **It listens on port 8484** on your local network only, for watching at home.
- **The full scan repeats every six hours.** You can also trigger one at any
  time from the website or the app.
- **A folder holds whatever you put in it.** You are not asked whether it is for
  films or programmes; what each file is comes from the file.
- **The credential file is worth protecting.** Anyone who copies
  `config.json` can impersonate your machine. It is written `0600` for that
  reason. If you think it has leaked, unlink the machine in the web UI and link
  it again — the old credential stops working immediately.
