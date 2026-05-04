# Malaysia Waktu Solat Dataset

A structured table of Islamic prayer times for all 60 JAKIM zones in Malaysia, covering 2020-01-01 to 2027-12-31 across 175,200 rows (60 zones multiplied by 8 years of daily schedules), plus 10,224 Hijri-to-Gregorian date mappings spanning from 2000-01-01 to 2027-12-28. All data is sourced from JAKIM's e-Solat portal (`e-solat.gov.my`). Published as machine-readable CSV and JSON.

## Summary

**175,200 prayer-time rows** across 60 zones. **10,224 Hijri-Gregorian mappings** covering 28 years.

### Prayer-time rows by state

| State | Count |
| :---- | ----: |
| Sabah | 26,280 |
| Sarawak | 26,280 |
| Kedah | 20,440 |
| Pahang | 20,440 |
| Perak | 20,440 |
| Johor | 11,680 |
| Terengganu | 11,680 |
| Negeri Sembilan | 8,760 |
| Selangor | 8,760 |
| Kelantan | 5,840 |
| Melaka | 2,920 |
| Perlis | 2,920 |
| Penang | 2,920 |
| Kuala Lumpur (FT) | 2,920 |
| Labuan (FT) | 2,920 |
| **Total** | **175,200** |

Putrajaya is grouped with Kuala Lumpur under JAKIM zone WLY01 ("Kuala Lumpur, Putrajaya"). It does not appear as a separate state in JAKIM's zone partitioning.

### Zones by state

| State | Count |
| :---- | ----: |
| Sabah | 9 |
| Sarawak | 9 |
| Kedah | 7 |
| Pahang | 7 |
| Perak | 7 |
| Johor | 4 |
| Terengganu | 4 |
| Negeri Sembilan | 3 |
| Selangor | 3 |
| Kelantan | 2 |
| Melaka | 1 |
| Perlis | 1 |
| Penang | 1 |
| Kuala Lumpur (FT) | 1 |
| Labuan (FT) | 1 |
| **Total** | **60** |

### Tarikh

10,224 daily Gregorian-to-Hijri mappings, 2000-01-01 to 2027-12-28, sourced from JAKIM's `tarikhtakwim` endpoint.

## Data files

- **`data/prayer_times.csv`** / **`data/prayer_times.json`**: 175,200 rows, all 60 zones, 2020-2027.
- **`data/by_year/prayer_times_{YYYY}.csv`** / **`data/by_year/prayer_times_{YYYY}.json`**: 21,900 rows each, one file per year 2020-2027. (Leap years have 21,900 rows, not 21,960; see Limitations.)
- **`data/zones.csv`** / **`data/zones.json`**: 60 JAKIM zone entries with state and location fields.
- **`data/tarikh.csv`** / **`data/tarikh.json`**: 10,224 Hijri-Gregorian mappings.

CSV files are UTF-8, comma-separated, snake_case headers. JSON files wrap the rows in a `metadata` block plus an array named after the dataset (`prayer_times`, `zones`, `tarikh`).

The combined `data/prayer_times.json` is 47 MB and the combined `data/prayer_times.csv` is 12 MB. GitHub's web UI does not preview files of that size inline. To browse a slice of the data on GitHub, open one of the per-year files under `data/by_year/` (each is around 6 MB). To work with the full file, clone the repo or `curl` the raw URL.

## Quickstart

### Get the data

Clone the whole repo:

```bash
git clone https://github.com/abualif120/malaysia-waktu-solat-dataset.git
cd malaysia-waktu-solat-dataset
```

Or pull individual files via raw URL:

```bash
curl -O https://raw.githubusercontent.com/abualif120/malaysia-waktu-solat-dataset/main/data/prayer_times.csv
curl -O https://raw.githubusercontent.com/abualif120/malaysia-waktu-solat-dataset/main/data/zones.csv
curl -O https://raw.githubusercontent.com/abualif120/malaysia-waktu-solat-dataset/main/data/tarikh.csv
curl -O https://raw.githubusercontent.com/abualif120/malaysia-waktu-solat-dataset/main/data/by_year/prayer_times_2027.csv
```

