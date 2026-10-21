# Rock Check-in Nuggets

Tips, Lava, and ZPL from the RX 2026 session **"Check-In Nuggets: Tips from the Trenches"** by Jim Michael (The Crossing, St. Louis, MO).

Everything here is for Rock RMS **Nextgen** check-in. Classic (WebForms) check-in is removed in Rock v21, planned for the first half of 2027 (see Spark's [Feature Retirement](https://community.rockrms.com/feature-retirement) page).

## What's here

| Folder | What it is |
|---|---|
| [`checkin-launcher`](checkin-launcher/) | A one-tap launcher page: each button opens check-in fully configured, so volunteers never pick a kiosk, configuration, or areas. One HTML block plus a Defined Type. |
| [`printer-config-label`](printer-config-label/) | A one-time ZPL label that configures a Zebra printer, including the backfeed setting that stops cutter jams. |

Each folder has its own README with setup steps.

## The rest of the talk

Some nuggets from the session don't need code:

- **Tokenized kiosk logins:** we used them for years and stopped. After iPadOS 26, the check-in app started asking for a login again at random, and the page that hands out the token has to be unauthenticated, so anyone with the URL gets in. Rock's normal Remember Me login stays signed in as long as a kiosk is used within 30 days.
- **Lava on Nextgen labels:** close to Classic, but not identical. Nextgen has four label types (Family, Person, Attendance, Checkout), each with its own data. Test every custom label before you migrate.
- **Use an MDM:** if you manage more than a few kiosks, an MDM pays for itself: push the check-in app, lock devices into Single App Mode, and control when OS updates land.

## Disclaimer

Shared as-is, from one church's experience. Test in a non-production Rock instance first.
