# Installing the agent

The agent is one small program. It has no installer, no dependencies to
manage, and no Docker. You download a file, run it once to pair it with your
account, then arrange for it to keep running.

Before you start, sign in at [api.spameri.cz](https://api.spameri.cz), go to
**Settings → Linked Machine** (`/settings/machine`) and press **Generate
pairing code**. You get an eight-character code like `H4KP-2QRW`. It is valid
for 15 minutes; if it runs out, generate another one.

## Before anything else: ffmpeg

**The agent does not install ffmpeg and does not download it. You have to
install it yourself.** This is the single most common reason a setup half-works.

Without `ffmpeg` and `ffprobe` on the machine:

- Scanning still finds your files, but every one of them ends up with no
  duration, no resolution and no codec information.
- Anything that cannot be played as-is fails instead of being converted on the
  fly.
- "Optimize for streaming" fails.

There is no warning at startup when ffmpeg is missing, so check it yourself
before you go further. On any of the platforms below, this should print a
version banner:

```
ffmpeg -version
ffprobe -version
```

Install instructions per platform are in the sections below.

## Which file to download

All builds are on the releases page:

**<https://github.com/Spameri/MediaLibDocumentation/releases/latest>**

| Your machine | File |
| --- | --- |
| Windows, 64-bit | `medialib-agent-windows-amd64.exe` |
| Linux, Intel or AMD | `medialib-agent-linux-amd64` |
| Linux or NAS, ARM (most modern NAS boxes, Raspberry Pi 4/5) | `medialib-agent-linux-arm64` |

If you are not sure whether your NAS is ARM or Intel, run `uname -m` over SSH.
`x86_64` means the amd64 build; `aarch64` or `arm64` means the arm64 build.

---

## Windows

### 1. Install ffmpeg

Open PowerShell and run one of these, depending on what you have:

```
winget install --id Gyan.FFmpeg -e
```

```
choco install ffmpeg
```

Close and reopen PowerShell afterwards, then confirm `ffmpeg -version` works.

If neither package manager is available, download a build from
<https://www.gyan.dev/ffmpeg/builds/>, unzip it, and add its `bin` folder to
your `PATH`.

### 2. Put the agent somewhere permanent

Create a folder and download the binary into it. `C:\medialib` is a good choice
because it stays out of `Program Files`, which matters: the agent replaces its
own file when it updates itself, and it needs write access to its own folder to
do that.

```
mkdir C:\medialib
curl.exe -L -o C:\medialib\medialib-agent.exe https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/medialib-agent-windows-amd64.exe
```

### 3. Pair it

```
C:\medialib\medialib-agent.exe pair --server https://api.spameri.cz --code H4KP-2QRW
```

Use your own code. The hyphen is optional — `H4KP2QRW` works the same.

You should see:

```
Paired as agent 01K3F7Q2XVJ8R5NBWZ4CDMHT9E.
Credential stored in C:\Users\you\AppData\Roaming\medialib-agent\config.json.
Start serving with: agent run
```

Pair as the same Windows account that will run the agent later. The credential
is written into that account's profile, and another account will not find it.

### 4. Try it once by hand

```
C:\medialib\medialib-agent.exe run
```

Leave it for a few seconds. You are looking for a line like
`tunnel established to wss://api.spameri.cz/tunnel`. Windows Firewall may ask
about incoming connections — allow it on private networks so that watching at
home works. Press `Ctrl+C` to stop it.

### 5. Make it start on its own

The agent has no Windows service of its own, so use Task Scheduler. These
commands use Command Prompt syntax — open **Command Prompt as Administrator**,
not PowerShell, because the quoting below is not the same in both.

The straightforward version, which starts the agent when you log in:

```
schtasks /Create /TN "MediaLib Agent" /TR "\"C:\medialib\medialib-agent.exe\" run" /SC ONLOGON /RL HIGHEST /F
```

This leaves a console window open while it runs. That is normal, and closing it
stops the agent.

If the machine reboots without anyone logging in — which is what you want for a
machine that just sits there — use a start-up task with your account's password
stored instead:

```
schtasks /Create /TN "MediaLib Agent" /TR "\"C:\medialib\medialib-agent.exe\" run" /SC ONSTART /RU "%USERNAME%" /RP * /RL HIGHEST /F
```

`/RP *` prompts for your Windows password and stores it so the task can run
without you. It must be the same account you paired with. Do not use `SYSTEM`
here: it has a different profile folder and would not find the credential you
just created.

Start it now without rebooting:

```
schtasks /Run /TN "MediaLib Agent"
```

To allow the home-network connection through the firewall explicitly:

```
netsh advfirewall firewall add rule name="MediaLib Agent" dir=in action=allow protocol=TCP localport=8484
```

### Where things live on Windows

| What | Where |
| --- | --- |
| Configuration and credential | `%AppData%\medialib-agent\config.json` |
| Everything else the agent keeps | `%AppData%\medialib-agent\` |

That folder also holds `state.db` (its record of your files), `lan-cert.pem` and
`lan-key.pem` (the certificate used for watching at home), and cached poster
images.

---

## Linux

### 1. Install ffmpeg

```
sudo apt install ffmpeg          # Debian, Ubuntu, Raspberry Pi OS
sudo dnf install ffmpeg          # Fedora
sudo pacman -S ffmpeg            # Arch
```

On Debian and Ubuntu this pulls in `ffprobe` as well.

### 2. Create a user and install the binary

Running the agent as its own account keeps it away from the rest of your system.

```
sudo useradd --system --create-home --home-dir /var/lib/medialib-agent --shell /usr/sbin/nologin medialib
sudo mkdir -p /opt/medialib-agent
sudo curl -L -o /opt/medialib-agent/medialib-agent https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/medialib-agent-linux-amd64
sudo chmod 0755 /opt/medialib-agent/medialib-agent
sudo chown -R medialib:medialib /opt/medialib-agent
```

On ARM, swap `medialib-agent-linux-amd64` for `medialib-agent-linux-arm64`.

The `chown` on the whole directory is deliberate. When the agent updates itself
it writes the new binary next to the old one and swaps them, which it cannot do
in a directory it does not own. If you would rather install to a root-owned
location such as `/usr/local/bin`, that works too — you just have to install
updates by hand from then on.

### 3. Give it access to your media

The `medialib` account needs to read your video folders. Read access is enough
for scanning and playback. If you plan to use "optimize for streaming" or
Webshare downloads, it needs write access to those folders too.

The simplest approach on a single-user machine is to add it to the group that
already owns the files:

```
sudo usermod -aG "$(stat -c '%G' /srv/media)" medialib
```

Check it worked, using one of your own paths:

```
sudo -u medialib test -r /srv/media && echo readable || echo "not readable"
```

### 4. Pair it

Pair as the `medialib` user, so the credential lands where the service will look
for it:

```
sudo -u medialib env HOME=/var/lib/medialib-agent \
  /opt/medialib-agent/medialib-agent pair --server https://api.spameri.cz --code H4KP-2QRW
```

The credential is written to
`/var/lib/medialib-agent/.config/medialib-agent/config.json` with permissions
`0600`.

### 5. Install the service

Save this as `/etc/systemd/system/medialib-agent.service`:

```ini
[Unit]
Description=MediaLib agent
Documentation=https://github.com/Spameri/MediaLibDocumentation
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=medialib
Group=medialib
# The agent stores its configuration under $HOME and refuses to start
# without one, so set it explicitly rather than relying on the account.
Environment=HOME=/var/lib/medialib-agent
ExecStart=/opt/medialib-agent/medialib-agent run
# Restart on any exit, including a clean one: after installing an update the
# agent exits deliberately so that the new binary is the one that comes back.
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
sudo systemctl enable --now medialib-agent
```

Watch it start:

```
sudo journalctl -u medialib-agent -f
```

Within a few seconds you should see `tunnel established to
wss://api.spameri.cz/tunnel`.

If you run a firewall, allow the home-network port:

```
sudo ufw allow from 192.168.0.0/16 to any port 8484 proto tcp
```

### Where things live on Linux

| What | Where |
| --- | --- |
| Configuration and credential | `$HOME/.config/medialib-agent/config.json` |
| Everything else the agent keeps | `$HOME/.config/medialib-agent/` |

With the service set up as above, `$HOME` is `/var/lib/medialib-agent`.

---

## NAS

A NAS is just Linux with a friendlier front end, so the Linux instructions apply
once you have a shell. Two things are usually harder than on a normal computer:
getting ffmpeg, and getting something to start at boot.

Enable SSH first — on Synology under **Control Panel → Terminal & SNMP**, on
QNAP under **Control Panel → Telnet / SSH** — then connect with
`ssh you@your-nas`.

### ffmpeg on a NAS

Check whether you already have it:

```
which ffmpeg ffprobe
```

Many NAS packages (Plex, Emby, Video Station) ship their own ffmpeg that is not
on your `PATH` and may be a cut-down build. If nothing turns up, the usual route
is [Entware](https://github.com/Entware/Entware/wiki), which gives you a package
manager that survives firmware updates:

```
opkg update
opkg install ffmpeg
```

If you do find an ffmpeg belonging to another package and want to use it, point
the agent at it explicitly rather than moving it. Add these two lines to
`config.json` after pairing:

```json
{
  "ffmpeg_path": "/var/packages/VideoStation/target/bin/ffmpeg",
  "ffprobe_path": "/var/packages/VideoStation/target/bin/ffprobe"
}
```

Be aware that vendor builds are sometimes missing encoders the agent needs. If
transcoding fails but a normal file plays, that is the likely cause.

### Installing and pairing

Most NAS boxes are ARM, so start with the arm64 build. Put it somewhere on the
data volume — the system partition is often small and gets wiped by firmware
updates.

```
mkdir -p /volume1/medialib
curl -L -o /volume1/medialib/medialib-agent https://github.com/Spameri/MediaLibDocumentation/releases/latest/download/medialib-agent-linux-arm64
chmod +x /volume1/medialib/medialib-agent
/volume1/medialib/medialib-agent pair --server https://api.spameri.cz --code H4KP-2QRW
```

If that fails with `cannot execute binary file`, you have an Intel NAS — use
`medialib-agent-linux-amd64` instead.

Your library paths on a NAS look like `/volume1/movies` and `/volume1/tv`
(Synology) or `/share/Multimedia/Movies` (QNAP). Note them down; you will type
them into the web UI in [first-steps.md](first-steps.md).

### Starting it at boot

**If your NAS has systemd** — check with `systemctl --version` — use the unit
file from the Linux section above, adjusting `ExecStart` to
`/volume1/medialib/medialib-agent run` and setting `Environment=HOME=` to a
directory on the data volume, for example `/volume1/medialib/home`. Pair as that
same user with the same `HOME` so the credential lands in the right place.

**On Synology DSM**, the supported route is the built-in scheduler: **Control
Panel → Task Scheduler → Create → Triggered Task → User-defined script**, with
the user set to `root`, the event set to **Boot-up**, and the script:

```
/volume1/medialib/medialib-agent run &
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
[api.spameri.cz/settings/machine](https://api.spameri.cz/settings/machine). Your
machine should be listed with a green **Online** badge, its platform, and the
agent version. This is the check that matters — it means the machine reached the
server and the server can reach it back.

**2. The logs.** The line you want is:

```
tunnel established to wss://api.spameri.cz/tunnel
```

`journalctl -u medialib-agent -f` on Linux; the console window on Windows.

Repeated `tunnel closed: … — reconnecting in 2s` lines mean it cannot get out to
the server. The delay doubles on each failed attempt up to a minute, so it keeps
trying without hammering anything.

**3. The version.** Confirms the binary itself runs:

```
/opt/medialib-agent/medialib-agent version
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

After installing an update it exits so that the service manager starts the new
binary. This is why `Restart=always` is in the unit file above. If a new build
fails to work three times in a row, the agent puts the previous one back by
itself.

To turn this off, add `"disable_auto_update": true` to `config.json` and restart
the agent. You are then responsible for replacing the binary yourself.

## Things worth knowing

- **One machine per account.** Pairing a second machine is refused with "This
  account already has a linked machine. Revoke it first." Unlink the old one
  under Settings → Linked Machine first.
- **The agent only ever connects outward** — to `api.spameri.cz` over HTTPS and
  one long-lived WebSocket. Nothing needs to reach in from the internet, so
  there is nothing to configure on your router.
- **It listens on port 8484** on your local network only, for watching at home.
- **The full scan repeats every six hours.** You can also trigger one at any
  time from the web UI.
- **The credential file is worth protecting.** Anyone who copies
  `config.json` can impersonate your machine. It is written `0600` for that
  reason. If you think it has leaked, unlink the machine in the web UI and pair
  again — the old credential stops working immediately.
