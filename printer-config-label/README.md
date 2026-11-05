# Printer Config Label

A one-time ZPL label that configures a Zebra check-in printer. Print it once to each printer (new, or after a factory reset) instead of using a driver, a vendor utility, or the printer's menus.

> Part of [rock-checkin-nuggets](../README.md), from the RX 2026 session "Check-In Nuggets: Tips from the Trenches." Tested on a Zebra ZD621 with Rock Nextgen check-in.

## Why you want this

**Cutter jams.** Zebra's default backfeed setting (`~JSN`) leaves the next label parked on the rubber platen roller between prints. Overnight it takes a curl, and the first label of the morning jams in the cutter. Backfeed-before (`~JSB`) parks the label inside the cutter instead, where it can't curl.

**Nextgen doesn't set it for you.** Rock's stock Classic labels include `~JSN`, so it went out with every print (the Classic fix was changing that one `N` to a `B`). Nextgen designed labels send no backfeed command at all, so the printer uses whatever setting it already has. A brand-new printer defaults to `~JSN`.

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
| `^KP8888` | Front-panel passcode (optional, see below) | Always: `8888` is a placeholder |
| `^JUS` | Save settings to memory | Never remove it |

The `^FX` lines are ZPL comments; the printer ignores them. Don't put `^` or `~` inside a comment, because the printer reads those as the start of a new command.

### Optional: front-panel passcode

For printers with a front panel (ZD500, ZD6xx, and others), the `^KP` line locks the printer's menus behind a 4-digit code so nobody changes settings at the kiosk. The file ships with `^KP8888` as a placeholder: change `8888` to your own code. It has to stay before `^JUS`, or the passcode reverts to the default `1234` at the next power cycle.

Keep in mind that a ZPL label's contents are visible to anyone who can edit check-in labels in Rock, and Nextgen's label preview sends the label to Labelary (api.labelary.com) to render it. If that matters to you, delete the `^KP` line and the comment above it.

## Printing it from Rock (Nextgen)

1. Open Rock's Nextgen check-in **Labels** page and add a new label. (Its location in the admin menus depends on your Rock version.)
2. Set **Label Format** to **ZPL**. Any label type works for Preview printing. Choose **Family** if you might print it through check-in instead (see below).
3. Paste in `printer-config.zpl`, adjusting the settings above first. Save.
4. Don't attach this label to any check-in area or group, so it never prints during check-in.
5. Open the label's **Preview**, pick the printer, and click **Print**. Repeat for each printer.

Keep the "Printer configured" line. Nextgen's preview renders the label through Labelary before it enables Print, and a label with nothing printable won't render. The printout also confirms the settings were applied.

If the printer's Device record has **Has Cutter** set, Rock adds `^MMC` just before the final `^XZ` when it prints. That's harmless here.

## If Print doesn't work (cloud-hosted Rock)

The Rock server sends a Preview print, not the kiosk. So the server has to reach the printer's IP on port 9100, either directly or through Rock's Cloud Print proxy. Most cloud-hosted Rock servers can't reach printers on a campus network. If you're one of them and don't run Cloud Print, Print won't reach the printer. You'll see an error such as "Timed out waiting for labels to print." or "Unable to print label: ...". You have two other options.

### Option A: print it through check-in

This works with any hosting, because the label prints the same way your real check-in labels do.

1. Make sure the label type is **Family**, so it prints once per check-in rather than once per person.
2. Attach the label to a check-in area you can check into right now: an area with an active schedule, or a test area with an always-open schedule (otherwise you'll get "Kiosk Closed for Today").
3. At each kiosk, check in a test family with one person. The config label prints along with that area's normal labels.
4. **Remove the label from the area as soon as you're done.** If you leave it attached, every check-in reconfigures the printer and runs `^JUS`, which is exactly the memory wear described above.

Repeat at each kiosk that has its own printer.

### Option B: send it from a PC with PowerShell

Any Windows PC that can reach the printer's IP (same network, or a firewall rule allowing TCP 9100) can send the file directly. You don't need a driver or Zebra's utilities. Change the IP and file path, then run:

```powershell
$printer = '10.1.2.3'
$file    = 'C:\path\to\printer-config.zpl'

$client = New-Object System.Net.Sockets.TcpClient($printer, 9100)
$stream = $client.GetStream()
$bytes  = [System.IO.File]::ReadAllBytes($file)
$stream.Write($bytes, 0, $bytes.Length)
$stream.Flush()
$client.Close()
```

Change `$printer` and run it again for each printer. This is the fastest way to do a whole campus. The printer's IP is on its Device record in Rock, or on a printer configuration report (hold the feed button on most Zebras).

## Verifying it worked

1. Print a check-in label and open the lid. The next label should be sitting in the cutter, not on the platen.
2. Power-cycle the printer, print again, and check again. If it's back on the platen, the `^JUS` didn't take.
