# Tuxxin QR Track — v2

![tuxxin-qr-track](https://github.com/user-attachments/assets/3bfd4d83-2f4f-47ba-94d8-4f608ba5f4b9)

A lightweight, self-hosted dynamic QR code tracking system built with PHP and SQLite.
Create, manage, and track QR codes for URLs, WiFi networks, vCards, and more — all from a secure, dark-mode dashboard or via a REST API.

---

## What's New in v2

- **Edit QR Codes** — Update the title and content of any QR code without changing its UUID or the printed code itself.
- **Trash & Restore** — Soft-deleted QR codes now appear in a Trash view and can be restored at any time.
- **Logout Button** — Admins can now explicitly end their session from the dashboard header.
- **API Rate Limiting** — Configurable per-IP throttle (`API_THROTTLE_LIMIT` / `API_THROTTLE_WINDOW` in `config.php`).
- **API Bug Fix** — "Get Single QR" with a blank UUID no longer silently returns all entries; it now returns `400 Bad Request`.
- **API Key Never Exposed** — The Live Console routes requests through a server-side proxy (`api_console_proxy.php`) so the API key is never embedded in browser HTML.
- **Constant-Time API Auth** — Replaced `!==` with `hash_equals()` to prevent timing-based key extraction attacks.
- **GET Key Fallback Removed** — API key is now accepted only via the `X-Api-Key` header; `?api_key=` GET support is dropped (prevents key leakage in server logs).
- **Token Optimization** — API reuses valid existing image tokens instead of creating a new one on every request, preventing `api_tokens` table bloat.
- **CSRF Protection** — All dashboard form submissions are protected with CSRF tokens.
- **Session Hardening** — Cookies are `HttpOnly`, `SameSite=Strict`, and `Secure` (on HTTPS), with a configurable idle timeout.
- **File Upload MIME Validation** — Logo uploads now verify actual file content via `finfo`, not just the file extension.
- **Type Whitelist** — QR type values are validated server-side against the allowed list before use.
- **Format Parameter Whitelist** — `generate_image.php` restricts `?format=` to `png` or `jpg` only.
- **Configurable Disabled Page** — Set `DISABLED_REDIRECT_URL` in `config.php` to redirect inactive QR scans to your own URL. Leave empty for a built-in "Link Inactive" screen with no redirect.
- **No Indexing** — `robots.txt`, `X-Robots-Tag` HTTP header, and `<meta name="robots" content="noindex, nofollow">` added across all pages.
- **Asset Path Fixes** — Logo and favicon use `BASE_URL`, so subdirectory installs render correctly on all pages.
- **Input Length Limits** — All form fields have `maxlength` attributes.
- **Redirect Fix** — Post-create redirect now uses `BASE_URL` (fixes broken redirect on subdirectory installs).
- **Dead Code Removed** — Duplicate modal and JS definitions removed from `themes/footer.php`.

---

## Features

- **Dynamic QR Codes** — Update the destination or content of any QR code instantly via the Edit button. The printed QR code never changes.
- **Detailed Tracking**
  - Logs real user IP (even behind Cloudflare Tunnels).
  - Captures geo-location (City, Region, Country, ISP) via ip-api.com with per-scan DB caching.
  - Detects device type via User Agent.
- **Privacy Aware** — Handles traffic from Apple iCloud Private Relay, Cloudflare, and other proxies.
- **Multiple Content Types**
  - 🔗 **URL** — Standard website links.
  - 📶 **WiFi** — Direct connection QR codes (WPA/WEP/Open). *(Not trackable — WiFi codes embed credentials directly.)*
  - 👤 **vCard** — Add contacts directly to phone address books.
  - 📍 **Maps, Phone, SMS, Email, Social Media.**
- **Secure API**
  - Full REST API for creating and retrieving QR codes.
  - Temporary expiring image tokens for secure external image downloads.
  - Built-in Live Console with server-side proxy (API key never reaches the browser).
  - Configurable per-IP rate limiting.
- **Customization** — Upload a PNG or JPG logo to embed in the center of any QR code. *(Not currently supported via API.)*
- **Management Dashboard**
  - Create, edit, and delete QR codes.
  - Real-time scan statistics with geo-location.
  - Soft Delete & Restore (Trash view).
  - Enable/Disable toggle (redirects scans to a configurable "Link Inactive" page).
  - Logout button.

---

## Requirements

- **OS:** Linux (Debian/Ubuntu recommended)
- **Web Server:** Apache (with `mod_rewrite` and `mod_headers`) or Nginx
- **PHP:** 8.0+ with extensions: `sqlite3`, `gd`, `curl`, `mbstring`, `fileinfo`
- **Database:** SQLite 3 (no separate server required)
- **Composer:** Required for the QR code generation library

---

## Quick Install

### 1. Clone & Install Dependencies

```bash
git clone https://github.com/tuxxin/qr-track.git
cd qr-track
composer install
```

### 2. Create Storage Directories Outside the Web Root

```bash
# Place these OUTSIDE your web-accessible folder for security
mkdir -p /home/youruser/db /home/youruser/tmp
chmod 775 /home/youruser/db /home/youruser/tmp
```

### 3. Configure `config.php`

Open `config.php` and update the following settings:

| Setting | Description |
|---|---|
| `ADMIN_USER` | Dashboard login username |
| `ADMIN_PASS` | Dashboard login password |
| `API_KEY` | Random secret key for the REST API — use the generator on the API Docs page |
| `BASE_URL` | Full URL with no trailing slash (e.g. `https://yourdomain.com` or `https://yourdomain.com/qr-track`) |
| `DB_PATH` | Absolute path to your SQLite file (e.g. `/home/youruser/db/tuxxin_qr.sqlite`) |
| `LOGO_DIR` | Absolute path to your logo upload folder (e.g. `/home/youruser/tmp`) |
| `TIMEZONE` | PHP timezone string (e.g. `America/New_York`) |
| `USE_CLOUDFLARE_TUNNEL` | `true` if your server is behind a Cloudflare Tunnel or similar proxy |
| `DISABLED_REDIRECT_URL` | URL to redirect inactive QR code scans to, or `''` to show the built-in page |
| `API_THROTTLE_ENABLED` | `true` to enable per-IP API rate limiting |
| `API_THROTTLE_LIMIT` | Max API requests per window per IP (default: `60`) |
| `API_THROTTLE_WINDOW` | Rate limit window in seconds (default: `60`) |
| `SESSION_LIFETIME` | Idle session timeout in seconds (default: `7200` = 2 hours) |

### 4. Configure `.htaccess`

Update `RewriteBase` to match your install location:

```apache
# Root install
RewriteBase /

# Subdirectory install
RewriteBase /qr-track/
```

### 5. Set Permissions

```bash
# The web server user (e.g. www-data) must be able to write to db/ and tmp/
chown -R youruser:www-data /home/youruser/db /home/youruser/tmp
chmod 775 /home/youruser/db /home/youruser/tmp
```

### 6. Initialize the Database

The SQLite database and all tables are created automatically on first page load. Simply visit `BASE_URL` in your browser.

### 7. Log In

Navigate to `BASE_URL/login.php` and sign in with the credentials you set in `config.php`.

---

## Nginx Configuration

```nginx
location /qr-track/ {
    try_files $uri $uri/ /qr-track/index.php?$query_string;

    location ~ ^/qr-track/p/([a-zA-Z0-9]+)$ {
        rewrite ^/qr-track/p/([a-zA-Z0-9]+)$ /qr-track/proxy.php?id=$1 last;
    }

    # Block sensitive files
    location ~ \.(sqlite|db|log)$ { deny all; }
    location = /qr-track/config.php { deny all; }
}
```

---

## API Reference

All requests require the `X-Api-Key` header.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api.php` | List all QR codes |
| `GET` | `/api.php?uuid={uuid}` | Get a single QR code by UUID |
| `POST` | `/api.php` | Create a new QR code |

See the built-in **API Docs & Console** at `BASE_URL/api_instructions.php` for full payload examples and a live testing tool.

---

## Security Notes

- All dashboard pages require an authenticated session.
- API key is verified using `hash_equals()` (constant-time comparison).
- The API key is never sent to the browser — the Live Console uses a server-side proxy.
- CSRF tokens protect all form submissions.
- Session cookies are `HttpOnly`, `SameSite=Strict`, and `Secure` on HTTPS.
- Logo uploads are validated by actual MIME type (not just extension).
- `config.php`, `.sqlite`, and `.db` files are blocked from direct web access via `.htaccess`.
- `robots.txt` and `X-Robots-Tag` headers prevent search engine indexing of all pages.

---

*Built by [Tuxxin](https://tuxxin.com)* | *Live Demo [QR-Track.Tuxxin.com](https://qr-track.tuxxin.com)*
