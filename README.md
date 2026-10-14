# pkg-updater

**pkg-updater** is a lightweight utility designed to automatically update Debian packages (`.deb`) from third-party sources (such as GitHub Releases or direct static URLs). It supports local caching, HTTP header-based change detection, version control via `dpkg`, and a built-in **self-update** mechanism. The utility can also seamlessly install applications defined in its configuration file.

---

## 🚀 Key Features

- **Dual Provider Support:**
  * `github` — Automatic release tracking via the GitHub API and downloading of matching asset files (e.g., filtered by architecture pattern like `amd64.deb`).
  * `direct` — Direct package downloading from static URL addresses (with change detection via HTTP headers like `Content-Length`, `ETag`, and `Last-Modified`).
- **Secure Updates:**
  * Integrity validation of `.deb` files prior to installation using `dpkg-deb`.
  * Integrated waiting mechanism for package manager locks (`dpkg`/`apt` lock timeout).
  * Version control (`dpkg-query` / `dpkg-deb`) to prevent redundant re-installations.
- **Flexible Configuration:**
  * Management of check intervals, cache paths, and configuration file conflict resolution behaviors (`--force-confdef`, `--force-confnew`, `--force-confold`).
- **System Integration:**
  * Detailed logging to both `syslog` and the console.
  * Support for delayed execution (`--time-delay`) suitable for background jobs and systemd services.
- **Self-Update Mechanism:**
  * Ability to update itself directly from the GitHub repository.

---

## ⚙️ Configuration

The default configuration file path is: `/etc/pkg-updater/pkg-updater.conf`

### Global Settings

```ini
[global]
initial_delay_minutes = 5
cache_dir = /var/cache/pkg-updater
self_update_interval_days = 0
self_update_headers_days = 7
```

```
# ==============================================================================
# APPLICATION SECTIONS
# ==============================================================================
# Supported Parameters:
#   enabled             : Enable or disable update checks for this section (true/false).
#                         Default: false
#
#   pkg_name            : Exact Debian package name registered in dpkg/apt.
#                         Default: Name of the section header
#
#   provider            : Update provider engine to use.
#                         - 'direct' : Standard download via static direct URL (default if omitted).
#                         - 'github' : Dynamic release lookup via GitHub API.
#
#   dpkg_conf           : Optional behavior for configuration file conflicts during dpkg install.
#                         - (omitted/default) : Installer decides based on package default settings.
#                         - 'old'             : Force keeping existing user configuration file.
#                         - 'new'             : Force overwriting with package's new configuration file.
#
#   url                 : Target URL for the package:
#                         - For provider 'direct': Direct HTTP/HTTPS link to the .deb file.
#                         - For provider 'github': Repository releases page URL.
#
#   asset_pattern       : [GitHub Provider Only] Pattern/substring to identify the correct .deb asset.
#
#   check_version_days  : [GitHub Provider Only] Interval in days between GitHub API release/version checks.
#   check_headers_days  : [Direct Provider Only] Interval in days between HTTP HEAD checks.
#   check_interval_days : [Universal] Forced full file redownload interval in days regardless of headers/API.
#
# NOTE: If interval parameters are omitted, they default to 0 and will be ignored/disabled.
# ==============================================================================
```

Updatable Applications

```Ini
[my-app]
pkg_name = my-app
provider = github
url = https://github.com/username/my-app/releases/latest
asset_pattern = amd64.deb
check_version_days = 3
enabled = true
```

```Ini
[static-app]
pkg_name = static-app
provider = direct
url = https://example.com/downloads/app-latest.deb
check_interval_days = 1
check_headers_days = 7
enabled = true
```


Install-only applications (disabled for auto-update checks):

```Ini
[install-app]
pkg_name = install-app
provider = direct
url = https://example.com/install-app
```

---

## 📖 Usage

The utility supports the following commands and options:

```Plaintext
Usage: pkg-updater [COMMAND] [OPTIONS]

Commands:
  install <APP>         Install specific application defined in config (if missing)
  update <APP>          Check and update specific installed application (or 'pkg-updater')

Options:
  -y, --yes             Automatic yes to prompts (for install command)
  -i, --info            Display status of all apps and global delay in config
  -d, --diag            Diagnose server HTTP capabilities for configured URLs
  -f, --force           Bypass download interval checks and force download/update
  -t, --time-delay [M]  Wait M minutes before checking (or default from config if M omitted)
  -c, --config PATH     Specify alternative path to configuration file
  -v, --verbose         Enable verbose output (detailed logging & file progress)
  -h, --help            Show this help message
```

