# PHP Hardening Template (`php.ini`)

This repository provides an opinionated `php.ini` template focused on hardening PHP for:

- Shared hosting and managed hosting providers
- Multi-tenant SaaS platforms
- Popular PHP-based CMS (WordPress, Joomla, Drupal, etc.)

The goal is to reduce the attack surface of PHP by:

- Disabling dangerous functions
- Restricting filesystem access
- Hardening session and cookie handling
- Providing production-safe error and logging settings

> ⚠️ This is a baseline hardening template. You **must** adapt paths and some settings (marked with `ADAPT`) to your environment.

---

## Goals

- Improve default security posture for PHP in production environments.  
- Provide a sane baseline for shared/multi-tenant hosting, where you cannot fully trust every application.  
- Remain compatible with mainstream CMS platforms by default (WordPress, etc.).

---

## Compatibility

- Target: PHP 8.1+ (should also work on recent 8.x versions).  
- Some legacy directives that were removed in PHP 7.1+ (e.g. session hash settings) are **not used** here to avoid warnings.  
- Designed for Linux environments (paths and examples assume a typical Linux filesystem layout).

---

## Usage

1. Download `php.ini.hardened`.
2. Review all `ADAPT:` comments and adjust:
   - `open_basedir`
   - `session.save_path`
   - log paths (if you do not log to stdout/stderr)
   - timezone.
3. Either:
   - Replace your existing `php.ini` with this file, **or**
   - Include it as an additional configuration file (e.g. `/etc/php.d/security.ini`, `/etc/php/8.2/fpm/conf.d/zz-hardening.ini`).
4. Restart PHP-FPM / Apache / Nginx+PHP after applying the configuration.

---

## Profiles

This template is tuned as a **Balanced** profile:

- Good security defaults.
- Still compatible with popular CMS and typical shared hosting workloads.

You can easily derive other profiles:

- **Strict**:  
  - `allow_url_fopen = Off`  
  - more aggressive `open_basedir`  
  - `session.cookie_samesite = "Strict"`  
  - smaller `upload_max_filesize` and `post_max_size`.

- **CMS-friendly** (what this template already aims to be):  
  - `allow_url_fopen = On` for updates and plugin installs.  
  - `curl` enabled.  
  - Session settings hardened but still practical.

---

## Key settings and rationale

### Error handling

- `expose_php = Off`: avoids leaking PHP version in HTTP headers.  
- `display_errors = Off`, `display_startup_errors = Off`: do not leak stack traces or file paths to users.  
- `log_errors = On`: errors are logged instead.

### Dangerous functions

- `disable_functions = exec,passthru,shell_exec,system,proc_open,popen`  
  - **Why**: reduce impact of RCE vulnerabilities by blocking OS command execution.  
  - **Impact**: applications relying on these functions will break; remove specific functions only if you know the implications.

### File system restrictions

- `open_basedir` is used to restrict PHP to a limited set of directories (webroot, sessions, temp).  
- You must adapt this to your directory layout (see comments in `php.ini.hardened`).

### Sessions and cookies

- `session.use_strict_mode = 1`: prevents session fixation by refusing uninitialized session IDs.  
- `session.use_only_cookies = 1`, `session.cookie_httponly = 1`: avoid session ID in URLs and protect against basic XSS-based theft.  
- `session.cookie_secure = 1` (recommended if your site is HTTPS-only).  
- `session.cookie_samesite = "Lax"` by default (can be changed to `Strict` or `None` depending on your use case).

### Upload and input limits

- `upload_max_filesize`, `post_max_size` and `max_input_vars` are set to reasonable defaults for CMS-style websites; you can shrink or increase them depending on your needs.

### OPcache

- OPcache is enabled with settings suitable for medium-sized multi-tenant environments.  
- You may want to tune memory and number of cached files if you have a very large number of PHP files.

---

## Author

Maintained by [Prosdo.com](https://prosdo.com) and Farabola Juan (Prosdo.com CEO).

If you are using this template in production or in your hosting stack, feel free to open an issue or share feedback.

---

## License

MIT License.
