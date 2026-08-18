# AGENTS.md

## Cursor Cloud specific instructions

This repository is **only a WordPress theme** (`TSURUTA LABO`, theme slug `tsurutalabo`). WordPress core, the database, and plugins are **not** part of the repo — they are provided by the Cloud Agent environment snapshot. The system packages (PHP 8.3, MariaDB, WP-CLI) and a fully configured WordPress install already exist in the environment; the startup update script only refreshes the repo's npm dependency (`svgo`).

### Where things live
- Repo (this theme): `/workspace`
- WordPress install: `/var/www/html` (WP core, plugins, uploads, DB config — captured in the snapshot, not in git)
- The theme is symlinked in: `/var/www/html/wp-content/themes/tsurutalabo -> /workspace`. Because it is a symlink, **editing files in `/workspace` is reflected immediately** with no build/copy step.
- Custom post types `add_residence` and `add_link_banner` (used by `front-page.php` / `components/banner.php`) are registered by a dev-only must-use plugin at `/var/www/html/wp-content/mu-plugins/tsuruta-dev-cpt.php`. In production these come from a separate plugin, not from this theme.

### Starting the services (needed on every fresh VM boot)
Neither MariaDB nor the web server auto-start. Start them with:
```bash
# 1) Start MariaDB (data dir persists in the snapshot)
sudo mariadbd-safe >/tmp/mariadb.log 2>&1 &
sleep 8

# 2) Start the WordPress dev server on port 8080 (run inside the WP install dir)
cd /var/www/html && wp server --host=0.0.0.0 --port=8080
```
Use `wp server` (WP-CLI's router-aware PHP built-in server) rather than a bare `php -S`, otherwise permalinks/rewrites break. Run WP-CLI from `/var/www/html`.

### Accessing the site
- Front site: `http://localhost:8080/`
- Admin: `http://localhost:8080/wp-admin/` — user `admin`, password `admin123` (local dev only).
- DB: name `wordpress`, user `wpuser`, password `wppass`, host `localhost`.

### Lint / build / test
- **Lint (PHP):** no linter is configured in the repo. Use `php -l <file>` for syntax checks (all templates currently pass).
- **Build:** none required — `css/*.css` and `js/*.js` (UIkit) are committed pre-built. `svgo` (via `npm install` / `npx svgo`) is an optional tool for compressing SVGs in `images/`; there is no dev-server/build pipeline.
- **Tests:** there is no automated test suite (no PHPUnit/Jest/Playwright). Verify changes by loading pages in the running site.

### Notes / gotchas
- Commit messages must be in Japanese (see `.cursorrules`).
- Editing `functions.php` or template files takes effect on the next page load (no restart needed). Restart `wp server` only if you change PHP that runs at bootstrap in an unexpected way or need to clear a fatal.
- The theme references external services (Google Tag Manager/Analytics, Google Maps embed, and the reservation app at `yoyaku.tsurutalabo.com`); these are external and not runnable locally, but they do not block local theme rendering.