### pandas (Python)

```python
import pandas as pd

# Load and inspect
pt = pd.read_csv("data/prayer_times.csv")
print(pt.shape)   # (175200, 11)
print(pt.dtypes)

# Today's Subuh time for Selangor zone SGR01
today = pt[(pt["zone"] == "SGR01") & (pt["date"] == "2026-05-04")]
print(today[["zone", "date", "subuh"]])

# Convert a Gregorian date to its Hijri equivalent via tarikh
tarikh = pd.read_csv("data/tarikh.csv")
hijri_today = tarikh.loc[tarikh["gregorian"] == "2026-05-04", "hijri"].values[0]
print(hijri_today)   # e.g. 1447-11-06

# Count prayer-time rows by state (join zones)
zones = pd.read_csv("data/zones.csv")
merged = pt.merge(zones, on="zone")
print(merged.groupby("state").size().sort_values(ascending=False))

# Pull just one year without cloning the whole repo
url_2027 = "https://raw.githubusercontent.com/abualif120/malaysia-waktu-solat-dataset/main/data/by_year/prayer_times_2027.csv"
pt_2027 = pd.read_csv(url_2027)
print(pt_2027.shape)
```

### R

```r
library(readr)

pt     <- read_csv("data/prayer_times.csv")
tarikh <- read_csv("data/tarikh.csv")

# Subuh times for zone SGR01 in January 2026
sgr01_jan <- pt[pt$zone == "SGR01" & startsWith(pt$date, "2026-01"), ]
print(sgr01_jan[, c("date", "subuh", "maghrib")])

# Hijri dates for a given Gregorian month
ramadan_check <- tarikh[startsWith(tarikh$gregorian, "2026-03"), ]
print(ramadan_check)
```

### jq (shell)

```bash
# All zones in Sabah
jq -r '.zones[] | select(.state == "Sabah") | "\(.zone)\t\(.location)"' data/zones.json

# Subuh time for zone SGR01 on 2026-05-04
jq -r '.prayer_times[] | select(.zone == "SGR01" and .date == "2026-05-04") | .subuh' data/prayer_times.json

# Count rows by year
jq -r '.prayer_times | group_by(.date[0:4]) | map({year: .[0].date[0:4], n: length}) | .[] | "\(.year)\t\(.n)"' data/prayer_times.json

# Find the Hijri date for 2026-05-04
jq -r '.tarikh[] | select(.gregorian == "2026-05-04") | "\(.gregorian) -> \(.hijri)"' data/tarikh.json
```

## Schema

### `data/prayer_times.csv`

| Column | Type | Notes |
| :---- | :---- | :---- |
| `zone` | string | JAKIM zone code (e.g. `SGR01`, `KDH01`). Foreign key to `zones.csv`. |
| `date` | string | ISO 8601 date, `YYYY-MM-DD`. Gregorian calendar. |
| `day` | string | English weekday name (`Monday`, `Tuesday`, ...). Verified against date arithmetic; see note below. |
| `imsak` | string | Pre-dawn abstention time, `HH:MM` (24-hour, UTC+8). Typically subuh minus 10 minutes. |
| `subuh` | string | Fajr (dawn prayer), `HH:MM`. |
| `syuruk` | string | Sunrise, `HH:MM`. |
| `dhuha` | string | Mid-morning voluntary prayer time, `HH:MM`. May be empty for very old API responses, but present for nearly all rows. |
| `zohor` | string | Dhuhr (midday prayer), `HH:MM`. |
| `asar` | string | Asr (afternoon prayer), `HH:MM`. |
| `maghrib` | string | Maghrib (sunset prayer), `HH:MM`. |
| `isyak` | string | Isha (night prayer), `HH:MM`. |

All times are 24-hour `HH:MM` format, Malaysia Time (UTC+8). The `day` column was verified against calendar arithmetic. Values may differ from the `day` field in JAKIM's source for a small number of dates in late 2027, where JAKIM's source had an off-by-N day-of-week error. The corrected values are published here.

