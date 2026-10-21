# Printer Config Label

A one-time ZPL label that configures a Zebra check-in printer. Print it once to each printer (new, or after a factory reset) instead of using a driver, a vendor utility, or the printer's menus.

> Part of [rock-checkin-nuggets](../README.md), from the RX 2026 session "Check-In Nuggets: Tips from the Trenches." Tested on a Zebra ZD621 with Rock Nextgen check-in.

## Why you want this

**Cutter jams.** Zebra's default backfeed setting (`~JSN`) leaves the next label parked on the rubber platen roller between prints. Overnight it takes a curl, and the first label of the morning jams in the cutter. Backfeed-before (`~JSB`) parks the label inside the cutter instead, where it can't curl.

**Nextgen doesn't set it for you.** Classic check-in labels sent `~JSN` with every print. Nextgen designed labels send no backfeed command at all, so the printer uses whatever setting it already has. A brand-new printer defaults to `~JSN`.

**Settings don't stick without `^JUS`.** Any printer setting sent without `^JUS` lasts only until the next power cycle. `^JUS` saves the settings to the printer's memory. Don't put it in your everyday labels, though: saving on every print wears out the printer's memory, which is why Rock's Classic label editor strips it.

## What it sets

| Command | Setting | Change it if... |
|---|---|---|
| `~JSB` | Backfeed before printing | Remove it for printers without a cutter |
| `^MMC` | Print mode: cutter | No cutter: use `^MMT` (tear-off) |
| `^MNW` | Media tracking: gap/die-cut labels | Black-mark stock: use `^MNM` |
| `^MTD` | Media type: direct thermal | You use ribbon (thermal transfer): `^MTT` |
| `^PR6,6` | Print and feed speed, inches per second | Preference |
| `~SD10` | Darkness, 0-30 | Prints are too light or too dark for your stock |
| `^JUS` | Save settings to memory | Never remove it |

The `^FX` lines are ZPL comments; the printer ignores them. Don't put `^` or `~` inside a comment, because the printer reads those as the start of a new command.

### Optional: front-panel passcode

For printers with a front panel (ZD500, ZD6xx, and others), add a line before `^JUS`:

```
^KP1234
```

using your own 4-digit code. Without `^JUS` after it, the passcode reverts to the default `1234` at the next power cycle.

Keep in mind that a ZPL label's contents are visible to anyone who can edit check-in labels in Rock, and Nextgen's label preview sends the label to Labelary (api.labelary.com) to render it. Leave the passcode out if that matters to you.

## Printing it from Rock (Nextgen)

1. **Admin Tools > Settings > Check-in > Check-in Labels**: add a new label.
2. Set **Label Format** to **ZPL**. Any label type works.
3. Paste in `printer-config.zpl`, adjusting the settings above first. Save.
4. Don't attach this label to any check-in area or group, so it never prints during check-in.
5. Open the label's **Preview**, pick the printer, and click **Print**. Repeat for each printer.

Keep the "Printer configured" line. Nextgen's preview renders the label through Labelary before it enables Print, and a label with nothing printable won't render. The printout also confirms the settings were applied.

If the printer's Device record has **Has Cutter** set, Rock adds `^MMC` just before the final `^XZ` when it prints. That's harmless here.

## Verifying it worked

1. Print a check-in label and open the lid. The next label should be sitting in the cutter, not on the platen.
2. Power-cycle the printer, print again, and check again. If it's back on the platen, the `^JUS` didn't take.
