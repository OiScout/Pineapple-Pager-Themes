# Biolume — working notes

A theme for the Hak5 WiFi Pineapple Pager. Author: OiScout.
Target: firmware **1.1.0-Stable**, `theme_framework_version` **0.7**.
Derived from `wargames` @ `ffefa63` (Hak5, shipped in ROM) — credit stays in the README.

## Colour logic — do not break this

The palette names are a fixed vocabulary of 30. Never add or remove one; components
reference them by name and a missing name leaves an element unstyled.

| Role | Name | Value |
|---|---|---|
| Resting text, list items, labels | `purple` | `188, 156, 247` |
| Selection, focus, titles, counts | `light_green` | `140, 255, 190` |
| Enabled / on / active | `green` | `46, 230, 140` |
| Icon and glyph tint | `magenta` | `200, 110, 215` |
| Timestamps, de-emphasis | `gray` | `120, 135, 128` |
| Errors | `red` | `255, 95, 95` |
| Dialog body text | `soft_white` | `226, 242, 235` |
| Ground | `black` | `6, 14, 12` |

`wargames` used `blue` for resting text and `yellow` for selection; Biolume repointed all
592 of those references to `purple` and `light_green`. Four files are deliberately exempt:
`error_text.json` (red — errors must read as errors), `alert_info_dialog_text.json` and
`confirmation_dialog_text.json` (`soft_white` for dialog legibility), and
`selected_signal.json` (yellow is a rung in the signal-strength ramp, not a selection colour).

`blue`, `light_blue`, `cyan`, `teal`, `orange`, `brown`, `tan`, `olive` and their dark
variants are defined but unreferenced. That is intentional — keep them defined.

## Architecture — one ground, titles as text

Screen titles are **text layers, not painted into the art**. The convention, taken from
`wargames`' own Pager Portal screens:

```json
{ "text": "Settings", "x": 5, "text_size": "large", "text_color_palette": "light_green" }
```

`y` is omitted deliberately — that is what the reference does. Because of this, one plate
(`assets/bio_ground.png`) serves 26 screens. Do not reintroduce per-screen titled plates.

Plates that still carry their own art are the screens that draw no text of their own and
would otherwise render blank: the boot animation, both battery alerts, overheat, button-lock,
the three update screens, and the framed payload log.

## Home dashboard — 3x2 grid

Rebuilt 2026-08-23 from the diamond strip to a full-screen grid. Tiles are 148x72 at
`x = 10 / 165 / 320`, `y = 30 / 110`, in reading order (RECON, PINEAP, PAYLOADS / ALERTS,
SETTINGS, PORTAL). `button_map` is a linear `previous`/`next`, so item order *is* the
traversal order — keep it matching the visual order.

Each tile is three layers plus a label, because `recolor_palette` is a **flat tint** and
would otherwise destroy any fill-vs-stroke distinction:

- `tile_fill.png` — rounded rect, darkness carried in the **alpha**, never recoloured, so
  the ground shows faintly through and it works on any background
- `tile_edge.png` — 1px outline, drawn white, recoloured `purple` at rest
- `tile_edge_selected.png` — same outline with a blurred halo in the alpha, recoloured
  `green`; tinting alpha-encoded glow is how you get a coloured glow out of a flat tint
- the glyph, recoloured `purple` / `light_green`, then the label text

`layers` and `selected_layers` are not additive, so all four are repeated in both.

**Label centring is estimated at 9.25 px/char**, measured off a device photo (the status-bar
clock `"06:45 PM"` starts at its declared `x:400` and ends at 474). If the labels sit off
centre on hardware, that constant is the thing to correct — one `x` per tile.

Retired with this change: `item_bg.png` (diamond), `highlight.png` (yellow ring), and
`biolume_bg_updated.png` (the spore-bloom plate — the grid covers it, so the dashboard now
uses the shared `bio_ground.png`). The bloom is regenerable from the build script.

The bottom-left line uses `$_FIRMWARE_VERSION`. That variable is generic, but it is
**unverified on this screen** — if it renders literally, delete the two bottom layers in
`biolume_dashboard.json`. AP count and PineAP state were deliberately left out: those are
recon- and settings-family variables with no evidence they bind on the dashboard.

The status bar is untouched — wargames' icon set, violet-tinted, no centred title.

## Layer order — convention, not a crash cause

In every reference screen holding both a title and `$_` variable layers, the title sits at
index 1, right after the base image and before any `variable_name` layer. Biolume now
matches that everywhere, so keep inserting titles at index 1 rather than appending.

**This was wrongly blamed for a crash on 2026-08-23 and did not fix it.** Do not treat layer
order as a suspect again without new evidence.

## FIXED — the payloads crash was a dropped `variable` binding