### `data/zones.csv`

| Column | Type | Notes |
| :---- | :---- | :---- |
| `zone` | string | JAKIM zone code. Primary key. Matches the pattern `[A-Z]{3}\d{2}` (e.g. `SGR01`, `PHG07`, `SBH09`). |
| `state` | string | Malaysian state or federal territory. Normalised to the consistent taxonomy used across the `malaysia-*-dataset` series. |
| `location` | string | JAKIM's location description for the zone (e.g. `Gombak, Petaling, Sepang, Hulu Langat, Hulu Selangor, S.Alam`). Preserved as published. |

### `data/tarikh.csv`

| Column | Type | Notes |
| :---- | :---- | :---- |
| `gregorian` | string | Gregorian date in ISO 8601 format (`YYYY-MM-DD`). Primary key. |
| `hijri` | string | Corresponding Hijri date (`YYYY-MM-DD`). Follows JAKIM's official takwim, which uses rukyah-based corrections. |
| `day` | string | English weekday name. Consistent with the `day` column in `prayer_times.csv` for overlapping dates. |

### Empty values

In CSV, missing data is represented by an empty cell. In JSON, missing data is represented by `null`. Source-side null markers (`-`, `TIADA`, and missing entries from partial API responses) are all normalised to empty or null here.

### What is intentionally not included

- **Future Hijri months JAKIM has not yet published.** The tarikh window stops at 2027-12-28, where the API begins to have gaps and a misaligned-month bug at Hijri 1450-04. Rows beyond that date are excluded rather than published with unreliable values.
- **Per-masjid prayer-time adjustments.** JAKIM publishes one canonical schedule per zone. Individual masjid often start prayers a few minutes earlier or later as a matter of jemaah practice. Those local adjustments are not part of this dataset.
- **Alternative astronomical-method tables.** MWL, ISNA, Egyptian, UOIF, and other method variants are out of scope. This dataset is JAKIM/MABIMS only.

## Sources

### Primary

**JAKIM e-Solat** at `https://www.e-solat.gov.my`, operated by Jabatan Kemajuan Islam Malaysia. Two API endpoints used:

- `esolatApi/takwimsolat` (prayer times by zone and period)
- `esolatApi/tarikhtakwim` (Hijri-Gregorian date mapping)

### Out of scope

State-level Islamic authority published tables (JAIS Selangor, JAINJ Johor, JAIS Sabah, and others) sometimes publish their own slightly different schedules. Those are not folded into this compilation. This dataset tracks what JAKIM's national portal publishes.

## Provenance

| File | Upstream source | Snapshot date | Rows |
| :---- | :---- | :---- | ----: |
| `data/prayer_times.csv` / `.json` | JAKIM e-Solat, `esolatApi/takwimsolat` | 2026-05-04 | 175,200 |
| `data/zones.csv` / `.json` | JAKIM e-Solat, zone metadata | 2026-05-04 | 60 |
| `data/tarikh.csv` / `.json` | JAKIM e-Solat, `esolatApi/tarikhtakwim` | 2026-05-04 | 10,224 |

The data was collected from the public API endpoints listed above on the snapshot date shown. This repository does not host the upstream API. To verify any individual row, query the e-Solat portal directly with the corresponding zone code and date.

### What was changed

- Column names translated from JAKIM's API field names to English snake_case. JAKIM's Arabic-transliteration names (`fajr`, `dhuhr`, `asr`, `isha`) were mapped to the Malay-transliteration names that match Malaysian usage (`subuh`, `zohor`, `asar`, `isyak`).
- State names normalised to a consistent taxonomy matching the wider `malaysia-*-dataset` series (`PULAU PINANG` to `Penang`, `W.P. KUALA LUMPUR` to `Kuala Lumpur (FT)`, `W.P. LABUAN` to `Labuan (FT)`, etc.).
- Time strings normalised to `HH:MM` (stripping seconds where the API returned `HH:MM:SS`).
- Day-of-week values verified against calendar arithmetic and corrected where JAKIM's source disagreed (only late-2027 dates affected).
- JAKIM API metadata fields dropped: `status`, `serverTime`, `dateRequest`, `bearing`, `lang`, `periodType`.
- Source-side null markers (`-`, `TIADA`, missing entries from incomplete API pages) normalised to empty cells in CSV and `null` in JSON.

