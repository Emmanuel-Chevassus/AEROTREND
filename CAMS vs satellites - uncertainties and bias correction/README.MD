Methane satellite column comparison: AIRS3STD versus CAMS EAC4 at Mace Head
This repository contains a Python/Jupyter workflow for comparing satellite-derived methane total columns from AIRS3STD with CAMS EAC4 total-column methane at the Mace Head atmospheric research site, Ireland. The workflow harmonises units, aggregates both products to monthly resolution, performs trend and distribution diagnostics, quantifies AIRS–CAMS agreement, propagates prescribed uncertainty through Monte Carlo sampling, and compares simple bias-correction approaches.
The analysis is designed for column-integrated methane intercomparison, not for surface methane validation. AIRS and CAMS do not represent exactly the same observing system: AIRS provides satellite retrievals of atmospheric methane column information, whereas CAMS provides a model/reanalysis total-column methane field. The comparison should therefore be interpreted as a consistency assessment between two column-scale products after temporal aggregation and unit conversion.
Repository contents
```text
methane_satellite_vs_CAMS_github_clean.ipynb
README.md
```
The notebook contains three main analysis blocks:
Monthly harmonisation, AIRS unit conversion, STL decomposition, and quantile-regression trend diagnostics
AIRS versus CAMS comparison with Monte Carlo uncertainty propagation and Taylor statistics
Bias-correction comparison using quantile mapping and ratio/additive adjustment methods
Scientific objective
The workflow addresses four questions:
Are the monthly AIRS and CAMS methane total-column time series consistent in magnitude and temporal variability after converting AIRS from molecules cm<sup>-2</sup> to mg m<sup>-2</sup>?
Do the two products show coherent low-frequency behaviour after seasonal-trend decomposition?
How large are the CAMS-minus-AIRS bias, error metrics, and Taylor-diagram statistics when AIRS is treated as the observational reference?
Do simple post-processing methods reduce the CAMS bias relative to AIRS without destroying temporal correlation?
Input data
The notebook expects two CSV files:
```text
D:/Time Series/AIRS3STD.006_mace_head.csv
D:/Time Series/CAMS_single_mace_head_main.csv
```
These paths are hard-coded near the top of the first notebook cell and should be changed before running the notebook on another machine.
Required AIRS input
The AIRS CSV must contain:
Column	Meaning
`date`	Sampling date or datetime
`TotCH4_A`	AIRS methane total column in molecules cm<sup>-2</sup>
The loader is written to handle AIRS-style files with metadata above the data table. It searches for the first line where the first column is `date`, then reads the file from that header row.
Required CAMS input
The CAMS CSV must contain:
Column	Meaning
`date`	Sampling date or datetime
`total_column_methane`	CAMS total-column methane in kg m<sup>-2</sup> or mg m<sup>-2</sup>, depending on the upstream extraction file
The current notebook column naming assumes the final harmonised output column is named:
```text
CAMS_total_column_methane_mg_m2
```
Check the CAMS input units before using the workflow. CAMS atmospheric composition variables are often stored in SI mass units, and the upstream extraction may already have converted them. A unit mismatch here would produce a physically meaningless comparison.
Unit conversion
AIRS methane total column is converted from molecules cm<sup>-2</sup> to mg m<sup>-2</sup> using:
```text
C_CH4 [mg m^-2] = N_CH4 [molecules cm^-2] × 10^4 × (M_CH4 / N_A) × 10^3
```
where:
Symbol	Value	Unit	Meaning
`N_CH4`	input	molecules cm<sup>-2</sup>	AIRS methane column
`M_CH4`	16.043	g mol<sup>-1</sup>	methane molar mass
`N_A`	6.02214076 × 10<sup>23</sup>	molecules mol<sup>-1</sup>	Avogadro constant
`10^4`	fixed	cm<sup>-2</sup> to m<sup>-2</sup>	area conversion
`10^3`	fixed	g to mg	mass conversion
The resulting multiplicative factor is approximately:
```text
2.6640 × 10^-16 mg m^-2 per molecule cm^-2
```
The factor is `10^4`, not `10^-4`, because a column reported per cm<sup>2</sup> must be scaled up to the equivalent number of molecules per m<sup>2</sup>. One square metre contains 10,000 square centimetres.
Temporal aggregation
Both products are aggregated to monthly means. The notebook uses a coverage threshold of 0.25 for calendar-based imputation in the first analysis block. Missing values are handled conservatively during pairwise AIRS–CAMS comparison by retaining only months with both AIRS and CAMS values.
The merged monthly dataset is written to:
```text
D:/PLOTS/CH4_comparison/ch4CAMSvsAIRSTD.csv
```
with columns equivalent to:
```text
date
AIRS_TotCH4_A_mg_m2
CAMS_total_column_methane_mg_m2
```
Methods implemented
1. STL decomposition
Each monthly methane series is decomposed using STL with the following base settings:
```text
period   = 12
seasonal = 7
trend    = 23
robust   = True
```
The decomposition is used to separate the seasonal cycle, low-frequency trend component, and residual variability. Z-scored STL trends are also compared between AIRS and CAMS to evaluate whether both products capture coherent interannual-scale variability after removing their mean and scale differences.
2. Quantile-regression trend diagnostics
The first analysis block computes quantile-regression slopes over a tau grid:
```text
τ = 0.05, 0.10, ..., 0.95
```
Key diagnostic quantiles are:
```text
τ = 0.25, 0.50, 0.75
```
Heteroskedasticity- and autocorrelation-consistent uncertainty is estimated using a HAC lag of 12 months. These diagnostics are intended to check whether trends differ between the lower, median, and upper parts of the methane-column distribution.
3. AIRS–CAMS pairwise agreement metrics
For all paired monthly values, CAMS is treated as the model/product being evaluated and AIRS as the reference. The notebook computes:
Metric	Meaning
`MB`	Mean bias, CAMS minus AIRS
`median_bias`	Median CAMS-minus-AIRS bias
`MGE`	Mean gross error
`NMB`	Normalised mean bias
`NMGE`	Normalised mean gross error
`RMSE`	Root-mean-square error
`FAC2`	Fraction of CAMS values within a factor of two of AIRS
`r`	Pearson correlation coefficient
`P-value`	Pearson correlation p-value
`median_ratio`	Median CAMS/AIRS ratio
Metrics are reported for all months and by meteorological season:
```text
DJF, MAM, JJA, SON
```
4. Monte Carlo uncertainty propagation
The uncertainty-aware comparison block applies prescribed relative monthly uncertainties:
```text
AIRS monthly relative uncertainty = 0.55 %
CAMS monthly relative uncertainty = 1.35 %
Number of Monte Carlo draws       = 20,000
Random seed                       = 42
```
For each draw, AIRS and CAMS monthly values are perturbed using Gaussian errors proportional to their monthly values. The workflow recomputes regression slope, intercept, Pearson correlation, mean bias, median bias, RMSE, MGE, median ratio, and p-value for each Monte Carlo realisation.
The Monte Carlo component is useful for showing whether the AIRS–CAMS mismatch is large relative to the assumed product uncertainties. The prescribed uncertainties should be treated as configurable assumptions, not universal product constants.
5. Taylor statistics
The notebook computes Taylor-diagram statistics relative to AIRS:
Quantity	Meaning
standard deviation	Absolute variability of each product
normalised standard deviation	Product standard deviation divided by AIRS standard deviation
correlation	Pearson correlation with AIRS
centred RMSE	RMSE after removing mean bias
normalised centred RMSE	Centred RMSE divided by AIRS standard deviation
The Taylor diagram should be interpreted carefully. A point close to the reference indicates similar variance, high correlation, and low centred RMSE, but it does not by itself diagnose absolute mean bias. Bias must be assessed using the bias and error metrics tables.
6. Bias-correction methods
The final notebook block compares raw CAMS against three simple bias-corrected alternatives:
Method	Description
Raw CAMS	Original CAMS monthly total-column methane
Quantile mapping	Maps the CAMS empirical distribution onto the AIRS empirical distribution
RAT-add	Additive mean adjustment: CAMS plus the AIRS-minus-CAMS mean difference
RAT-multi	Multiplicative ratio adjustment: CAMS multiplied by the AIRS/CAMS total-column ratio
These are diagnostic corrections, not mechanistic retrieval corrections. Quantile mapping can improve marginal distribution agreement while potentially altering extremes and physical interpretability. Ratio or additive adjustment preserves temporal structure more directly but cannot correct distribution-shape errors.
Output files
All default outputs are written under:
```text
D:/PLOTS/CH4_comparison
```
Harmonised monthly data and trend diagnostics
```text
ch4CAMSvsAIRSTD.csv
AIRS/trend_summary.csv
CAMS/trend_summary.csv
01_stl_decomposition.png
02_tau_grid.png
03_qr_three_quantiles.png
05_zscore_trend_combined.png
06_timeseries_scatter.tiff
```
Exact subfolder names depend on how the notebook creates per-series output directories during execution.
Final uncertainty-aware comparison
```text
08_timeseries_scatter_violin_trend_taylor_uncertaintyMC_final.tiff
08_timeseries_scatter_violin_trend_taylor_uncertaintyMC_final.png
08_model_metrics_table_uncertaintyMC_final.csv
08_model_metrics_table_uncertaintyMC_final.docx
08_taylor_statistics_uncertaintyMC_final.csv
08_uncertainty_monte_carlo_summary_final.csv
08_uncertainty_monte_carlo_draws_final.csv
```
Bias-correction comparison
Written under:
```text
D:/PLOTS/CH4_comparison/RAT_vs_quantilemapping
```
with outputs:
```text
RAT_vs_QQ_bias_correction_comparison.tiff
RAT_vs_QQ_bias_correction_comparison.png
RAT_vs_QQ_corrected_monthly_data.csv
RAT_vs_QQ_summary_metrics.csv
RAT_vs_QQ_summary_metrics.docx
```
Python requirements
The notebook uses the following Python packages:
```text
numpy
pandas
matplotlib
scipy
statsmodels
python-docx
```
A minimal Conda environment can be created with:
```bash
conda create -n methane-cams-airs python=3.11 numpy pandas matplotlib scipy statsmodels python-docx jupyter -y
conda activate methane-cams-airs
jupyter notebook
```
or with `pip`:
```bash
python -m venv .venv
.venv\Scripts\activate
pip install numpy pandas matplotlib scipy statsmodels python-docx jupyter
jupyter notebook
```
On Linux or macOS, activate the virtual environment with:
```bash
source .venv/bin/activate
```
How to run
Clone this repository.
Place the AIRS and CAMS CSV files on your machine.
Open `methane_satellite_vs_CAMS_github_clean.ipynb`.
Edit the hard-coded paths in the first cell:
```python
AIRS_CSV = Path(r"D:/Time Series/AIRS3STD.006_mace_head.csv")
CAMS_CSV = Path(r"D:/Time Series/CAMS_single_mace_head_main.csv")
OUT_ROOT = Path(r"D:/PLOTS/CH4_comparison")
```
Run the first cell to create the merged monthly AIRS–CAMS file and base trend diagnostics.
Run the second cell to generate the uncertainty-aware comparison figure, metrics, Taylor statistics, and Monte Carlo output tables.
Run the third cell to generate the quantile-mapping and ratio-adjustment bias-correction comparison.
Run the notebook cells in order. The second and third cells depend on the merged CSV produced by the first cell.
Reproducibility notes
The Monte Carlo section is reproducible because the random seed is fixed:
```text
RNG_SEED = 42
```
Figures are exported at 300 dpi in the later analysis blocks, with TIFF outputs using LZW compression. The notebook sets Matplotlib defaults internally, so figure style should be reproducible across machines apart from font availability and backend-specific rendering differences.
Important assumptions and limitations
AIRS is treated as the reference product, but this does not mean AIRS is error-free. The comparison is a product-consistency assessment.
The uncertainty values are user-prescribed assumptions. They are used for Monte Carlo propagation and should be revised if a different uncertainty model is justified.
Monthly aggregation suppresses synoptic variability. This is appropriate for broad product comparison but unsuitable for event-scale methane plume analysis.
Citation and acknowledgement
When using this workflow in a manuscript or report, cite the underlying AIRS and CAMS data products, the CAMS EAC4 reanalysis documentation, and the statistical methods used for STL decomposition, quantile regression, and Taylor-diagram evaluation. This repository provides analysis code only; it does not replace citation of the data providers or retrieval/reanalysis documentation.
