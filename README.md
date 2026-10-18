<p align="center">
  <img src="assets/icon.png" alt="Proxy Toolkit" width="110">
</p>

<h1 align="center">Proxy Toolkit</h1>

<p align="center">
  A desktop app for Debian-based Linux that <b>checks proxies</b> and then <b>uses them</b>:
  a local rotating / chaining proxy server.
 
   <p align="center">Python · PyQt6 · asyncio
</p>

---

## Features

**1 · Proxy Checker**
- Load proxies by **drag & drop**, **Browse**, or **paste** (all three are merged, duplicates removed)
- Checks HTTP / HTTPS / SOCKS4 / SOCKS5 — protocol is auto-detected when no scheme is given
- Real HTTPS request through each proxy; the exit IP must differ from yours
- Anonymity level (transparent / anonymous / elite), multi-target test (ipify, Cloudflare, Google, custom URL)
- Remote-DNS support check, stability rounds, speed score
- Country / city / ISP / datacenter lookup
- Sort, filter, favourites, blacklist, export to JSON / CSV / TXT / per-country lists

**2 · Proxy Rotation / Chain**
- One local port (default `127.0.0.1:1080`) that speaks **SOCKS5, SOCKS4/4a and HTTP(S)**
- Rotation modes: round-robin, random, fastest first, sticky per domain, **manual (hold one IP)**
- Chain modes: strict, dynamic, random — like `proxychains`, but built in, with a drag & drop chain builder
- **🔄 Change IP** button: tests the next live proxy, switches to it and (optionally) closes open connections
- **Kill-switch**: the server never connects directly — no live proxy means the connection is refused
- Background health monitor: dead proxies leave the pool, recovered ones return
- Filters by country / minimum score, "require remote DNS" to avoid DNS leaks

## Requirements

- Debian / Ubuntu or another Debian-based distribution (tested with Python 3.12)
- Python 3, `git`, and the packages below

## Install

```bash
# 1. system packages
sudo apt update
sudo apt install git python3 python3-venv python3-pyqt6 libxcb-cursor0

# 2. download
git clone https://github.com/saimum10/proxy-toolkit.git
cd proxy-toolkit

# 3. first run: creates a .venv, installs the Python dependencies, starts the app
./run.sh

# 4. Install the launcher (application menu + icon)
./install_launcher.sh

# 5.delete installer file
cd ..
rm -rf proxy-toolkit

```

The first start takes a little longer because the dependencies are installed into the local `.venv`
(nothing is installed system-wide).


## Update

```bash
cd proxy-toolkit
git pull
rm -f .venv/.deps_ok      # re-installs the dependencies on the next start
./run.sh
```

## Uninstall (delete everything)

```bash
# launcher + icons
rm -f ~/.local/share/applications/proxy-toolkit.desktop
rm -f ~/.local/share/icons/hicolor/*/apps/proxy-toolkit.png
rm -f ~/.local/share/icons/hicolor/icon-theme.cache
update-desktop-database ~/.local/share/applications 2>/dev/null || true

# saved favourites / blacklist (optional)
rm -rf ~/.config/proxy-toolkit

# the app itself (the cloned folder, including its .venv)
cd .. && rm -rf proxy-toolkit
```

## Usage

### 1 · Proxy Checker
1. Drop `.txt` files on the window, click **Browse files…**, or paste proxies (one per line):
   ```
   1.2.3.4:8080
   socks5://1.2.3.4:1080
   user:pass@1.2.3.4:3128
   1.2.3.4:8080:user:pass
   ```
2. Adjust the settings and click **Start check**.
3. Sort / filter the table. Right-click rows for favourite, blacklist, *Add to chain* and copy.
   **Export** saves the visible rows as JSON, CSV, TXT or per-country lists.

### 2 · Proxy Rotation / Chain
1. Get proxies into the pool — any of these:
   - paste / browse / drop a list into **Add proxies** and press **Check & add**
   - tick **Add live proxies from the Checker tab automatically** (same switch on both tabs)
   - press **Add live proxies from Checker**
2. Choose a mode and press **Start server**.
3. Point your browser, curl or Burp at `127.0.0.1:1080`:
   ```bash
   curl --proxy socks5h://127.0.0.1:1080 https://api.ipify.org
   curl --proxy http://127.0.0.1:1080    https://api.ipify.org
   ```
   Firefox: *Settings → Network Settings → Manual proxy* → SOCKS Host `127.0.0.1`, Port `1080`,
   SOCKS v5, and tick **Proxy DNS when using SOCKS v5**.

### Modes

| Mode | What it does |
|---|---|
| Round-robin | Every new connection uses the next live proxy in turn |
| Random | Every new connection uses a random live proxy |
| Fastest first | Always the lowest-latency proxy; if it fails, the next fastest |
| Sticky per domain | One website keeps the same exit IP, other sites rotate — good for logins |
| Manual · hold one IP | The exit IP stays fixed until you press **Change IP** |
| Chain · strict | you → A → B → C → site; if any hop is down the connection is blocked |
| Chain · dynamic | Same, but dead hops are skipped |
| Chain · random | A new random chain of N live proxies for every connection |

Strict and dynamic chains use the **Chain builder** (add from the pool, drag to reorder).

## Safety notes

- The **kill-switch** protects traffic that is sent to this local proxy. Programs that bypass it are
  not covered.
- Browsers can leak your real IP through **WebRTC**, which no proxy can stop. Disable WebRTC in the
  browser (Firefox: `media.peerconnection.enabled = false`) and verify on a leak-test site.
- **Remote DNS**: hostnames are sent to the proxy, so no local DNS lookup happens — as long as the
  client sends hostnames (Firefox: *Proxy DNS*, curl: `socks5h://`).
- The anonymity level is measured with a plain-HTTP header-echo page (httpbin.org /
  postman-echo.com). SOCKS proxies relay raw TCP, so they are always reported as *elite*.
- Country / city / ISP data comes from [ip-api.com](https://ip-api.com) (free tier: 15 batch
  requests per minute, non-commercial use).
- Use this tool only with proxies you are allowed to use and on systems you are authorised to test.

## Troubleshooting

| Problem | Fix |
|---|---|
| `Could not load the Qt platform plugin "xcb"` | `sudo apt install libxcb-cursor0` |
| `Permission denied` when running `./run.sh` | `chmod +x run.sh install_launcher.sh main.py` (or start it with `bash run.sh`) |
| Launcher icon missing in the dock (Wayland) | Run `./install_launcher.sh` and start the app from the menu |
| `Cannot start the server` / port in use | Change the port in *Listen address* |
| A site still shows your real IP | Check the browser really uses the proxy (Firefox has its own proxy settings), then check WebRTC |
| Country filter excludes everything | Enable *Country / city / ISP lookup* in the Checker settings and check again |
| A part of the window is cut off | Increase the window size; the side panels scroll and follow the system font size |