### What was preserved verbatim

Prayer-time `HH:MM` values as computed by JAKIM. Zone codes (`SGR01`, `KDH03`, etc.). Zone location labels as published. The JAKIM zone partitioning itself (which districts fall in which zone).

## Limitations

- **Times are JAKIM's astronomical computations, not field-observed.** The muezzin's actual azan may differ from the published time by a few minutes at most masjid.
- **MABIMS criteria only.** JAKIM uses the MABIMS convention (the standard for Brunei, Indonesia, Malaysia, and Singapore) for fajr and isha angle thresholds. Other Islamic authorities use different angle conventions and will compute different times for the same date and location.
- **`imsak` is derived, not independently observed.** For most zone-days it is subuh minus 10 minutes, but JAKIM's published values do not always follow that rule. Two patterns to be aware of:
  - **Perlis (zone PLS01) publishes `imsak` equal to `subuh`** for every day in the dataset (2,920 rows). This is JAKIM's published convention for that zone, not a calculation error here.
  - A small number of Sabah-zone days (around 320 rows out of 175,200) carry `imsak`/`subuh` differences other than 10 minutes (mostly 11, 20, or 21 minutes). These follow JAKIM as published.
- **One known JAKIM source-data typo: SBH08 on April 3 every year.** JAKIM's e-Solat API returns `imsak: 15:20` for zone SBH08 (Tawau, Lahad Datu and surrounding areas in Sabah) on April 3 of every year from 2020 through 2027. The value is clearly a typo in JAKIM's source (15:20 is the asr time on the same day, suggesting a copy-paste). The dataset preserves the value as published. Treat the SBH08 April 3 `imsak` field as unreliable until JAKIM corrects upstream. All other prayer fields on those rows are sensible.
- **Hijri dates follow JAKIM's official takwim.** JAKIM uses observation-based (rukyah) corrections applied to a base arithmetic calendar. Other authorities (Saudi Umm-al-Qura, the arithmetic Tabular Hijri Calendar) can differ by plus or minus one day on month boundaries.
- **Leap-day gap.** JAKIM's `takwimsolat` API consistently returns 28 days for February in leap years. February 29, 2020 and February 29, 2024 are absent for all zones. As a result, leap years have 365 prayer-time rows per zone, not 366. The per-year files are named accurately but row counts will be 365, not 366, in 2020 and 2024. Refresh once JAKIM patches their endpoint.
- **Tarikh range cap at 2027-12-28.** JAKIM's `tarikhtakwim` API has known gaps starting late 2027 and a misaligned-month bug at Hijri 1450-04 where the response dates are off by a full Hijri year. The published `data/tarikh.csv` is trimmed to the last fully contiguous Gregorian date (2027-12-28) before any glitch. A future refresh extends the window once JAKIM's data covers 2028 onward without holes.
- **Prayer times end at 2027-12-31.** Future years not yet published by JAKIM are absent. Times beyond 2027-12-31 require a new snapshot.
- **New zones created after 2020-01-01 would have a shorter coverage window.** No such gaps were observed in the current snapshot, but a future JAKIM rezoning could introduce them.
- **This is the national schedule only.** Individual states' Islamic authorities (JAIS, JAINJ, etc.) sometimes publish their own slightly different schedules. Those are not reflected here.
- **All times are Malaysia Time (UTC+8).** Malaysia does not observe daylight saving time, so no seasonal offset adjustment is needed.

## Privacy and personal data

