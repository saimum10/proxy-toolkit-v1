# Proxy Toolkit

Debian, Python 3 + PyQt6 + asyncio.  Two parts: **Checker** and **Proxy Server** (rotation / chains).

## Install (once)
    sudo apt install python3-venv python3-pyqt6 libxcb-cursor0
    cd proxy-toolkit
    ./run.sh          # first run creates .venv and installs the rest

Optional menu entry:  ./install_launcher.sh

## 1 · Checker tab
1. Drop .txt files onto the window, click "Browse files…", paste with the "Paste" button / Ctrl+V,
   (all three are merged; duplicates and blacklisted entries are removed).
2. Adjust settings, click "Start check".
3. Sort / filter the table, right-click rows (favourite, blacklist, add to chain, copy).
   Export visible rows as JSON / CSV / TXT or per-country lists.

Favourites and blacklist: ~/.config/proxy-toolkit/lists.json

## 2 · Proxy Server tab
1. Proxies enter the pool in three ways: paste / browse / drop a list into "Add proxies" (it is
   checked first), automatically after each Checker run (tick-box "Add live proxies … automatically",
   present on both tabs), or with the button "Add live proxies from Checker".
2. Pick a mode, press "Start server". Default address: 127.0.0.1:1080
   (SOCKS5, SOCKS4/4a and HTTP/HTTPS proxy on the same port).
3. Point your browser / curl / burp to it, e.g.
       curl --proxy socks5h://127.0.0.1:1080 https://api.ipify.org
       curl --proxy http://127.0.0.1:1080   https://api.ipify.org
   Firefox: SOCKS v5 + "Proxy DNS when using SOCKS v5".

Modes
- Round-robin / Random / Fastest first / Sticky per domain (same site = same exit IP)
- Manual · hold one IP: the exit IP stays fixed until you press **🔄 Change IP**. The button
  tests the next live proxy with a real request, switches to it and (optionally) closes open
  connections so the browser reconnects through the new IP. Pressing it in another single-proxy
  mode switches to Manual automatically. If the held proxy dies, the next live one takes over.
- Chain · strict   : you → A → B → C → site; any hop down = connection blocked
- Chain · dynamic  : same, but dead hops are skipped
- Chain · random   : new random chain of N live proxies per connection
  Build strict / dynamic chains in the Chain builder (drag to reorder, or use the arrows).

Safety
- Kill-switch: the server never connects directly. No eligible proxy = connection refused.
  (It protects traffic sent to this proxy; programs that bypass it are not covered.)
- "Require remote DNS": hostnames go only through proxies that resolve DNS remotely.
- Health monitor: pool re-tested in the background; dead proxies leave the active pool and
  return when they work again. A failed connection triggers an immediate re-test.
- Country filter needs the Country lookup (Checker settings) to have run.

## Notes
- Anonymity level is measured with a plain-HTTP header-echo page (httpbin.org / postman-echo.com).
  SOCKS proxies relay raw TCP, so they are always reported as "elite".
- "Remote DNS" = the proxy resolves hostnames itself (socks5h / socks4a / HTTP CONNECT).
- Country / city / ISP come from ip-api.com (free tier: 15 batch requests/min, non-commercial).
