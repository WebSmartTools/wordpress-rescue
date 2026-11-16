# Changelog

## v1.0.1 - 2026-11-16

- Premium: `rescue.php`, a standalone run-once emergency file. Upload it to the site root by FTP, visit it in a browser, confirm, and it disables all plugins and switches the theme to a default theme, so a white-screen site loads again and wp-admin is reachable. It shows a confirmation page first, writes a backup of the old plugin and theme values, and is marked throughout to be deleted immediately after use. It never leaves the site on its own; the delete step is in the file header, on the confirmation page, and on the result page.
- Premium: `force-logging-snippet.php`, a drop-in snippet for `wp-config.php` that forces PHP errors into `wp-content/debug-custom.log` regardless of server defaults, with full instructions in the file header.
- Skill and decision tree: a "no log at all" path. When `debug.log` is empty or missing, check the server error log first, force logging on with the snippet, reproduce the error once, then re-read the log or re-run `diagnose.py`. The decision tree gains entry 10 for this.
- `diagnose.py`: a new `--config` mode (`python3 diagnose.py --config wp-config.php [functions.php ...]`) that scans `wp-config.php` and a theme `functions.php` for leading or trailing whitespace before `<?php` or after `?>`, including a UTF-8 BOM, and reports each problem as an error. Log parsing is unchanged.
- Deliberately not added: a file-permissions check in `diagnose.py`. It reads a log file, not the site, so a permissions check needs filesystem access to the live site. The 755/644 check lives in the decision tree (entry 9), which now says so in a note, and the `diagnose.py` docstring says the same.

## v1.0.0 - 2026-11-16

First release.

- Free skill: the wordpress-rescue skill for Claude Code, diagnosis-first from the actual debug.log, with the safe rollback procedure, the plugin and theme isolation sequences, a worked diagnosis example, and the full error decision tree for nine errors (`references/error-decision-tree.md`). Includes install steps for Claude Code, Codex, Cursor, and Claude.ai, and four hand-written sample debug.log files.
- Premium pack: the free skill plus the diagnosis script (`diagnose.py`), the rollback checklist (`rollback-checklist.md`), and the error decision tree.
- Price: $15, one-time payment, single-user licence.
