# Contributing

Thanks for helping improve the Malaysia Waktu Solat Dataset. The scope is bounded: only what JAKIM publishes via the e-Solat system. This repo covers two data products: prayer times across approximately 60 JAKIM zones, and Hijri/Gregorian tarikh. Please keep contributions inside that scope.

## Proposing a correction

Open an issue using this template:

- **Dataset file**, e.g. `data/prayer_times.csv`, `data/by_year/prayer_times_2026.csv`, `data/zones.csv`, or `data/tarikh.csv`
- **For prayer time corrections**: zone code (e.g. `WLY01`), date (`YYYY-MM-DD`), field name (`subuh`, `syuruk`, `zohor`, `asar`, `maghrib`, `isyak`), current value, proposed value
- **For tarikh corrections**: Gregorian date, field (`hijri` or `day`), current value, proposed value
- **For zone corrections**: zone code, field (`state` or `location`), current value, proposed value
- **Source URL**: an e-Solat profile link or screenshot showing the correct value
- **Notes** (optional): any caveats

A correction without a source URL will be left open until one is found. Values are not changed based on general knowledge or local memory.

## When the upstream source is wrong

This dataset tracks what JAKIM publishes on e-Solat (https://www.e-solat.gov.my). If the source data itself is incorrect, the right place to fix it is upstream with JAKIM directly. Raise the discrepancy with them first, then open an issue here once the upstream record has changed so the affected rows can be refreshed.

## Adding a missing zone

If a zone appears in e-Solat but is missing from `data/zones.csv`, open an issue with:

1. The zone code and the e-Solat URL evidencing it.
2. The normalised state label (e.g. `Selangor`).
3. The locality label as it appears in e-Solat.

The dataset is refreshed periodically against e-Solat, so missing zones are usually picked up in the next refresh. Please do not submit pull requests that wholesale replace data files; corrections happen at the row level via issues.

## What not to contribute

- **Alternative calculation-method tables.** Prayer times from MWL, ISNA, Egyptian, or UOIF methods are out of scope. This dataset is JAKIM/MABIMS only.
- **Non-Malaysian zones.** Zones outside Malaysia do not belong here.
- **Computed values not in e-Solat's response.** If the field does not appear in the e-Solat API or published schedule, do not add it.
- **Per-masjid adjustments.** If a masjid in your area starts Subuh five minutes earlier than the zone schedule, that is a local override, not a dataset correction.
- **Format changes** (column reorders, schema migrations, normalising free-text fields to integers). Open a discussion issue first.
- **Wholesale data replacements.** PRs that overwrite entire CSV or JSON files will be closed. Refreshes happen at the row level via issues, not via bulk replacement.
