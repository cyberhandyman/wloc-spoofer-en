# WLOC Location Spoofer - Guide

[简体中文](shortcut-guide.md) · **English**

## How it works

```
User opens the picker page in mobile Safari
  → picks a location on the map / searches a place name / pastes a map link
  → taps "Save to Device"
  → the page requests https://gs-loc.apple.com/wloc-settings/save?lon=x&lat=y
  → the proxy module intercepts the request → wloc-settings.js writes to $persistentStore
  → next time Apple location is triggered → wloc.js reads the coordinates → modifies the location response
```

If the module is not enabled → the request is not intercepted → the page prompts you to check the MITM/module configuration.

---

## Usage

### 1. Install the module (one-time)
Subscribe to the module for your platform and enable MITM.

### 2. Open the picker page
Open the public picker page in Safari (adding it to the Home Screen is recommended):
```
https://your-worker-domain/
```

> The Worker is a purely static page and stores no data. Coordinates are written directly to your device's local storage.

### 3. Choose a location
- **Tap the map** — pick directly
- **Search a place name** — type something like "The Bund, Shanghai"
- **Paste a link** — copy a share link from Apple Maps / Google Maps / Amap / Baidu
- **Current location** — use browser geolocation

### 4. Save to Device
Tap "Save to Device" → a ✓ means success.

---

## Deploy the public picker page

The Worker is a purely static page service and needs no bindings:

```bash
cd worker
npx wrangler deploy
```

Or in the CF Dashboard → Workers → create a new Worker → paste `wloc-worker.js` → deploy.

No KV, no database, no environment variables required.

---

## Module configuration

The module contains two script rules (configured automatically, no user action needed):

| Rule | Type | Path | Purpose |
|------|------|------|------|
| Apple WLOC | http-response | `/clls/wloc` | Modify the location response |
| WLOC Settings | http-request | `/wloc-settings/save` | Receive writes from the picker page |

MITM hostnames: `gs-loc.apple.com, gs-loc-cn.apple.com` (already included in the module)

---

## Troubleshooting save failures

When the page shows a red banner, check:
1. **Module enabled** — confirm the WLOC module toggle is on in your proxy tool
2. **MITM certificate** — the CA certificate is installed and trusted
3. **MITM hostname** — includes `gs-loc.apple.com`
4. **Proxy connection** — the current network is routed through the proxy (Safari requests go through the proxy)

---

## Alternative: manual editing (BoxJS)

If you prefer not to use the picker page, you can edit `wloc_settings` directly in BoxJS:
```json
{"longitude":121.4737,"latitude":31.2304,"accuracy":25}
```

Priority: saved coordinates > module parameters > default value
