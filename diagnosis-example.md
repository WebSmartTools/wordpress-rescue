# Worked Example: A Fatal Error in a Booking Form Plugin

This is a diagnosis of the sample log `sample-debug-log-fatal-plugin.log` only. It is a hand-written sample with a fictional plugin name. It is not a diagnosis of any real site.

## The log

The last line is the one that stopped the page:

```
[16-Nov-2026 06:12:04 UTC] PHP Fatal error: Call to undefined function sample_booking_render() in /var/www/html/wp-content/plugins/sample-booking-form/includes/booking.php on line 87
```

The two lines before it are a Notice and a Warning in the same plugin. They are noise; the fatal error is the cause.

## The diagnosis

- **Error class:** fatal-plugin
- **Failing plugin:** sample-booking-form (the path is `wp-content/plugins/sample-booking-form/`)
- **File and line:** `includes/booking.php`, line 87
- **Cause:** the code calls `sample_booking_render()`, a function that does not exist. The plugin's main file did not load it, or the plugin was only partly updated, so the function was never defined.

## The fix

1. Back up first: copy the `sample-booking-form` plugin folder and export the database.
2. Deactivate the plugin: `wp plugin deactivate sample-booking-form`, or rename its folder to `sample-booking-form.off`.
3. Load the site and confirm the white screen is gone.
4. Update the plugin fully from a fresh copy, or reinstall it. Do not edit `booking.php` to remove the call; that hides the symptom and leaves the plugin broken.

## The rollback

Reactivate the plugin (`wp plugin activate sample-booking-form`) or rename the folder back, or restore the plugin folder from the backup. The site returns to the white screen it had, and you try the next fix instead of leaving a half-fixed plugin in place.

## What the script reports

Running `python3 diagnose.py sample-debug-log-fatal-plugin.log` reports:

```
sample-debug-log-fatal-plugin.log:
  ERROR: fatal error in plugin "sample-booking-form" (/var/www/html/wp-content/plugins/sample-booking-form/includes/booking.php on line 87): Call to undefined function sample_booking_render()
  ERROR: fix: deactivate the plugin, verify the site loads, then update or reinstall it from a fresh copy
  ERROR: rollback: reactivate the plugin or restore its folder from the backup taken before the fix
  warning: notice: Undefined index: sample_setting (/var/www/html/wp-content/plugins/sample-booking-form/includes/settings.php on line 18)
  warning: Invalid argument supplied for foreach() (/var/www/html/wp-content/plugins/sample-booking-form/includes/list.php on line 54)
  FAILED: 3 error(s), 2 warning(s)
```

Exit code is 1, because a fatal error class was found.
