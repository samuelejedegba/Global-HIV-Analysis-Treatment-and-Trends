# Global HIV Analysis — Power BI Dashboard

An end-to-end analysis of the global HIV epidemic built in Power BI, using WHO
indicator data. The project moves from raw data to an interactive three-page
dashboard, with a focus on asking the right analytical question rather than
just reporting numbers.

![Dashboard Overview](Overview%20-%20HIV%20analysis.PNG)

## The Questions

This analysis set out to answer three questions:

1. What is the current global scale of the HIV epidemic, and is treatment working?
2. Where does the burden fall hardest, and which countries are most at risk?
3. Is treatment coverage keeping pace with new infections?

## The Data

The dataset is drawn from WHO HIV indicators covering roughly 150 countries.
One feature of the data shaped almost every decision in the build: **the
indicators are not all reported on the same cadence.**

- **HIV deaths** are reported every year across the full range (2000–2024).
- **New infections** and **ART coverage** are only reported in nine specific
  years (2000, 2005, 2010, 2015, and 2020–2024).

This mismatch matters. Any measure that combines a yearly indicator with a
nine-year indicator, for example, deaths per new infection, will compare
different spans of time unless the years are aligned first. To handle this, a
custom `IsCommonReportingYear` flag was built in Power Query (using
`List.Contains`) so that cross-indicator measures could be restricted to the
years where both numbers genuinely exist.

## Approach

The work followed four stages: data preparation and modelling in Power Query,
DAX measure development, dashboard design, and a written findings review. The
model uses a star-schema layout with the indicator data in long (tall) format,
one row per country-indicator-year.

## Key Measures

A few of the measures that drive the dashboard, and the reasoning behind them:

**PLHIV (Latest Year)** — People currently living with HIV. Built with
`LASTNONBLANK` so that each country reports its own most recent year of data
rather than being forced to a single hard-coded year. This is a *stock* measure
(a cumulative total that carries forward), so it must be read at its latest
value, not summed across years.

> Headline figures: **35M** people currently living with HIV, against a
> **mortality rate of 3.8%** and a **survival rate of 96.2%** — the numbers
> behind the "treatment is working" story.

**Total New Infections / Total HIV Deaths** — These are *flow* measures (yearly
events), so summing across years is correct here.

**Deaths per 1,000 New Infections** — `DIVIDE([Total HIV Deaths],
[Total New Infections]) * 1000`, restricted to common reporting years so the two
figures cover the same span. Without that restriction the ratio inflates badly
(25 years of deaths over 9 years of infections).

**ART Coverage (Weighted)** — Built with `SUMX` iterating country by country to
rebuild real treatment headcounts (`PLHIV × coverage%`) before combining, rather
than taking a plain average of percentages. A simple average treats a country's
90% and another's 10% as equal; the weighted version respects how many people
sit behind each percentage.

> The two figures diverge: **simple average = 44.82%**, but **weighted =
> 51.1%**. The weighted figure sits higher because the largest-population
> countries tend to have above-average coverage, so weighting by headcount pulls
> the global figure up. Reporting the weighted version is the honest headline:
> it reflects the coverage the *typical person living with HIV* actually
> experiences, not the average across countries regardless of size.

**Untreated Population** — `PLHIV × (1 - ART Coverage)`. Counts *people* going
without treatment rather than reporting a coverage rate, which surfaces where
the absolute need is greatest.

**Countries Below 50% ART** — a count of countries where treatment coverage
falls under half.

## The Stock vs Flow Lesson

The single most important analytical decision in the project was recognising
that **the same column, aggregated two different ways, produces two completely
different and differently meaningful numbers.**

Summing the PLHIV column across all years produced a figure in the hundreds of
millions. Reading it at its latest year produced ~35 million. The maths was
correct in both cases; the *choice of aggregation* was everything. Summing a
cumulative stock measure double-counts the same people year after year and
answers a meaningless question. Knowing which aggregation matches the question
is the difference between writing a formula and doing analysis.

## A Correction Worth Recording

**Thailand** tops the deaths-per-1,000-new-infections chart, which at first
glance suggests poor treatment performance. On checking the source data, the
real explanation was different: it reflects a **shrinking epidemic**. New
infections have fallen sharply, so the ratio of deaths to *new* infections rises
even as the country's HIV situation improves. The finding was stated honestly
rather than left to imply failing care, a reminder that a surprising number is
a prompt to verify, not to publish.

## Beyond the Brief — The Prevention/Treatment Quadrant

The finding the project is built around isn't in the original question list.
Plotting each country on two axes, **treatment coverage** (how well it treats
those already infected) against **new infections** (how well it prevents new
cases), creates four quadrants.

The revealing group sits in the "treats well, but still generating many new
infections" quadrant: several **high-burden African countries have adequate ART
coverage yet still produce the most new infections.** In plain terms, they are
doing the treatment job but the prevention tap is still running. That reframes
the policy question from "treat more" to "prevent more" for exactly the
countries a coverage-only view would rate as succeeding.

## Recommendations

What the data suggests policymakers should do:

**1. Pair prevention with treatment funding.** The quadrant shows high-coverage
countries still generating new infections. Treatment budgets need to be matched
with prevention investment, or the epidemic sustains itself.

**2. Target the 94 under-served countries.** The global average hides that 94 of
150 countries remain below 50% ART coverage. Directing resources there, weighted
by HIV burden, delivers the greatest impact.

**3. Direct resources to the largest untreated populations.** South Africa,
Nigeria and Mozambique have the most people living with HIV but not on treatment.
This is where expanding treatment access saves the most lives — a sharper target
than the deaths-per-1,000 ranking, which is inflated by shrinking epidemics
rather than poor care.

## Files in This Repository

- `Overview - HIV analysis.PNG` — page 1: Global HIV Trends & Treatment Effectiveness
- `Treatment and Mortality - HIV analysis.PNG` — page 2: Treatment Coverage and Mortality
- `Geography and Risk - HIV analysis.PNG` — page 3: Geography and Risk
- `HIV_Analysis.pbix` — the full Power BI file (model, measures, and visuals)

## Tools

Power BI Desktop (Power Query, DAX, star-schema modelling, custom GeoJSON map),
WHO HIV indicator data.

---

*Data source: WHO HIV indicators. Figures reflect the reporting years available
in the dataset and should be read alongside the reporting-cadence note above.*
