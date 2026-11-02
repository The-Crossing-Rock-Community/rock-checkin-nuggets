# Rock Check-in Nuggets

Tips, Lava, and ZPL from the RX 2026 session **"Check-In Nuggets: Tips from the Trenches"** by Jim Michael (The Crossing, St. Louis, MO).

Everything here is for Rock RMS **Nextgen** check-in. Classic (WebForms) check-in is removed in Rock v21, planned for the first half of 2027 (see Spark's [Feature Retirement](https://community.rockrms.com/feature-retirement) page).

## The slides

The full deck from the session: [`RX2026_CheckIn_Nuggets_public.pptx`](RX2026_CheckIn_Nuggets_public.pptx). GitHub can't preview PowerPoint files, so click the file and use the download button (opens in PowerPoint, Keynote, or Google Slides).

It covers five nuggets. Two come with code in this repo; the other three are summarized under [The rest of the talk](#the-rest-of-the-talk).

## What's here

| Folder | What it is |
|---|---|
| [`checkin-launcher`](checkin-launcher/) | Nugget 1. A one-tap launcher page: each button opens check-in fully configured, so volunteers never pick a kiosk, configuration, or areas. One HTML block plus a Defined Type. |
| [`printer-config-label`](printer-config-label/) | Nugget 3. A one-time ZPL label that configures a Zebra printer, including the backfeed setting that stops cutter jams. |

Each folder has its own README with step-by-step setup. You'll need Rock admin rights to follow them.

A few terms you'll run into:

- **Lava:** Rock's template language. You paste it into a block on a Rock page; nothing gets installed on the server.
- **ZPL:** the command language Zebra label printers speak. Rock check-in labels are ZPL under the hood.
- **Defined Type:** a list you manage in Rock (Admin Tools > Settings > General > Defined Types). The launcher uses one as its list of buttons.

## The rest of the talk

Some nuggets from the session don't need code:

- **Nugget 2, tokenized kiosk logins:** the idea is to put a long-lived login token for a kiosk-only account into a link (a Rock ShortLink), so kiosks log themselves in and never prompt for a password. We used this for years and stopped. After iPadOS 26, the check-in app started asking for a login again at random, and the page that hands out the token has to be unauthenticated, so anyone with the URL gets in. Rock's normal login with **Keep me logged in** checked stays signed in as long as a kiosk is used within 30 days, which is simpler and safer.
- **Nugget 4, Classic vs. Nextgen labels:** Lava on Nextgen labels is close to Classic, but not identical. Nextgen has five label types (Family, Person, Attendance, Checkout, Person Location), and the type you pick decides which data your Lava can reach. Many things that needed Lava on Classic are built into Nextgen's label designer. Inventory your custom merge fields and test every custom label before you migrate.
- **Nugget 5, use an MDM:** if you manage more than a few kiosks, a mobile device management tool (Mosyle, Jamf, Intune, and others) pays for itself: push the check-in app, lock devices into Single App Mode, and control when OS updates land. See the community recipe [How to set up the iPad Rock check-in app via MDM](https://community.rockrms.com/recipes/436/how-to-setup-the-ipad-rock-check-in-app-via-mdm).

## Disclaimer

Shared as-is, from one church's experience. Test in a non-production Rock instance first.
