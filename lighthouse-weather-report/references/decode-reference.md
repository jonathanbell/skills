# Lighthouse Report Decode Reference

Extended notes for the `lighthouse-weather-report` skill. Read this when a token
in a report does not fit the common cases in SKILL.md.

Official source (codes only; does not cover the trailing `E` or barometric
remarks): Environment Canada, "Decode information for lighthouse reports".

## Contents
- Standard 9-group report - field-by-field
- The trailing `E` (estimated wind)
- Barometric pressure remarks
- The separate supplementary (5-group) report
- Worked examples
- Edge cases and gotchas

## Standard 9-group report - field-by-field

The official format names nine groups. On the live web page they are
concatenated into one string per station, and groups with nothing to report are
dropped, so decode by shape rather than by position.

| Group | Field | Notes |
|-------|-------|-------|
| 1 | Station name | The row label, not part of the string |
| 2 | Sky condition | CLR, PC/PT CLDY, CLDY, OVC, -X, X |
| 3 | Visibility | Nautical miles (nm); trailing `F` = fog; fractions allowed |
| 4 | Weather elements | Optional; R/RW/S/SW/L/F/T/A with +/- intensity |
| 5 | Wind direction | 8 compass points; direction wind blows *from* |
| 6 | Wind speed | Knots; optional `G` gust; trailing `E` (see below) |
| 7 | Sea state | Wave height in feet + SMTH/RPLD/CHP/MOD-MDT/RUF |
| 8 | Swell | Optional; LO/MOD/HVY + 8-point direction |
| 9 | Remarks | Optional; pressure, occasional weather, etc. |

## The trailing `E` (estimated wind)

The official decode page does **not** document a trailing `E` on the wind group.
Working it out from the live data: of 17 Pacific stations observed in one fetch,
16 carried the `E` and only Langara reported wind without it (`S02`). Langara
Island has automated instrumentation; staffed lightstations estimate wind by
eye. The most consistent reading is therefore:

- **Trailing `E` = wind speed estimated** (visual), not instrument-measured.

Treat this as a well-supported inference. In output, say "estimated" and do not
present it as official. If the user disputes it, that is fine - it is a judgment
call. Calm is written `CLM` and carries no `E`.

`G` is the documented gust marker: `G<number>` gusts to that speed, bare `G`
means gusty without a recorded peak. A token like `NW23EG` is NW 23 kn,
estimated, gusty.

## Common remark tokens

The remarks group is free-form text from the keeper, but a small vocabulary
recurs. Recognise these so you do not over- or under-interpret:

| Token | Meaning |
|-------|---------|
| `1025.0R` (number + R/F/S) | Pressure in hPa/mb + tendency: R rising, F falling, S steady. Rising = settling weather; falling = system approaching. |
| `OCNL <wx>` | Occasional weather, e.g. `OCNL RW-` = occasional light rain showers. |
| `F BNK DSNT <dir>` | A **distant** fog bank, offshore, in that direction. **Not** fog at the station - do not report the station as foggy from this. |
| `SWT <n>` | Sea-water (surface) temperature in degrees Celsius. |
| `RIP MID CH` | Tide rip and/or chop mid-channel. A current/sea-state note, not a weather condition at the lighthouse itself. |
| `HALO` | Optical halo around the sun or moon - usually high cirrus. Mention as-is. |

When in doubt about a remark, reproduce it verbatim in the report rather than
inventing an interpretation.

## Fog at the station vs. fog in remarks (important)

Lightstation reports distinguish "fog at the station" from "fog visible
offshore." The skill must not confuse them:

- **Fog at the station** is signalled by `F` in the visibility group
  (e.g. `06F` = 6 nm in fog, `1/4F` = 1/4 nm in fog) or by `F` appearing as a
  standalone weather-element token between visibility and wind.
- **Fog elsewhere** appears as a remark: `F BNK DSNT <dir>` means a fog bank is
  visible in the distance from that direction. The station itself may have full
  visibility - report it that way.

Example: `PC 15 S12E 2FT CHP F BNK DSNT SW` = partly cloudy, 15 nm visibility
(no fog at the station), S wind ~12 kn estimated, 2 ft choppy seas; a distant
fog bank is visible to the SW.

## Supplementary (5-group) report

Some stations also issue a separate "Supplementary Weather Information" report
with a different shape:

`<Station> <Time UTC> <Sky layers> <Temperature C> <Dew point C>`

- **Time**: 4-digit UTC, e.g. `2030`.
- **Sky layers**: cloud described in layers as `<amount> <height>`, where amount
  is `CLR` clear, `FEW` few (1-2 eighths), `SCT` scattered (3-4), `BKN` broken
  (5-7), `OVC` overcast (8 eighths); height in hundreds of feet. `ABV 25` means
  cloud above 2500 ft. Example: `CLD EST 15 FEW 25 FEW OVC ABV 25`.
- **Temperature** and **Dew point**: whole degrees Celsius.

This skill's main job is the standard 9-group report. Only decode a
supplementary report if the user explicitly asks for temperature/dew-point data
or pastes one.

## Worked examples

`CAPE BEALE  CLDY 06F NW09E 2FT CHP LO SW`
Cloudy; 6 nm visibility in fog; NW wind ~9 kn (estimated); seas 2 ft, choppy;
low swell from the SW.

`BOAT BLUFF  X 1/4F NW02E RPLD`
Sky obscured; 1/4 nm visibility in fog; NW wind ~2 kn (estimated); rippled seas;
no swell reported.

`CARMANAH  OVC 15 SW E22G30 2FT CHP LO-MOD SW OCNL RW-`
Overcast; 15 nm visibility; light snow shower; E wind 22 kn gusting 30; seas
2 ft, choppy; low-to-moderate swell from the SW; occasional light rain showers.

`ESTEVAN  PC 15 NE20E 4FT MDT LO SW 1025.0R`
Partly cloudy; 15 nm visibility; NE wind ~20 kn (estimated); seas 4 ft,
moderate; low SW swell; pressure 1025.0 mb and rising.

## Edge cases and gotchas

- **`N/A` or blank**: station not reporting - almost always outside daylight
  observing hours (stations report roughly every 3 hours during daylight). Say
  so plainly; do not invent data.
- **`SW` ambiguity**: in the weather-element slot it is *snow shower*; in the
  swell slot it is the *southwest* direction. Position disambiguates.
- **`MOD` vs `MDT`**: both mean moderate. `MOD` also appears as a swell height.
  Context (sea-state slot vs swell slot) tells them apart.
- **Missing sea height**: a bare `RPLD`/`SMTH` with no `<n>FT` is fine - report
  just the descriptor.
- **Ranges**: values like `LO-MOD` mean a range (low to moderate).
- **Station-name collisions**: BC has more than one "Chatham" and similar names.
  Match within the expected region from the SKILL.md mapping.
- **WebFetch drift**: WebFetch summarizes through a small model and may
  paraphrase or drop `N/A` rows. Always ask it for the *verbatim raw string*,
  and if the 14 mapped stations are not all accounted for, fetch the `mapID=02`
  page too before concluding a station is missing.
