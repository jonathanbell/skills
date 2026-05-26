---
name: lighthouse-weather-report
description: >-
  Fetches and decodes BC coast lighthouse (lightstation) weather reports from
  Environment Canada and turns the cryptic 9-group code into a clear,
  human-readable markdown report. Use this whenever the user asks about
  lighthouse weather, lightstation reports, BC coast or Vancouver Island marine
  conditions, weather.gc.ca lightstation data, or wants a coded string like
  "CLDY 15 NW09E 2FT CHP LO SW" decoded into plain language - even if they do
  not say the word "skill" or name a station. Also triggers on requests to
  check coastal/marine weather for Vancouver Island stations such as Cape
  Beale, Estevan, Nootka, Lennard, Chatham, Quatsino, Cape Scott, Trial
  Island, Chrome, Merry, or Pulteney.
---

# Lighthouse Weather Report

Environment Canada publishes weather observed by Canadian Coast Guard keepers at
staffed lightstations along the BC coast. The reports are accurate but written
in a terse coded format that is hard to read at a glance. This skill fetches the
live data, decodes it, and produces one combined, human-readable markdown report
for a fixed set of Vancouver Island stations.

## Workflow

1. **Fetch the live data.** Use WebFetch on the Pacific lightstation page:
   `https://weather.gc.ca/marine/weatherConditions-lightstation_e.html?mapID=01`

   Ask WebFetch to return, **verbatim**, the raw coded report string for each
   station, plus the "Issued ... UTC" timestamp for each region. A good prompt:
   *"For each lightstation listed, give the station name and its raw report
   string exactly as shown, with no interpretation. Also give the issued
   date/time for each region heading. Include stations whose value is N/A."*

   If any of the 14 stations below are missing from that page, fetch
   `...?mapID=02` as well and look there. Stations report roughly every 3 hours
   **during daylight only**, so at night most will read `N/A` - that is normal.

2. **Decode each station** using the tables further down. Decode only the 14
   stations in the mapping below - ignore all other stations on the page.

3. **Write one combined markdown report** using the output format below, grouped
   by the three regions. Print it in the conversation. Only write it to a `.md`
   file if the user asks for a file.

For unusual tokens, barometric remarks, or the separate 5-group "supplementary"
temperature/dew-point format, see `references/decode-reference.md`.

## Stations to report (the only ones that matter)

Report exactly these 14, under these three region headings. Match by station
name; if a name is ambiguous on the page, take the one in the expected region.

**West Coast Vancouver Island South**
| KEY | General location |
|-----|------------------|
| NOOTKA | Southern tip of Nootka Island (slightly protected) |
| ESTEVAN | Between Nootka Island and Flores Island |
| LENNARD | South of Wickaninnish Island |
| CAPE BEALE | South of the Broken Group, near the northern start of the West Coast Trail |

**Vancouver Island North**
| KEY | General location |
|-----|------------------|
| CAPE MUDGE | Southern Quadra Island / Campbell River |
| CHATHAM | North of Quadra Island |
| PULTENEY | Port McNeill |
| SCARLETT | Northwest of Port Hardy |
| CAPE SCOTT | Cape Scott |
| QUATSINO | Between Cape Scott and Brooks Peninsula |

**Vancouver Island South**
| KEY | General location |
|-----|------------------|
| TRIAL IS | Trial Island |
| MERRY | Near Sechelt / northeastern side of the Strait of Georgia |
| ENTRANCE | South end of the Strait of Georgia |
| CHROME | Denman Island |

## Decoding the report string

A standard report is a space-separated string of groups in this fixed order.
Some groups are omitted when there is nothing to report, so decode by **shape**,
not by counting positions.

`<Sky> <Visibility> [Weather] <Wind> <Sea state> [Swell] [Remarks]`

A value of `N/A` (or a blank/missing string) means the station is not currently
reporting - usually because it is outside daylight observing hours.

### Sky
| Code | Meaning |
|------|---------|
| CLR | Clear |
| PC / PT CLDY | Partly cloudy |
| CLDY | Cloudy |
| OVC | Overcast |
| -X | Partly obscured |
| X | Obscured (sky hidden, e.g. by fog) |

### Visibility
A number in **nautical miles** - the marine convention, and the same unit the
rest of a marine report uses, so report it as `nm` (`15` = 15 nm). Fractions
occur (`1/4` = quarter nautical mile). A trailing `F` means fog is the cause:
`06F` = 6 nm in fog, `1/4F` = 1/4 nm in fog.