This dataset contains no personal data. Prayer times are astronomical values computed and published by JAKIM. Zone codes are public administrative classifications. Hijri-Gregorian mappings are calendrical facts derived from JAKIM's takwim. No individual is identified in any row.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for how to file corrections. In short: open an issue with the upstream e-Solat URL or API response that contradicts a row, and the maintainer will verify and correct it. A PR with a corrected value is also welcome, provided the issue references the JAKIM source.

This dataset tracks what JAKIM publishes. If JAKIM has not yet published prayer times for a given zone or year, the right fix is to wait for a new snapshot, not to fill in computed values from a third-party library.

## License and terms of use

### Compilation licence

This compilation is released under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**. You are free to share and adapt the dataset for any purpose, including commercially, provided you give appropriate credit, link to the licence, and indicate any changes you made. Full text in [`LICENSE`](LICENSE).

The CC BY 4.0 licence here covers **this compilation**: the schema design, English snake_case column names, the Malay-transliteration column naming (`subuh`, `zohor`, `asar`, `isyak`), the normalised state taxonomy, the day-of-week correction, and the editorial decisions documented under Provenance above.

### What is and is not copyrighted

Underlying factual data (prayer times, Hijri dates, zone codes) is not itself copyrightable in Malaysia or in most jurisdictions. Facts cannot be owned. What can be owned is the creative selection and arrangement that turns raw API responses into a structured compilation. That arrangement is what CC BY 4.0 on this repository covers.

If you reuse the upstream API responses directly (the JAKIM e-Solat portal endpoints), that source's own terms apply, not this compilation's licence.

### Upstream-source terms

JAKIM's e-Solat at `www.e-solat.gov.my` is a Malaysian federal government publication intended for public citizen use. Its terms of use are not always explicitly published. In their absence, reuse for non-commercial research and civic purposes is conventional practice in Malaysia. If you redistribute large verbatim extracts of raw JAKIM API responses, that part is subject to JAKIM's terms directly. CC BY 4.0 on this compilation does not relicense the upstream API output.

### Fair dealing under Malaysian law

Section 13 of the Copyright Act 1987 (Malaysia) recognises fair dealing exceptions for purposes including research, private study, criticism, review, and reporting of current events. This compilation falls within research and statistical-reporting purposes. Most academic, civic, and app-development uses of this dataset sit comfortably within fair dealing.

### What you must do (CC BY 4.0 attribution)

- Credit the dataset, ideally using the citation block below.
- Link to the licence (`https://creativecommons.org/licenses/by/4.0/`).
- Indicate any changes you made if you modified the data.

### Disclaimer

This dataset is provided **as is**, without warranty of any kind, express or implied, including warranties of accuracy, completeness, or fitness for a particular purpose. Verify critical data points against the upstream source URLs before relying on them for safety, financial, or legal decisions. The maintainer is not liable for any loss or damage arising from use of this dataset.

## Citation

Suggested citation:

> abualif120. (2026). *Malaysia Waktu Solat Dataset.* 175,200 prayer-time rows across 60 JAKIM zones for 2020-2027, plus 10,224 Hijri-Gregorian mappings spanning 2000-2027, compiled from JAKIM e-Solat. Available at: https://github.com/abualif120/malaysia-waktu-solat-dataset. Licensed under CC BY 4.0.

BibTeX:

```bibtex
@dataset{abualif120_malaysia_waktu_solat_2026,
  author  = {{abualif120}},
  title   = {Malaysia Waktu Solat Dataset},
  year    = {2026},
  note    = {175,200 prayer-time rows across 60 JAKIM zones (2020-2027) plus 10,224 Hijri-Gregorian mappings, compiled from JAKIM e-Solat},
  url     = {https://github.com/abualif120/malaysia-waktu-solat-dataset},
  license = {CC-BY-4.0}
}
```

## See also

- [malaysia-airports-dataset](https://github.com/abualif120/malaysia-airports-dataset). Sister dataset, 113 Malaysian aviation nodes.
- [malaysia-masjid-dataset](https://github.com/abualif120/malaysia-masjid-dataset). Sister dataset, 26,387 masjid and surau across Malaysia.
