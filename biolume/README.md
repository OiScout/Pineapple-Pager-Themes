# Biolume

A bioluminescent theme for the Hak5 WiFi Pineapple Pager.

**Author:** OiScout
**Developed against:** firmware 1.1.0-Stable (`theme_framework_version` 0.7)
**Status:** stage 2 — palette, color logic, and artwork complete; awaiting hardware walkthrough

## The idea

Deep-sea bioluminescence. Jade and mint are the signal; violet is the water around them.
Most cyberpunk themes make purple the loud one and green the accent — Biolume inverts that,
so recon sweeps, payload runs, and PineAP state read like a readout from something alive.

The color logic is deliberate and consistent across all 127 screens:

| Role | Palette name | Value |
|---|---|---|
| Resting text, list items, labels | `purple` | `188, 156, 247` |
| Selection, focus, titles, counts | `light_green` | `140, 255, 190` |
| Enabled / on / active state | `green` | `46, 230, 140` |
| Icon and glyph tint | `magenta` | `200, 110, 215` |
| Timestamps, de-emphasis | `gray` | `120, 135, 128` |
| Errors | `red` | `255, 95, 95` |
| Dialog body text | `soft_white` | `226, 242, 235` |
| Ground | `black` | `6, 14, 12` |

All 30 palette names are defined — none removed, none invented.

## Legibility

Every color used for body text was checked against the theme's own `black`:

- resting text (`purple`) — 8.6:1
- selection (`light_green`) — 16.0:1
- active state (`green`) — 11.9:1
- dialog text (`soft_white`) — 16.9:1

For reference, `wargames` resting text sits at 12.2:1. Biolume's violet is dimmer by design —
it is the water, not the signal — but it stays above the 7:1 mark so dense recon and payload
lists remain readable outdoors. If it proves too dim in direct sun, raise `purple` toward
`198, 172, 250` (9.9:1) without touching a single component file.

## Artwork

The ground is one procedurally drawn plate: a violet wash from above, a jade bloom rising from
below, mycelial filaments, and drifting spores. Twenty-six screens share that single file.

Screen titles used to be painted into each background, which is why `wargames` needed 42
full-screen plates. Biolume moves them into text layers instead — the convention `wargames`
already uses on its Pager Portal screens (`"x": 5`, `text_size: "large"`) — so one ground
serves every submenu, dashboard, and list.

| | `wargames` | Biolume |
|---|---|---|
| Asset files | 163 | 138 |
| Total asset weight | 772 KB | **240 KB** |
| Full-screen plates | 42 | 26 |
| Plate weight | 704 KB | 119 KB |

Plates that still carry their own art are the ones whose screens draw no text of their own:
the boot animation, the two battery alerts, the overheat and button-lock screens, the three
update screens, and the framed payload log. The dashboard has an original Biolume mark — a
spore bloom — in place of the Hak5 pineapple.

Everything was quantized with `pngquant --nofs` at 16 colors; the glows show no banding there,
though they band visibly at 12. The three QR codes are the shipped originals, re-encoded at 4
colors — smaller and, if anything, crisper.

## Known limitations

- **Not hardware-tested past the palette.** Stage 1 (color only) was confirmed on device. The
  artwork and the title text layers have not been. The titles are the thing to check first: if
  one sits wrong, it is an `x`/`y` edit in that screen's background layer, nothing more.
- **Some screens lost distinctive art.** `wargames` gave alerts, recon payloads, and the power
  menu their own radar plates. Those now use the shared ground. That is the trade the byte
  budget bought; if one of them deserves its own plate back, it costs about 6 KB.
- **Firmware 1.0.8 / 1.0.9 will warn.** This is a framework 0.7 theme. On older firmware the
  Pager reports a version mismatch and substitutes the seven Pager Portal screens from the
  default theme; everything else still renders. No separate 0.6 build is planned.
- **The validator reports three unreferenced upgrade assets.** They are referenced, through
  `checking_image_path` / `downloading_image_path` / `validating_image_path` in
  `check_for_updates.json`, which the validator does not scan. `wargames` reports the same.

## Fixes carried over `wargames`

Two copy-pasted `screen_name` values still present in `wargames` at commit `ffefa63` are
corrected here — the same class of bug Hak5 fixed elsewhere in the 1.1.0 pass:

- `components/dialogs/edit_number_dialog.json` said `edit_string_dialog`
- `components/pineap/submenu_pineap_ssidpool.json` said `pineap_evilwpa_submenu`

## Install

```bash
scp -r biolume root@172.16.52.1:/root/themes/
```

Then **Settings → General → Theme → Biolume**. `/mmc/root/themes/` works too if you are
short on internal storage.

## Credit

Layout, component tree, and the entire glyph set are inherited from **`wargames`**, the
default theme shipped in Pager ROM by Hak5, at commit `ffefa63`. Biolume is a reskin of that
work, not an original layout.
