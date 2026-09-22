# TeamTalk Media Streamer

A blazing-fast media streaming bot for [TeamTalk](https://bearware.dk/) servers, built in Rust. Supports **Spotify**, **YouTube**, and **Direct Audio Links / Web Radios**.

[![License: Freeware](https://img.shields.io/badge/License-Freeware-green.svg)](#license--terms-of-use)

**No virtual audio devices, no loopback cables, no routing setup.** The bot injects decoded PCM straight into TeamTalk's audio mixer, so there is nothing to configure on the audio side — install it, point it at a server, and it plays.

---

## Supported Services

### Spotify
- Tracks, albums, playlists, search, liked songs, and radio recommendations.
- > A **Spotify Premium** account is required — free accounts will not work.

### YouTube & YouTube Music
- Videos, Shorts, playlists, albums, search, and audio extraction played through [yt-dlp](https://github.com/yt-dlp/yt-dlp).
- YouTube playback uses **cookies** to play reliably. Export them with a browser extension:
  1. Install a cookies-export extension — **Get cookies.txt LOCALLY** ([Chrome / Edge](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc), or the equivalent for Firefox).
  2. Open a **private / incognito** window and sign in to YouTube.
  3. With the YouTube tab open, use the extension to export a `cookies.txt` file.
  4. **Close the incognito window** (do *not* log out) so the exported cookies stay valid.
  5. Put the file where the bot looks for it — `data/config/cookies.txt` (Windows) or `~/.config/TeamTalkMediaStreamer/config/cookies.txt` (Linux) — or set `youtubeCookiesFile` in your config to its path.

### Direct Audio Links & Live Web Radios
- Direct streaming from HTTP/HTTPS audio links (Icecast, Shoutcast, ZenoFM, direct MP3/AAC streams, online radio feeds).
- Seamless track detection and live icy-metadata extraction for real-time station and title display.

### Limiting a Bot to Specific Services
A bot can be restricted to specific services. Setup asks during configuration on Windows and on Linux. A YouTube-only bot never touches the Spotify login saved on your machine. Using `spt`, `ytv`, or `ytm` on a bot without that service will notify the user, and `a` (*About*) shows what the bot offers.

---

## Requirements

- A **TeamTalk 5 server** to connect to, and a TeamTalk account for the bot.
- **Linux runtime packages:** `libpulse0` (for TeamTalk SDK), `ffmpeg` (for direct stream decoding), and `ca-certificates` (for HTTPS stream TLS verification).

---

## Installation

Download the latest build from the [**Releases page**](https://github.com/fauzan-january/TeamTalkMediaStreamer/releases).

### Windows

1. Download `TeamTalkMediaStreamer-windows-x86_64.zip`, extract it, and run `TeamTalkMediaStreamer.exe` — a tray icon appears.
2. On first run, it prompts you to create a config (a setup dialog). Fill it in and the bot connects.
3. Use the tray menu for **Spotify Auth**, **Install YouTube tools**, and each bot's start / stop / restart / logs / edit / **Remove Server**.

### Linux (x86_64, Ubuntu 22.04+ / glibc)

1. Install runtime dependencies:
   ```bash
   sudo apt update && sudo apt install -y libpulse0 ffmpeg ca-certificates
   ```
   - `libpulse0`: needed by the TeamTalk SDK.
   - `ffmpeg`: needed for decoding direct audio streams and web radios.
   - `ca-certificates`: needed for TLS/SSL certificate verification on HTTPS streams.
2. Download and extract the archive TeamTalkMediaStreamer:

Download from GitHub:
   ```bash
   wget https://github.com/fauzan-january/TeamTalkMediaStreamer/releases/download/v1.0.2/TeamTalkMediaStreamer-linux-x86_64.tar.gz
   ```
   Extract the archive:
   ```bash
   tar -xzf TeamTalkMediaStreamer-linux-x86_64.tar.gz
   ```
3. Put the binary on your `PATH` (installs itself into `~/.local/bin`):
   ```bash
   ./TeamTalkMediaStreamer install
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.profile
export PATH="$HOME/.local/bin:$PATH"
   ```
4. Run the interactive setup wizard:
   ```bash
   TeamTalkMediaStreamer
   ```
5. Install YouTube tools (yt-dlp, bgutil-pot, and Deno runtime):
   ```bash
   TeamTalkMediaStreamer yt install
   ```
6. *(Optional)* Install systemd service for 24/7 background operation:
   ```bash
   TeamTalkMediaStreamer service install
   TeamTalkMediaStreamer start myserver
   ```

### Linux (aarch64 / Raspberry Pi 64-bit)

Runs on Raspberry Pi (Pi Zero 2 W through Pi 5) on **64-bit Raspberry Pi OS** (Debian 12 / bookworm or newer). Same steps as x86_64 using the `aarch64` archive:
```bash
sudo apt update && sudo apt install -y libpulse0 ffmpeg ca-certificates
wget https://github.com/fauzan-january/TeamTalkMediaStreamer/releases/download/v1.0.2/TeamTalkMediaStreamer-linux-x86_64.tar.gz
tar -xzf TeamTalkMediaStreamer-linux-aarch64.tar.gz
./TeamTalkMediaStreamer install
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.profile
export PATH="$HOME/.local/bin:$PATH"
```

---

## Updating the Bot

The bot features a built-in self-updater: it checks GitHub for a newer release, shows what changed, verifies cryptographic checksums, and swaps the binary in place.

- **Windows:** The tray checks on startup and offers updates; there is also a **Check for updates** item in the tray menu.
- **Linux:** Run `TeamTalkMediaStreamer update`. If bots are running as systemd services, it offers to restart them on the new version.
- **Updating YouTube Tools:** Run `TeamTalkMediaStreamer yt update`, or choose **Update tools** in the tray menu.

---

## Running Multiple Bots

Multiple instances are supported out of the box — one per config file, each with its own server and account.

- **Windows:** The tray manages all bots. Right-click → **Add Server** once per bot; every config shows up in the tray menu with its own start / stop / restart / logs, and **Remove Server**.
- **Linux:** Create each bot's config with `TeamTalkMediaStreamer add <name>`. Manage instances with `list`, `start`, `stop`, `restart`, `logs`, `watch`, `edit`, `remove`, and `doctor`.

---

## Configuration

Configs are JSON files generated by the setup wizard. Locations:
- **Windows:** `data\config\<name>.json` (`data` sits next to the executable)
- **Linux:** `~/.config/TeamTalkMediaStreamer/config/<name>.json`

Common fields you might edit:

| Field | What it does |
|---|---|
| `host` | TeamTalk server address |
| `tcpPort` / `udpPort` | Server ports (usually both `10333`) |
| `botName` | The bot's display name / nickname in the channel |
| `clientName` | The bot's client information name (e.g. `TeamTalkMediaStreamer`) |
| `username` / `password` | The bot's TeamTalk login credentials |
| `ChannelName` | Channel to join, e.g. `/Music` |
| `ChannelPassword` | Password if the channel is protected |
| `spotifyQuality` | `NORMAL`, `HIGH`, or `VERY_HIGH` |
| `maxVolume` | Volume cap, 0–100 |
| `defaultService` | `Spotify`, `YouTube`, or `Direct` on startup |
| `enabledServices` | Which services this bot may use, e.g. `["spotify", "youtube", "direct"]` |
| `youtubeCookiesFile` | Path to your YouTube `cookies.txt` (optional) |
| `adminMode` | Who may use admin commands: `Everyone`, `TtRights`, `List`, or `Both` (default) |
| `admins` | Usernames treated as admins (used by `List` / `Both`) |
| `defaultLanguage` | Default language code for bot replies, e.g. `id`, `en`, `es`, `pt`, `ru` |
| `autoReconnect` | Reconnect automatically if disconnected |

---

## Admin Permissions

Admin commands (`sd`, `rs`, `jc`, `cdl`, `sds`, `cci`, `l`, `hs`, `gci`, `ua`, `ucr`) can be restricted to authorized users. Set `adminMode` in your config:
- **Everyone** — No restrictions; any user can run admin commands.
- **TtRights** — Accounts with admin rights on the TeamTalk server.
- **List** — Only usernames explicitly listed in `admins`.
- **Both** (default) — Server admins *or* listed usernames.

---

## Multi-Language Support (i18n)

Bot replies are available in **5 built-in languages**:
- **Indonesian (`id`)**
- **English (`en`)**
- **Spanish (`es`)**
- **Portuguese (`pt`)**
- **Russian (`ru`)**

Users can set their own language with `cl <code>` (e.g. `cl id`, `cl en`), or reset with `cl clear`. Admins can set the server-wide default language with `cdl <code>`. Custom translations can be placed in `lang/<code>.lang`.

---

## Commands Reference

Send commands to the bot via **private message** (e.g. `p song`, `h`) or in a **channel message** prefixed with `/` (e.g. `/p song`, `/h`).

### Playback & General Commands

| Command | Description |
|---|---|
| `gs` | Show greeting message and quick start guide |
| `a` | Show bot information (*About*) |
| `dci` | Show developer contact information |
| `log` | Show changelog / history of changes |
| `h` / `h <command>` | Show help overview or detailed help for a specific command (e.g. `h q`) |
| `p [query/url]` | Play track from search query or URL immediately (replaces currently playing track). Without arguments, toggles play/pause |
| `pn [query/url]` | Add track from search query or URL to queue to play next |
| `th [count]` | Play today's Top Hits playlist and replace current queue |
| `lk` | Play Liked Songs (Spotify service only) |
| `src <query>` | Search and pick a track from numbered search results to play (`c` to cancel) |
| `rp` | Restart current track from the beginning |
| `s` | Stop playback and clear the queue |
| `n` | Skip to the next track |
| `b` | Return to previous track (or restart if >3s in) |
| `sn [seconds]` | Seek playback forward by specified seconds (default 10s) |
| `sb [seconds]` | Seek playback backward by specified seconds (default 10s) |
| `cti` | Show current track information (title, artist, position, duration, mode) |
| `q` | Show current playback queue |
| `qc` | Clear all upcoming tracks from queue |
| `q rm <index>` | Remove a track from queue by its index number |
| `v [0-100]` | Show or set playback volume |
| `m [st\|rt\|tl\|rtl]` | Set playback mode (`st` = Single Track, `rt` = Repeat Track, `tl` = Track List, `rtl` = Repeat Track List) |
| `sfl` | Toggle queue shuffle on / off |
| `ap` | Toggle autoplay recommendations on / off |
| `dl` | Download current track or playlist to the TeamTalk channel |
| `gl` | Get web URL of the current track |
| `lrc [query]` | Show lyrics for current playing track or searched track |
| `srl [1-20]` | Show or set search results limit |

### Services & Session Commands

| Command | Description |
|---|---|
| `spt` | Switch service to Spotify |
| `ytv` | Switch service to YouTube Video |
| `ytm` | Switch service to YouTube Music |
| `st` | Show uptime and session statistics |
| `cl [code]` | Set personal language preference (`cl clear` to reset) |

### Admin Commands

| Command | Description |
|---|---|
| `sds` | Show and modify server configuration interactively |
| `cci` | Modify bot client information interactively (nickname, status, gender, client name) |
| `cdl [code]` | Set server default language |
| `ua [+user / -user]` | Add, remove, or list authorized bot administrators |
| `gci [path]` | Get channel ID of current channel or specified path |
| `jc <path/ID[\|password]>` | Join a specific channel |
| `hs` | Hide or show bot playback status |
| `l` | Lock or unlock bot for non-admin users |
| `ucr` | Toggle unknown command response on / off |
| `rs` | Restart the bot |
| `sd` | Shut down the bot |

## Credits & Attribution

- **Modified by:** Fauzan, S.Kom. | PT Kemenangan Digital Indonesia ([GitHub](https://github.com/fauzan-january/TeamTalkMediaStreamer/))
- **Original Project:** Based on `ttspotify-rs` by LuciferM242 ([GitHub](https://github.com/LuciferM242/ttspotify-rs))

---

## License & Terms of Use

This software is distributed as **Free to Use (Freeware)** for personal and community use.

- **Disclaimer of Warranty:** The software is provided "as is", without warranty of any kind, express or implied. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability.
- **Attribution:** Built upon the original foundation created by LuciferM242 (`ttspotify-rs`).
