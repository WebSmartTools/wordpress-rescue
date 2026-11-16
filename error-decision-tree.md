# WordPress Error Decision Tree

Start at the symptom, follow the tree to the log line, and only then fix. Each entry has the same four parts: what the log line looks like, the most likely cause, the fix, and the rollback. The sample logs in `examples/` are written by hand with fictional plugin and theme names; no real site log is used anywhere in this skill.

## 1. White screen of death

- **What the log line looks like:** `[16-Nov-2026 06:00:00 UTC] PHP Fatal error: Call to undefined function example_function() in /var/www/html/wp-content/plugins/example-plugin/includes/handler.php on line 42`
- **Most likely cause:** A fatal error in a plugin or theme. The file path names the owner.
- **Fix:** Deactivate the plugin named in the path (WP-CLI: `wp plugin deactivate <slug>`, or rename its folder to `<slug>.off`). If the path is in a theme, switch to a default theme. Load the site to confirm it works, then update or replace the broken plugin or theme.
- **Rollback:** Reactivate the plugin (`wp plugin activate <slug>`) or rename the folder back, or switch back to the original theme. The site returns to the broken state, and you try the next fix.

## 2. Error Establishing a Database Connection

- **What the log line looks like:** `[16-Nov-2026 06:00:00 UTC] WordPress database error Access denied for user 'example_user'@'localhost' (using password: YES) for query SELECT * FROM wp_posts` or `Unknown database 'example_db'`, or `Can't connect to MySQL server on 'localhost'`.
- **Most likely cause:** Wrong credentials in `wp-config.php`, the database user lost its permissions, the database was dropped or renamed, or the MySQL server is down.
- **Fix:** Read the exact MySQL error. Access denied means the username, password, or host in `wp-config.php` is wrong, or the user has no grant on the database. Unknown database means the `DB_NAME` value is wrong or the database is gone. Can't connect means MySQL is not running; restart it from the hosting panel. Correct the one wrong value, then load the site.
- **Rollback:** Restore the previous `wp-config.php` from the backup taken before the fix. If you changed database user grants, restore the previous grant.

## 3. HTTP 500

- **What the log line looks like:** A PHP Fatal error line as in error 1, or nothing at all in `debug.log` and a matching entry in the server error log.
- **Most likely cause:** A fatal error (read the log), a memory limit hit during a heavy page, or a server rule blocking the request.
- **Fix:** Read `debug.log` first. If it names a file, follow error 1. If it is empty, check the server error log for the same minute; a 500 with no PHP error is usually a server limit or a corrupt `.htaccess` rule (see error 7).
- **Rollback:** Undo the one change you made, then re-read the log. If you raised a limit, put it back.

## 4. Redirect loops

- **What the log line looks like:** Usually nothing. `debug.log` stays empty because WordPress never gets far enough to log, or the loop happens before WordPress loads.
- **Most likely cause:** `siteurl` and `home` disagree (one is `http`, the other `https`, or one has `www` and the other does not), a redirect plugin rule that points at itself, or a bad rewrite rule in `.htaccess`.
- **Fix:** Check `siteurl` and `home` in the database (`wp_options` table) and make them match the real address, including the `https` and `www` parts. If they match, rename `.htaccess` to `.htaccess.bak` and load the site; if the loop stops, the rule was in `.htaccess`. Then regenerate permalinks from Settings, Permalinks, Save.
- **Rollback:** Restore the old `.htaccess` from the `.bak` file, or set the two URLs back to their previous values from the backup.

## 5. Plugin conflicts

- **What the log line looks like:** A fatal error in one plugin file, or no error at all and the site breaks only when two plugins are active together.
- **Most likely cause:** Two plugins define the same function, hook into the same filter in opposite ways, or load two versions of the same library.
- **Fix:** Deactivate all plugins, confirm the site works, then reactivate one at a time and load the site after each. The plugin that breaks it is the suspect. If it only breaks with two active, the pair is the conflict. Leave the suspect off, then update it, replace it, or report the exact log line to its author.
- **Rollback:** Reactivate the plugins in their original order from the backup list. If you updated a plugin, restore its folder from the backup.

## 6. Theme conflicts