Symptom (2026-08-23): opening Payloads from the home dashboard crashed the firmware, as did
Portal -> Payloads -> Update Payloads. The other five tiles were fine. Stock wargames 1.1.0
was fine.

Cause: rebuilding `biolume_dashboard.json` as a grid meant writing `menu_items` from scratch,
which silently dropped a key that only the PAYLOADS item carries:

```json
{ "id": "PAYLOADS", "target": "payloads_dashboard", "variable": "$_PAYLOAD_CATEGORIES" }
```

**All 52 themes in the Hak5 repo carry that exact binding on their PAYLOADS tile.** It is not
optional. Without it the payload screens have no category context and the firmware faults.

### The rule

When rebuilding a component's `menu_items`, never author them from scratch. Copy the
reference item and change only `x`, `y`, `layers` and `selected_layers`. Every other key —
`id`, `target`, `variable`, `unselectable` — carries behaviour.

Guard to run after any menu rebuild:

```python
ref={i['id']:i for i in json.load(open(REF))['menu_items']}
missing=[(i['id'],k) for i in mine['menu_items'] for k in ref.get(i['id'],{})
         if k not in ('layers','selected_layers','x','y') and k not in i]
```

### How long this took, and why

Four wrong theories before the right one — layer ordering, PNG bit depth, deleted orphan
assets, and templates — because every check was against *validity* and the theme was always
valid. `validate_theme.py` cannot catch a dropped `variable`; nothing references it, no path
breaks, no palette name goes undefined.

What actually found it was a three-rung bisect built from stock wargames (`_bisect/bt1..bt3`
in this repo), each rung adding one layer of Biolume. All three passed, which isolated the
fault to the one file none of them contained. **Reach for the bisect earlier next time** —
static analysis was exhausted long before I stopped doing it.

Also learned the hard way: never patch the live theme while a test on it is outstanding. One
bisect result was destroyed that way, and one earlier "still crashes" report came from a
build where the patch had never been applied at all.

## The bottom status line is gone

`$_FIRMWARE_VERSION` does not resolve on the home dashboard — confirmed on hardware, it
rendered nothing. The two bottom layers were removed. Do not re-add `$_` variables to that
screen without testing them there first; the variable families in the schema are documented
per screen type, and the dashboard binds none of them.

## Untinted art is the recurring bug in this theme

`wargames` draws most of its UI chrome as **baked cyan PNGs with no `recolor_palette`**, so a
palette swap leaves them blue. This has now bitten three times: the status bar, the dashboard
tiles, and the settings toggles and list icons. Each time the fix was the same — add
`recolor_palette`, not repaint the art.

Swept 2026-08-23: 450 layers across 46 files. The map, by role:

- outline/icon glyphs -> `purple` (`menu.png`, `divider/divleft/divright`, `checkbox`, `x`,
  `info`, `warning`, `wizard`, `keyboard`, `flame`, `optiondialog/check|x|button_outline`,
  `payloadlog/scroll_*`, `keyboard/_key-bg|_spacebar|_hex-bg`, `spinner/*`)
- solid fills -> `dark_purple` (`optiondialog/button_bg` at 91% opaque)
- selection boxes -> `light_green` (`payload_dialog_selected_box`)
- toggle ON -> track `purple`, knob `black`, tick `light_green`;
  toggle OFF -> track `dark_purple`, knob `purple`. Bright track = on, dim track = off,
  which is the polarity `wargames` uses.

Two things stay as they are: `disabled_keyboard.png` and the other `disabled_*` glyphs are
meant to be grey, and the six `keyboard_layout_*.png` plates are multi-tone backgrounds that a
flat tint would destroy — those were hue-shifted at the pixel level instead.

**To find the next one:** list every `image_path` layer with no `recolor_palette`, then check
the average hue of each asset. Anything in the 170-235 range with saturation above 25 is
leftover wargames cyan.

## Colour validation — verified clean, and how

Run after any colour edit. Note the **permissive** value pattern: an earlier version of this
check matched values with `"([a-z_]+)"`, which silently skipped anything containing a capital
or a digit — a malformed value would not have been reported at all.

```python
# every *_palette field, whatever its value
for k, v in node.items():
    if k.endswith('_palette') and k != 'color_palette':
        assert isinstance(v, str) and v in palette_names, (file, k, repr(v))
```

State as of 2026-08-23, checked across all 128 components plus `theme.json`:

- 30 palette entries, every channel an int in 0-255
- fields in use: `text_color_palette` (1090), `recolor_palette` (856), `bar_color_palette` (1)
- 17 distinct values, all resolving
- no inline `{r,g,b}` outside the palette except `background_color` and `payload_log.json`'s
  `default_text_color` / `error_text_color`, which are wargames originals carrying an `a` key
