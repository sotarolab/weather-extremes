# Weather Extremes — Analysis & Attribution

Event-driven analyses combining statistical extreme value theory with observational data and numerical weather prediction. Each analysis starts from a real forecast or observed event, builds a statistical baseline from long-term station records, and quantifies rarity and — where applicable — the role of anthropogenic climate change.

## Analyses

| Event | Location | Method | Post |
|-------|----------|--------|------|
| [DC Heatwave — July 2026](analyses/dc_heatwave_2026/) | Reagan National (KDCA) | GEV · NS-GEV · WWA attribution | [LinkedIn](#) |

## Methodology

All attribution analyses follow the **World Weather Attribution (WWA) protocol** (Philip et al., 2020):

- Stationary GEV fitted to annual block maxima by maximum likelihood
- Non-stationary GEV with μ(t) = μ₀ + α·GMST(t), σ and ξ held stationary
- GMST covariate: NASA GISTEMP v4 (anomalies relative to 1951–1980)
- Attribution metrics: Probability Ratio (PR) and Fraction Attributable Risk (FAR)
- Uncertainty: 500-sample paired bootstrap on (temperature, GMST) resamples

Each analysis includes a `docs/` folder with mathematical derivations and a `references/` folder with key papers.

## Data

Data is **not included** in this repository. Each analysis notebook documents its sources and how to retrieve them. Common sources:

- **Station observations:** NOAA ASOS via [Iowa State Mesonet](https://mesonet.agron.iastate.edu/)
- **Forecast data:** NOAA GFS via [Herbie](https://herbie.readthedocs.io/)
- **GMST:** [NASA GISTEMP v4](https://data.giss.nasa.gov/gistemp/)

## References

Philip, S. et al. (2020). A protocol for probabilistic extreme event attribution analyses. *Advances in Statistical Climatology, Meteorology and Oceanography*, 6, 177–203.

Stott, P.A., Stone, D.A., & Allen, M.R. (2004). Human contribution to the European heatwave of 2003. *Nature*, 432, 610–614.

Coles, S. (2001). *An Introduction to Statistical Modeling of Extreme Values*. Springer.

## Requirements

```bash
pip install -r requirements.txt
```