- **What the log line looks like:** A fatal error in `wp-content/themes/<slug>/`, often in `functions.php`, after a theme update or a PHP version change.
- **Most likely cause:** The theme calls a function that was removed in the new PHP version, or a theme file was partly updated.
- **Fix:** Switch to a default theme (Twenty Twenty-Four or Twenty Twenty-Five) and confirm the site works. Then update the theme, or keep the default theme while the theme author fixes it. If the admin is unreachable, switch with WP-CLI (`wp theme activate twentytwentyfour`) or rename the theme folder to `<slug>.off` so WordPress falls back.
- **Rollback:** Switch back to the original theme, or rename the folder back.

## 7. `.htaccess` corruption

- **What the log line looks like:** Usually nothing in `debug.log`. The server error log may show a rewrite error, or the site returns 500 on every page including pages that worked before.
- **Most likely cause:** A bad edit to `.htaccess`, a plugin that wrote a broken rule, or a rule copied from another server.
- **Fix:** Rename `.htaccess` to `.htaccess.bak` and load the site. If it works, the file was the cause. Go to Settings, Permalinks, and Save to let WordPress write a fresh default file. Re-add custom rules one at a time, testing after each.
- **Rollback:** Delete the new file and rename `.htaccess.bak` back to `.htaccess`.

## 8. PHP version and memory limits

- **What the log line looks like:** `[16-Nov-2026 06:00:00 UTC] PHP Fatal error: Allowed memory size of 268435456 bytes exhausted (tried to allocate 123456 bytes) in /var/www/html/wp-content/themes/example-theme/functions.php on line 12` or `PHP Fatal error: Call to undefined function get_some_block() in /var/www/html/wp-content/plugins/example-plugin/file.php on line 30` after a PHP upgrade, or `PHP Deprecated: Function create_function() is deprecated`.
- **Most likely cause:** Memory exhaustion means one plugin or the theme uses too much memory on a page. An undefined function after a PHP upgrade means a plugin or theme uses a function removed in the new PHP version.
- **Fix:** For memory, find the file in the log line and deactivate that plugin or theme first; only raise `WP_MEMORY_LIMIT` in `wp-config.php` if the log shows the site itself needs more (for example a large import). For a PHP version error, update the plugin or theme to a version that supports the new PHP, or pin the site back to the previous PHP version from the hosting panel while you update.
- **Rollback:** Restore the previous `WP_MEMORY_LIMIT` value, or switch the PHP version back, from the backup note taken before the change.

## 9. File permissions

- **What the log line looks like:** `[16-Nov-2026 06:00:00 UTC] PHP Warning: move_uploaded_file(/var/www/html/wp-content/uploads/2026/11/image.jpg): failed to open stream: Permission denied in /var/www/html/wp-admin/includes/file.php on line 1200` or an update failure saying the update could not be installed.
- **Most likely cause:** `wp-content/uploads` or a plugin folder is owned by the wrong user, or a folder has 600 instead of 755 permissions, so the web server cannot write to it.
- **Fix:** Set folders to 755 and files to 644, and make sure `wp-content` is owned by the web server user. Fix ownership first; permissions second. Never set anything to 777.
- **Rollback:** Restore the previous owner and permission values from the backup note. If an update failed halfway, restore the plugin folder from the backup before retrying.
- **Note:** `diagnose.py` cannot check this. It reads a log file, not the site, so a permissions check needs filesystem access to the live site. This entry is the check.

## 10. No log at all (`debug.log` empty or missing)

- **What the log line looks like:** `debug.log` does not exist, or it exists and is empty, while the site shows a white screen or a 500 error.
- **Most likely cause:** Logging is off (`WP_DEBUG_LOG` is false or was never set), the failure happens before WordPress loads `wp-config.php`, or the server hides PHP errors (display off, logging off, and no error log set).
- **Fix:** Check the server error log first, for the same minute as the failure; it records errors WordPress never sees. Then force logging on: add the force-logging snippet from the premium pack to `wp-config.php` (it writes PHP errors to `wp-content/debug-custom.log` regardless of server defaults), or set `WP_DEBUG` and `WP_DEBUG_LOG` to `true` and `WP_DEBUG_DISPLAY` to `false` by hand. Reproduce the error once by loading the broken page. Then read the new log, or run `diagnose.py` on it in the premium pack, and follow the error it names.
- **Rollback:** Remove the snippet (or set `WP_DEBUG` back to `false`) once the fix is verified. Keep the log file for reference, or delete it. Leaving `WP_DEBUG_LOG` on is safe as long as `WP_DEBUG_DISPLAY` stays `false`.