### Weather elements (often absent)
`R` rain, `RW` rain shower, `S` snow, `SW` snow shower, `L` / `-L` drizzle,
`F` fog, `T` thunder, `A` hail. Intensity prefix: `+` heavy, `-` light
(`RW-` = light rain shower). Note `SW` here means *snow shower*; the same
letters in the swell group mean *southwest* - position tells them apart.

### Wind
Shape: `<direction><speed>[E][G[gust]]`, or `CLM` for calm.
- **Direction**: one of N, NE, E, SE, S, SW, W, NW - the direction the wind
  blows *from*.
- **Speed**: knots (1 kn approximately 1.85 km/h).
- **Trailing `E`**: the speed was **estimated** by eye rather than measured by
  instrument. This is an inference, not from the official docs - so phrase it as
  "estimated" and do not over-claim. (Evidence: nearly every station carries the
  `E`; the lone exception is Langara, which has automated instruments.)
- **`G`**: gusting. `G` followed by a number = gusting to that speed
  (`22G30` = 22 kn gusting 30). `G` with no number = gusty.

Examples: `NW09E` = NW wind, ~9 kn (estimated). `NE20E` = NE wind, ~20 kn
(estimated). `E22G30` = E wind, 22 kn gusting 30. `CLM` = calm.

### Sea state
Shape: `[<height>FT] <state>`. Height is wave height in feet (may be absent).
State: `SMTH` smooth, `RPLD` rippled, `CHP` choppy, `MOD` or `MDT` moderate,
`RUF` rough. Example: `2FT CHP` = 2 ft choppy seas; `RPLD` alone = rippled.

### Swell (often absent)
Shape: `<height> <direction>`. Height: `LO` low (0-2 m), `MOD` moderate
(2-4 m), `HVY` heavy (over 4 m). Direction is the 8-point compass bearing the
swell comes *from*. Example: `LO SW` = low swell out of the southwest.

### Remarks (often absent)
Anything trailing. Common patterns:
- **Pressure** like `1025.0R`: hPa (mb) + tendency letter (`R` rising,
  `F` falling, `S` steady).
- **Occasional weather**: `OCNL RW-` = occasional light rain showers.
- **Distant fog bank**: `F BNK DSNT <dir>` (e.g. `F BNK DSNT SW`) means a fog
  bank is visible offshore in that direction - **this is not fog at the
  station**. Do not call the station foggy from a remark like this.
- **Sea-water temperature**: `SWT 11.9` = sea-surface temp 11.9 C.
- **Tide rips / chop**: `RIP MID CH` = tide rip and chop mid-channel.
- **Optical phenomenon**: `HALO` = halo around sun or moon.

See `references/decode-reference.md` if a remark is unclear.

**Important:** fog is at the station only when `F` is in the **visibility group**
(e.g. `06F`, `1/4F`) or as a standalone **weather-element** token. A `F BNK ...`
in remarks describes weather offshore, not conditions at the lighthouse.

## Output format

Produce one markdown document. Lead with a title and a provenance line, then the
three region sections in the order above. For each station give a friendly
one-or-two sentence summary in plain English, then the raw code on its own line
so a reader can verify. Keep summaries concise - this is a quick-read briefing,
not an essay.

```markdown
# BC Lighthouse Weather Report

Fetched <local date/time> from Environment Canada (weather.gc.ca).
Observations issued <UTC timestamp(s) from the page>.

## West Coast Vancouver Island South

### NOOTKA - Southern tip of Nootka Island (slightly protected)
Clear skies, 15 nm visibility. North wind around 7 kn (estimated). Seas 1 ft and
choppy, with a low southwest swell.
`Raw: CLR 15 N07E 1FT CHP LO SW`

### ESTEVAN - Between Nootka Island and Flores Island
Partly cloudy, 15 nm visibility. Northeast wind around 20 kn (estimated) - the
strongest in the area. Seas 4 ft and moderate, low southwest swell. Pressure
1025.0 mb and rising.
`Raw: PC 15 NE20E 4FT MDT LO SW 1025.0R`
```

For a station reading `N/A`, write a single line such as:
`*No current report (outside daylight reporting hours).*` - keep the KEY and
general-location heading so the report is always complete.

If the user asks for only one region or one station, still use this format but
include just what they asked for.