- `_unused` block intact, all 58 top-level keys present

## The theme directory must contain nothing but the theme

Found 2026-08-24 while chasing "Invalid color defined by theme". The build process had been
using the live theme directory as scratch space:

- `biolume/_to_delete/` — 948 KB holding **68 stray component JSONs** from five different
  build stages, plus tarballs. Every one of them shipped with `scp -r biolume`.
- `.DS_Store` in `components/` and `assets/` — non-JSON files sitting where anything globbing
  those directories would try to parse them.

Both are now outside the theme, in `../_scratch/`. **Never stage, extract or park files inside
the theme directory.** Anything that is not `theme.json`, `components/*.json`,
`assets/**/*.png` or a README does not belong there, and a partial component tree from an
older build is exactly the kind of thing a full theme scan will choke on.

Check before shipping:

```bash
find biolume -type f ! -name '*.png' ! -name '*.json' ! -name '*.md'   # must be empty
find biolume/components -name '*.json' | wc -l                          # must be 128
```

## Missing colour information — the alpha channel counts

`buttons_locked.png`, `low_battery_alert.png` and `critical_battery_alert.png` are **overlays**.
`wargames` ships them as palette PNGs with a `tRNS` chunk (10-31 transparent entries) so they
dim the screen behind. The generated Biolume versions were fully opaque with no `tRNS` — the
alpha information was simply gone, so they covered the screen instead of compositing over it.

Rebuilt with the drawn content opaque and the ground at 80% as a scrim. When generating a
plate, check whether the reference version carries transparency before writing it flat:

```python
# tRNS present in the reference but absent in yours = lost alpha
```

## Colour audit — what is verified complete

Run all of these after any colour work. The first is the one that matters most: it looks for
colour information that is *missing*, not merely invalid.

1. **Census, per file, against the reference** — count every colour-bearing key
   (`text_color_palette`, `recolor_palette`, `bar_color_palette`, `string_template`,
   `use_template`, `text_color`, `background_color`) and flag any file where Biolume has
   fewer than `wargames`. Currently: **0 shortfalls across all 128 components**.
2. **Permissive reference check** — every key ending `_palette` must hold a string present in
   the palette. Match values with `"([^"]*)"`, never `"([a-z_]+)"`; the narrow pattern
   silently skips malformed values instead of reporting them.
3. **Template maps** — `string_templates` 29, `toggle_templates` 6, `radio_templates` 2,
   `signal_templates` 2, identical to `wargames`, every registered file present.
   (`components/templates/list_page_current.json` is unregistered in `wargames` too — upstream
   orphan, not a defect.)
4. **Asset encoding** — colour type, bit depth, palette size and `tRNS` compared per asset
   against the reference. Sub-8-bit palettes are fine; `wargames` ships 1-, 2- and 4-bit.

## The bottom status line is gone

`$_FIRMWARE_VERSION` does not resolve on the home dashboard — confirmed on hardware, it
rendered nothing. The two bottom layers were removed. Do not re-add `$_` variables to that
screen without testing them there first; the variable families in the schema are documented
per screen type, and the dashboard binds none of them.

## Untinted art is the recurring bug in this theme

`wargames` draws most of its UI chrome as **baked cyan PNGs with no `recolor_palette`**, so a
palette swap leaves them blue. This has now bitten three times: the status bar, the dashboard
tiles, and the settings toggles and list icons. Each time the fix was the same — add
`recolor_palette`, not repaint the art.

Swept 2026-08-23: 450 layers across 46 files. The map, by role:

- outline/icon glyphs -> `purple` (`menu.png`, `divider/divleft/divright`, `checkbox`, `x`,
  `info`, `warning`, `wizard`, `keyboard`, `flame`, `optiondialog/check|x|button_outline`,
  `payloadlog/scroll_*`, `keyboard/_key-bg|_spacebar|_hex-bg`, `spinner/*`)
- solid fills -> `dark_purple` (`optiondialog/button_bg` at 91% opaque)
- selection boxes -> `light_green` (`payload_dialog_selected_box`)
- toggle ON -> track `purple`, knob `black`, tick `light_green`;
  toggle OFF -> track `dark_purple`, knob `purple`. Bright track = on, dim track = off,
  which is the polarity `wargames` uses.

Two things stay as they are: `disabled_keyboard.png` and the other `disabled_*` glyphs are
meant to be grey, and the six `keyboard_layout_*.png` plates are multi-tone backgrounds that a
flat tint would destroy — those were hue-shifted at the pixel level instead.

**To find the next one:** list every `image_path` layer with no `recolor_palette`, then check
the average hue of each asset. Anything in the 170-235 range with saturation above 25 is
leftover wargames cyan.

## Colour validation — verified clean, and how

