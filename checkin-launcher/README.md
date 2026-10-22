# Rock Check-in Launcher

A one-tap front end for Rock RMS check-in. Volunteers see a page of large buttons ("Normal," "Volunteer Only," and so on), and each button opens check-in fully configured. Nobody picks a kiosk, configuration, areas, or theme.

It's a single HTML block with Lava. There's no plugin, no external app, and nothing to install on your server.

**Nextgen check-in only.** Classic (WebForms) check-in is removed in Rock v21, planned for the first half of 2027 (see Spark's [Feature Retirement](https://community.rockrms.com/feature-retirement) page). If you're still on Classic, consider this one more reason to move.

> Tested on Rock v18+ (Nextgen). Part of [rock-checkin-nuggets](../README.md), from the RX 2026 session "Check-In Nuggets: Tips from the Trenches."

## How it works

Nextgen check-in reads its setup from the URL. Its setup page accepts these query string parameters:

| Parameter | What it sets | Required? |
|---|---|---|
| `CheckinConfigId` | Check-in configuration | Yes, unless `KioskId` and `GroupTypeIds` are both passed |
| `GroupTypeIds` | Check-in areas, comma-separated | Yes |
| `KioskId` | Kiosk Device Id | No, if Rock can identify the kiosk itself (see below) |
| `Theme` | Check-in theme | Optional |

Pass a complete set and check-in skips the setup screen. The launcher builds one of these URLs for each button, using values you manage in a Defined Type. To add or change a button, edit the Defined Type; you never edit the page or touch an iPad.

## Setup

### 1. Create the Defined Type

**Admin Tools > Settings > General > Defined Types**, add a new one, for example "Check-in Launcher Options."

Note its Id from the URL (`.../DefinedType/227` means the Id is `227`). In recent Rock versions you may need to open the three-dot menu and choose **Audit** to see the Id. You'll need it in step 3.

Add these attributes to the Defined Type:

**Check-in Areas** (required)
- Key: `CheckInAreas`
- Field Type: Multi-Select
- Required: Yes
- Repeat Columns: 4
- Values: paste this query so the list maintains itself as you add areas:

```sql
SELECT DISTINCT gt.Id AS [Value], gt.Name AS [Text]
FROM GroupType t
JOIN DefinedValue dv ON dv.Id = t.GroupTypePurposeValueId AND dv.Value = 'Check-in Template'
JOIN GroupTypeAssociation a1 ON a1.GroupTypeId = t.Id
LEFT JOIN GroupTypeAssociation a2 ON a2.GroupTypeId = a1.ChildGroupTypeId AND a2.ChildGroupTypeId <> a2.GroupTypeId
LEFT JOIN GroupTypeAssociation a3 ON a3.GroupTypeId = a2.ChildGroupTypeId AND a3.ChildGroupTypeId <> a3.GroupTypeId
CROSS APPLY (VALUES (a1.ChildGroupTypeId), (a2.ChildGroupTypeId), (a3.ChildGroupTypeId)) v(Id)
JOIN GroupType gt ON gt.Id = v.Id
WHERE EXISTS (SELECT 1 FROM [Group] g WHERE g.GroupTypeId = gt.Id AND g.IsActive = 1)
ORDER BY [Text]
```

This lists every check-in area that directly contains active groups, up to three levels deep, and leaves out parent areas that only hold other areas.

**Check-in Configuration** (optional)
- Key: `CheckinConfigId`
- Field Type: Single-Select
- Required: No
- Values:

```sql
SELECT t.Id AS [Value], t.Name AS [Text]
FROM GroupType t
JOIN DefinedValue dv ON dv.Id = t.GroupTypePurposeValueId AND dv.Value = 'Check-in Template'
ORDER BY [Text]
```

Leave it blank on a button to use the default configuration from the settings block. Many organizations use one configuration for every kind of check-in, which keeps kiosk behavior consistent and makes troubleshooting easier. If that's you, leave this blank everywhere.

**Theme** (optional)
- Key: `Theme`
- Field Type: Single-Select
- Required: No
- Values (Nextgen check-in themes):

```sql
SELECT th.Name AS [Value], th.Name AS [Text]
FROM Theme th
JOIN DefinedValue dv ON dv.Id = th.PurposeValueId
WHERE dv.Value = 'Check-in' AND th.IsActive = 1
ORDER BY [Text]
```

Leave it blank to use check-in's default theme.

### 2. Add the buttons

Add one Defined Value per button. The **Value** is the button's label. Check the areas it should include, and optionally set a configuration or theme. Only active values appear, in the Defined Type's sort order.

### 3. Create the launcher page

1. **Admin Tools > Settings > CMS > Pages**: under your check-in site, add a page (for example "Check-in Start") using the **Checkin** layout, with a route such as `nextgen-checkin/start`.
2. Add an **HTML Content** block to the page.
3. Paste in the contents of `checkin-launcher.lava`.
4. Fill in the **SETTINGS** block at the top:

| Setting | What to enter |
|---|---|
| `launcherDefinedTypeId` | The Defined Type Id from step 1 |
| `defaultConfigId` | Id of the configuration used when a button doesn't set one |
| `pageHeading` | Text above the buttons |
| `revealTaps`, `revealSeconds` | Quick taps in the invisible top-right corner that show each button's areas and URL (default 5), and how long they stay visible (default 30 seconds) |
| `pageBackgroundColor`, `headingColor`, `buttonColor`, `buttonTextColor` | Optional colors for the launcher page. Blank uses the theme of the site the page belongs to (Admin Tools > Settings > CMS > Sites). This is separate from the per-button check-in Theme, which styles the check-in screens after a button is tapped |

If the page shows a yellow warning instead of buttons, the Defined Type Id is wrong or the Defined Type has no active values.

### 4. Point your kiosks at the launcher

Set each kiosk's check-in URL (in the Rock check-in app, or the browser's start page) to the launcher page, for example `https://yourrock.org/nextgen-checkin/start`.

## Identifying the kiosk: two patterns

**One Device per campus** (most common, especially with client printing)
Use the plain launcher URL. Without a `KioskId`, Rock finds the kiosk's Device record by IP address: it matches the IP it sees against each check-in kiosk Device's **IP Address / Hostname** field. The check-in setup block can also match by name if **Enable Kiosk Match By Name** is turned on: Rock does a reverse DNS lookup on the IP and matches the hostname against the Device's hostname field or its name.

If Rock is hosted in the cloud, it sees each campus's public IP, so one Device per campus with that public IP works well.

**One Device per physical kiosk** (server printing, or kiosks that need different locations)
Give each kiosk its own launcher URL with its Device Id: `.../nextgen-checkin/start?KioskId=12`. The launcher adds that `KioskId` to every button. This works no matter how your network or hosting is set up.

Notes for IP matching:
- Rock has to see the right IP. Check for NAT, proxies, or a CDN between the kiosks and Rock.
- Use DHCP reservations so kiosk IPs don't change.
- Name matching needs a reverse DNS (PTR) record. iPads usually don't register one.

## Troubleshooting

- **Yellow warning instead of buttons:** check `launcherDefinedTypeId`, and that the Defined Type has active values.
- **Check-in shows the setup screen:** the URL is missing something. To see each button's URL, quickly tap the invisible top-right corner of the launcher (5 taps by default); the URL appears under each button's areas. You need `CheckinConfigId` and `GroupTypeIds`, and either `KioskId` or an IP/name match to a check-in kiosk Device.
- **"Kiosk Closed for Today":** check-in launched correctly, but no schedules are active for those areas right now.
