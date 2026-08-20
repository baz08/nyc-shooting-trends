# NYC Shooting Incident Trends

An analysis of every shooting incident reported to the NYPD since 2006, examining how lethality varies across boroughs and how incidents concentrate by time of day.

**Stack:** R · tidyverse · ggplot2 · R Markdown

---

## Problem

NYC publishes a complete record of reported shooting incidents — one row per victim, with borough, timestamp, and an indicator for whether the incident was classified as a murder. Raw incident counts mostly track population, so they say little on their own. This analysis focuses instead on the **death rate** (deaths ÷ incidents), which asks a different question: given that a shooting occurred, how often was it fatal, and does that differ by where and when?

## Data

[NYPD Shooting Incident Data (Historic)](https://data.cityofnewyork.us/api/views/833y-fsy8/rows.csv?accessType=DOWNLOAD), NYC Open Data. Pulled live from the API at knit time, so the report re-runs against current data with no manual download.

Cleaning steps:
- Parsed `OCCUR_DATE` into a proper date type and extracted a `YEAR` field.
- Converted the `STATISTICAL_MURDER_FLAG` boolean into a numeric `death` column so it could be summed.
- Dropped columns not used in the analysis — precinct, jurisdiction code, location descriptors, and all perpetrator demographic fields.

## Analysis

**Yearly death rate.** Incidents and deaths aggregated by year, with the ratio plotted on a log scale across the full series.

**Borough comparison.** The same rate computed per borough per year, plotted against the citywide average so each borough can be read as above or below the city baseline.

**Time of day.** Deaths aggregated by incident time and plotted across the 24-hour cycle.

**Linear model.** `lm(deaths_per_year ~ cases_per_year)` fit on the yearly aggregates, with fitted values plotted against actuals.

## Findings

- **Lethality differs meaningfully by borough.** Brooklyn sits above the citywide death rate in most years, while Manhattan sits below it for the majority of the series — a gap that raw incident counts alone would not reveal.
- **Deaths concentrate heavily between roughly 8 PM and 5 AM.** The overnight window carries a visibly disproportionate share of fatal incidents relative to daytime hours.
- **Shootings are less often fatal than expected.** Going in, I assumed a much higher share of incidents would be lethal; the year-over-year death rate is consistently and substantially lower than that intuition suggested. Annual incident volume was also lower than I anticipated.

## Bias and limitations

**Reporting, not occurrence.** This dataset records incidents *reported to and recorded by* the NYPD. Under-reporting — whether from inability to report, distrust, or fear of retaliation — is not randomly distributed across neighborhoods, so both the counts and the rates carry the shape of the reporting process, not just the underlying events.

**My own priors.** I came in expecting shootings to be far more frequently lethal and more frequent overall than they turned out to be. I kept the borough aggregation uniform and left reported counts unaltered specifically so that expectation couldn't quietly shape the result — and the data contradicted me on both points.

**Known limitations in the current analysis:**
- The time-of-day aggregation groups on the exact timestamp rather than binning by hour, which makes that chart far noisier than it needs to be. Binning with `lubridate::hour()` is the obvious fix.
- The borough breakdown in the time-of-day chart doesn't isolate boroughs correctly — the summed value spans all boroughs at each timestamp, so the per-borough lines shouldn't be read as borough-specific. The citywide overnight pattern still holds; the borough split needs regrouping before any claim rests on it.
- The linear model relates yearly deaths to yearly incidents, which is close to definitional — deaths are a subset of incidents, so a strong fit is expected and not informative. A model of the rate against time, or a Poisson model of counts, would say something the raw ratio doesn't.
- Victim demographic fields are retained through cleaning but never analyzed. They're recorded by the reporting officer rather than self-reported, so they describe the reporting process as much as the population, and would need that caveat if used.

## Repo structure

```
├── NYshoooting.Rmd    # full analysis, knits to HTML/PDF
└── README.md
```

## Reproducing it

```r
install.packages("tidyverse")
rmarkdown::render("NYshoooting.Rmd")
```

Data is fetched from the NYC Open Data API at knit time — no manual download required.

## Context

Final project for DTSA 5301 (Data Science as a Field), M.S. Data Science, University of Colorado Boulder.
