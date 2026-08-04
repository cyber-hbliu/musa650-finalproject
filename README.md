# Detecting urban decay and new development from Landsat in Philadelphia, Detroit and Atlanta, 2015 to 2025

I wanted to know whether a convolutional network trained on Landsat imagery could identify where a city is coming apart. Blight detection from satellites is a standing promise in planning analytics. Cities have limited demolition budgets and land banks with waiting lists, and a citywide map of where decline is accelerating would be useful to them.

The model works, in the narrow sense that it beats a spectral baseline on the decay class in two of three cities. Then I checked it against Philadelphia's own records, and the two disagree. Not weakly. The correlation between detected decay and recorded vacancy is negative at every spatial scale I tested, and it holds after controlling for the amount of building in each cell.


## Workflow

```mermaid
flowchart TB
  subgraph SOURCES["Data sources"]
    L["Landsat Collection 2 Level-2<br/>Planetary Computer STAC"]
    N["Annual NLCD Collection 1.2<br/>MRLC GeoServer WMS"]
    T["TIGER/Line 2024<br/>places and census tracts"]
    P["Philadelphia open data<br/>311, vacant buildings, permits"]
  end

  L --> C["Summer median composite<br/>2015 and 2025<br/>7 bands + NDVI + NDBI = 18 channels"]
  N --> R["Intensity rank per year<br/>open space 1 to high 4"]
  R --> M["Median 2015-2017 vs median 2023-2025<br/>persistence rule"]
  M --> Y["Three-class label<br/>0 no change / 1 new development / 2 decay"]
  T --> K["City limit mask<br/>EPSG:5070, 30 m grid"]
  K --> C
  K --> Y

  C --> S["Checkerboard spatial blocks<br/>256 px, fold 0 train / fold 1 test"]
  Y --> S
  S --> U["U-Net, 3 levels, 32/64/128<br/>weighted CE + Tversky<br/>40 epochs"]

  U --> E["Held-out evaluation<br/>average precision, F1, IoU"]
  C --> B["Spectral null baseline<br/>ΔNDBI and ΔNDVI"]
  B --> E
  U --> X["Frozen transfer to Detroit and Atlanta<br/>global vs per-city standardisation"]
  X --> E

  U --> V["Aggregate to census tracts<br/>and to 90 m to 2400 m grids"]
  P --> V
  V --> W["Spearman and partial Spearman<br/>controlling for built area"]

  E --> F["Figures and tables"]
  W --> F
```

