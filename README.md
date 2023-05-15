# PyPSA-CEE

## Introduction

`PyPSA-CEE` is a dataset that can be used with the `PyPSA` open energy modelling framework to analyze the mechanics of the CEE region's power system. The model was created by Ember and described in a [briefing on CEE wind and solar targets](https://ember-climate.org/insights/research/in-it-together-cee-power-system/).

## Citation and license

The recommended citation is:

`Ember. (2023). PyPSA-CEE - An open energy model of the CEE power sector based on the PyPSA framework. https://github.com/ember-climate/pypsa-cee`

`PyPSA-CEE` is based on the [PyPSA](https://pypsa.readthedocs.io/en/latest/index.html) open energy modelling framework used by many research institutions around the world. Whenever using `PyPSA-CEE`, please also give credit to the authors of `PyPSA` following the guidelines [described in the PyPSA documentation](https://pypsa.readthedocs.io/en/latest/citing.html).

[Similarly to PyPSA](https://pypsa.readthedocs.io/en/latest/introduction.html?highlight=license#licence), `PyPSA-CEE` is distributed under the [MIT license](https://github.com/instrat-pl/pypsa-pl/blob/main/LICENSE).

## Setup

To use the model you will need to have a basic understanding of `Python` and `conda`, an LP solver and a computer with enough memory (16 GB RAM). The number of nodes in the model was kept at a minimum to make it possible to run with the open solver [HiGHS](https://www.maths.ed.ac.uk/hall/HiGHS/).

Before you start working with `PyPSA-CEE`, please follow the [installation guidelines for PyPSA](https://pypsa.readthedocs.io/en/latest/installation.html).

The basic installation steps can be summarized as follows:

1. Clone the `PyPSA-CEE` repository
2. Create a new `conda` environment
3. Install an LP solver (make sure it is available in the `Python` terminal)
4. Install the `PyPSA` package through `conda`

## Getting started

To make the model as accessible as possible `PyPSA-CEE` is contained in just one excel sheet (per scenario) and one Python file. The excel sheet contains all the input data, and the Python file is used to import them into the model, run the optimization and analyse the results.

After you complete the installation procedue, it is quite easy to get first results:

1. Open the `pypsa-cee.ipynb` file script in the `pypsa-cee` folder using your IDE.
2. Set your solver name in cell 7: `network.optimize(solver_name='highs')`. 
3. Save the file and run it (if an error pops up, make sure the terminal points to the correct directory and Python can see your solver).
4. Once the optimization is finished, you can view selected results in the bottom of the Jupyter notebook file.

The notebook shows only basic outputs like electricity generation, but there are several examples of processing results in the `PyPSA` documentation - such as plotting storage discharge profiles, SRMC, emissions, line loading etc.

## Model design

Please see [Ember's briefing on CEE wind and solar targets](https://ember-climate.org/insights/research/in-it-together-cee-power-system/) for a full explanation of the assumptions behind the modelled scenarios. An excerpt is provided below.

The two scenarios provided with the model use our own 2030 capacity assumptions and demand expectations from ENTSO-E's TYNDP  (Distributed Energy pathway). The capacity assumptions come from a variety of sources - cross-checked gas and coal unit databases from [Global Energy Monitor](https://globalenergymonitor.org/projects/global-gas-plant-tracker/) and [Beyond Fossil Fuels](https://beyondfossilfuels.org/coal/) for the existing fossil fuel fleet, national coal/nuclear phaseout announcements. The unit capacities for the existing fleet were aggregated and checked against Ember's data, IRENA, SolarPower Europe, WindEurope and others. Planned/in construction units were included. Wind and solar capacities come from a variety of sources, as described in detail in Ember's CEE report. Renewable capacities besides wind and solar were sources from NECPs and the existing fleet. 

The model is run for each hour of the year 2030, with 29 nodes representing representing all EU countries except Luxembourg, Malta and Cyprus, as well as the United Kingdom, Norway, Switzerland, Turkey and Russia. All existing interconnections were represented, with their capacities based on 2021 NTCs as well as ENTSO-E's 2025 reference grid. 

The input data includes weather and demand profiles from ENTSO-E's Pan-European Climate Database (PECD) for the most critical climatic year: 2008 (the baseline year). Other climate years from PECD can be used by changing the profiles manually in the Excel sheet.

Dispatching is optimised based on short-run marginal costs. 2030 price forecasts were based on the latest available futures contracts, except for lignite costs that were sourced from IEEFA and adapted according to the unit’s efficiency. Running costs for non-fossil fuel generators were set to represent their dispatching priority in the merit order. CHP operation is optimized only within boundaries based on average 2021 hourly generation for CHP units in the region. A minimum load of 40% for nuclear plants was assumed based on ENTSO-E’s ERAA input data. No ramp up/down limits were introduced due to their impact on model solving times, but these can be added in future works based on PyPSA-CEE.

Please note that `PyPSA-CEE` is not configured as a capacity expansion model (CEM), but rather a merit-order style dispatch model. The installed capacity for different technologies is provided manually as part of the scenario design. This decision was made to ensure that the implemented scenarios are realistic and aligned with national studies, industry forecasts, the existing potentials and the possible expansion within the limited 2030 timeframe. Running the model as a CEM means the capacity additions are set by the optimizer and do not always correspond to reality. It is possible to set the `capex` values for each technology and convert `PyPSA-CEE` to a CEM (an example is provided in the `gen_gas_peak` tab).

## Changing input parameters

To change the scenario, access the Excel sheet `pypsa-ember-cee-baseline-scenario-2030.xlsx`.

A few categories of tabs can be seen:

- `load` - demand profiles for each country.
- `pv`, `wind`, `wind_offshore`, `chp`,`ror`,`inflow`  - solar, wind, hydro and chp generation profiles for all hours of the year.
- `lines`, `links`, `buses`, `cbf` - technical components of the model - the definition of the nodes, lines and interconnectors
- `prices` - price assumptions
- `gen_XXX`, `st_XXX` - capacity assumptions for each generator and storage technology. Most parameters should be self-explanatory, the `p_nom` parameter is the installed capacity, for more explanation please refer to the `PyPSA` components documentation

## Further improvements

Many improvements can be made in the `PyPSA-CEE` model, some of them include:

- Heating - including heat demand in the model. This would make the CHP generation profile more realistic as currently it is independent from climatic assumptions.
- Hydrogen - including electrolyzers in the model, which would make it easier to analyse the potential of using curtailed energy for electrolysis. Implementing electrolyzers within the model was impossible due to the use of ENTSO-E aggregated demand profiles.
- Thermal unit flexibility - adding ramp-up/down and minimum utilization constraints for thermal units. This was removed from the model due to the significant increase in model solving times, but it is possible to implement these constraints in `PyPSA`.
- Unit maintenance schedule - the model does not include planned and unplanned outages of generation units, which could mean that the availability of especially older plants is higher than in reality. A maintenance schedule and a random outage algorithm could be included to represent the actual working conditions of generators.

You are welcome to expand and modify `PyPSA-CEE` according to your needs (honoring the license - see `Citiation and license` section). You can also submit pull requests onto the `PyPSA-CEE` repository, but due to a lack of resources we cannot ensure we will be able to review and approve those.

## References

Please look at the Ember briefing cited in the `Introduction` for a detailed list of sources and a description of the methodology, the scenario design assumptions, price parameters etc.

The most significant model references are listed below:

- PyPSA: T. Brown, J. Hörsch, D. Schlachtberger, PyPSA: Python for Power System Analysis, 2018, Journal of Open Research Software, 6(1), arXiv:1707.09913, DOI:10.5334/jors.188
- PyPSA-ENTSO-E: Matteo de Felice, Simulating European power systems using open tools and data, 2023, https://github.com/matteodefelice/pypsa-entsoe
- ENTSO-E Ten-year Network Development Plan (TYNDP) 2022: https://2022.entsos-tyndp-scenarios.eu/download/
- ENTSO-E European Resource Adequacy Assessment (ERAA) 2021: https://www.entsoe.eu/outlooks/eraa/2021/eraa-downloads/index.html
- PyPSA-UK: Ember. (2022). PyPSA-UK - An open energy model of the UK power sector based on the PyPSA framework. https://github.com/ember-climate/pypsa-uk
- PyPSA-PL: Czyżak, P., Mańko, M., Sikorski, M., Stępień, K., Wieczorek, B. (2021). PyPSA-PL - An open energy model of the Polish power sector based on the PyPSA framework. Instrat. https://github.com/instrat-pl/pypsa-pl

## Release notes

- 0.1.0 - initial release of PyPSA-CEE model with two 2030 scenarios described in [Ember's briefing on CEE wind and solar targets](https://ember-climate.org/insights/research/in-it-together-cee-power-system/).
