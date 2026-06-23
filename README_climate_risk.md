Climate-risk extension to OnSSET

This branch adds an experimental climate-risk cost adjustment to the OnSSET electrification workflow. The aim is to allow settlement-level climate risk factors to influence least-cost technology selection by increasing technology-specific LCOE values where assets are exposed to climate hazards.

Summary of changes

The original OnSSET workflow is extended in two places:

onsset/onsset.py has been modified to accept settlement-level risk-factor columns and include them in the LCOE calculation.
The scenario notebook has been modified to merge risk-factor inputs, run risk and no-risk scenarios, and compare resulting technology choices and LCOE values.

The risk factors are applied as capital-cost-equivalent penalties, but they are not added directly to the reported upfront investment cost. Instead, the risk cost is annualised and added to the annual cost stream used to calculate LCOE. This means the risk adjustment affects technology choice through the LCOE calculation while keeping the standard investment-cost outputs interpretable as conventional technology investment.

Risk-factor columns

The model expects settlement-level risk-factor columns such as:

rf_total_SA_PV
rf_total_GRID_POLE
rf_total_GRID_TRANSFORMER
rf_total_MG_PV
rf_total_MG_WIND
rf_total_MG_PV_GENERATION
rf_total_MG_WIND_GENERATION

These are merged into the OnSSET settlement dataframe using the settlement id.

For standalone PV, the scenario notebook can distinguish between ground-mounted and rooftop assumptions. If the input risk file contains separate columns such as rf_total_SA_PV_ground and rf_total_SA_PV_roof, the notebook selects one of these and maps it to the generic column rf_total_SA_PV, which is then used by onsset.py.

Implementation in onsset.py

The Technology.get_lcoe() method has been extended with three optional penalty inputs:

line_penalty
transformer_penalty
generation_penalty

The transmission and distribution cost calculation has also been modified to separate line costs, transformer costs, connection costs, and mini-grid powerhouse costs. This allows risk factors to be applied only to the relevant exposed components. For example, grid line risk can be applied to line costs, while transformer risk can be applied to transformer and substation costs.

The resulting risk costs are annualised using a fixed annuity assumption and added to the annual cost stream used in the LCOE calculation.

Scenario structure

The scenario notebook is designed to run three comparable cases:

No-risk baseline
Risk-adjusted case with ground-mounted standalone PV
Risk-adjusted case with rooftop standalone PV

The main toggles are:

RUN_RISK = False  # no-risk baseline

or:

RUN_RISK = True
SA_PV_RISK_MODE = "ground"  # or "roof"

Outputs are saved using distinct filenames, for example:

SierraLeone_NoRisk_Results.csv
SierraLeone_Risk_SAPV_ground_Results.csv
SierraLeone_Risk_SAPV_roof_Results.csv
Post-processing outputs

The notebook includes post-processing to compare risk-adjusted scenarios against the no-risk baseline. The main outputs include:

mean and median LCOE change;
LCOE change in USD/kWh and percent;
number of settlements whose least-cost technology changes;
population affected by technology reassignment;
reassignment matrices showing flows between technologies, such as SA_PV → Grid or MG_PVHybrid → SA_PV;
LCOE-change maps with both full-range and 5th–95th percentile capped colour scales.

The main comparisons are:

ground-mounted SA_PV risk case vs no-risk baseline;
rooftop SA_PV risk case vs no-risk baseline.

The roof-vs-ground comparison is mainly used as a sensitivity or adaptation comparison.