Run after any colour edit. Note the **permissive** value pattern: an earlier version of this
check matched values with `"([a-z_]+)"`, which silently skipped anything containing a capital
or a digit — a malformed value would not have been reported at all.

```python
# every *_palette field, whatever its value
for k, v in node.items():
    if k.endswith('_palette') and k != 'color_palette':
        assert isinstance(v, str) and v in palette_names, (file, k, repr(v))
```

State as of 2026-08-23, checked across all 128 components plus `theme.json`:

- 30 palette entries, every channel an int in 0-255
- fields in use: `text_color_palette` (1090), `recolor_palette` (856), `bar_color_palette` (1)
- 17 distinct values, all resolving
- no inline `{r,g,b}` outside the palette except `background_color` and `payload_log.json`'s
  `default_text_color` / `error_text_color`, which are wargames originals carrying an `a` key
- `_unused` block intact, all 58 top-level keys present

## OPEN — "Invalid color defined by theme" on cancelling the theme picker

Reported 2026-08-23: Settings -> General -> Theme, then back out **without** selecting.

Biolume itself validates clean (above), so treat this as unattributed. Two things make it
likely **not** caused by the recent tint pass:

1. Every previous visit to that menu ended in *selecting* a theme. The cancel path may never
   have been exercised before, so the error could be long-standing.
2. The picker enumerates every theme in `/root/themes`, which currently also holds the
   `bt1` / `bt2` / `bt3` bisect builds. The offending palette may belong to another theme.

Tests, cheapest first: reproduce under stock wargames (exonerates Biolume entirely); then
remove bt1-bt3 from `/root/themes` and retry.

## Only use `recolor_palette` where the reference uses it

The single most likely cause of "Invalid color defined by theme", found 2026-08-24.

The tint pass added 450 `recolor_palette` keys. **250 of them sat in structural contexts
`wargames` never uses.** Normalising paths (item and state names wildcarded) gave seven
container kinds present in Biolume and absent from the reference:

```
.background.layers[]                          19 screens (settings + pineap submenus)
.save_cancel_menu.pages[].menu_items[].layers[]   5 option dialogs  <-- the theme picker
.*.rows[][].selected_layers[]                  4 keyboards
.menu_items[].animation[]                      spinner.json
.scroll_up_indicator[] / .scroll_down_indicator[]  payload_log.json
.template.background.layers[]                  submenu_pineap_clients.json
```

`wargames` never tints inside `animation`, `scroll_*_indicator`, keyboard `rows`, or a
screen's `background.layers[]` at all. Firmware that only implements recolour for menu-item
layers would hit an unhandled colour instruction on a **full theme load** — which is exactly
what cancelling out of the theme picker triggers, and why the error appears there and not
during normal screen-by-screen use.

Fixed by baking the colour into the 16 affected PNGs (flat tint, alpha preserved) and removing
the keys. Identical appearance, zero novel structure. `recolor_palette` count went 856 -> 606,
and every remaining context now matches the reference exactly.

Guard to run after any tint work:

```python
# normalise paths, wildcarding item/state names; then
assert not (mine_contexts - wargames_contexts), sorted(mine_contexts - wargames_contexts)
```

**The general rule this session keeps re-learning:** the theme format is only as permissive as
the reference demonstrates. A key that validates, resolves, and renders can still be in a place
the firmware does not read it. Diff structure against `wargames`, not just values.

## Art

The ground is generated, not hand-drawn — violet wash from above, jade bloom from below,
mycelial filaments, drifting spores. The generator lives in the Claude project
(`gen_plates.py` / `stage2.py`); ask for it if a plate needs regenerating.

Status-bar glyphs are tinted through `recolor_palette: "purple"` rather than recoloured art,
so they follow the palette. The **battery** glyphs are the exception — their PNGs were
recoloured directly so the charging bolt stays yellow and the charged arrow stays green; a
flat tint would flatten both.

## Before any commit or PR

```bash
python3 validate_theme.py .          # from the pineapple-pager-theme skill; must be 0/0
pngquant --force --nofs --strip --quality 0-100 16 -o out.png in.png
```

16 colours, not 64 — the glows are low-amplitude over near-black, quantize clean at 16, and
band visibly at 12. QR codes go at 4 colours and must stay the shipped originals.

Budget to hold: **240 KB of assets, 26 full-screen plates** (`wargames`: 772 KB, 42 plates).

The validator's "3 unreferenced upgrade assets" note is a false positive — they are referenced
through `checking_image_path` and friends, which it does not scan. `wargames` reports the same.

## Status

Confirmed on hardware: the violet-text / mint-selection split, and the status-bar preview.
**Unverified:** the 20 text-layer title positions, the replaced plates, and the battery
charging states. Those are the hardware walkthrough, and they come before any PR.
