# Data Dictionary — 2024 Electricity Disconnection EDA

**Source of definitions:** `90819finalproj.ipynb` (final analysis notebook).  
**Underlying source:** Energy & Policy Institute's EIA Form 112 workbook, *EIA Form 112 Data (Last Upd. 6_10_2026)*.  
**Unit of observation:** One utility in 2024.  
**Analysis population:** 1,122 electric utilities: 169 investor-owned, 444 municipal, and 509 cooperative utilities.

## 1. Cleaned analysis dataset (`cleaned_df.csv`)

The notebook exports `cleaned_df.csv` in **Task 6**, *before* the rate-validation and outlier-analysis columns are added. Consequently, the exported CSV has **six columns**, described below. Its filename is the notebook's literal output name; the repository may place it in `data/cleaned/`.

| Variable | Type / unit | Meaning | Source and treatment |
|---|---|---|---|
| `state` | Text; state abbreviation | State associated with the utility. | Renamed from IOU `State`; retained as `state` in municipal and cooperative worksheets. |
| `utility_name` | Text | Name of the reporting utility. | Renamed from IOU `Utility`; retained from public-power worksheets. |
| `customer_count` | Numeric; customers | Customer count used as the denominator for the disconnection rate. | Renamed from IOU `Number of Customers`; retained from public-power worksheets. May be noninteger where the source uses an average customer count. |
| `shutoffs` | Numeric; disconnection events | Total reported 2024 service disconnections for the utility. | Renamed from IOU `Total 2024 Disconnections`; retained from public-power worksheets. Events need not represent distinct customers. |
| `disconnection_rate` | Numeric; decimal proportion | Number of disconnection events divided by customer count. For example, `0.1232` corresponds to **12.32 disconnection events per 100 customers**. | IOU `2024 Disconnection Rate` is divided by 100 to convert from percentage points to a proportion. Municipal and cooperative `shutoff_rate` is renamed without rescaling. |
| `producer_type` | Ordered categorical label | Utility ownership category used for grouping and comparison. | Assigned in Task 4. Valid values: `Investor-Owned`, `Municipal`, `Cooperative`, in that order. |

**Interpretation note:** `disconnection_rate` is a rate of events per customer exposure, **not necessarily the percentage of unique customers disconnected**. Multiple disconnections of the same customer can contribute multiple events.

## 2. Additional columns created in the in-memory `df` during the EDA

These columns are created **after** `cleaned_df.csv` is exported, so they are **not included in that CSV**.

| Variable | Type / unit | Meaning | Calculation / notebook task |
|---|---|---|---|
| `calculated_rate` | Numeric; decimal proportion | Independently reconstructed disconnection rate. | `shutoffs / customer_count`; Task 7. |
| `rate_difference` | Numeric; proportion-point difference | Difference between the harmonized source rate and reconstructed rate. | `disconnection_rate - calculated_rate`; Task 7. |
| `absolute_rate_difference` | Numeric; nonnegative proportion-point difference | Absolute magnitude of the rate discrepancy, useful for identifying records with the largest disagreement. | `abs(rate_difference)`; Task 7. |
| `lower_bound` | Numeric; decimal proportion | Lower threshold for a potential outlier, computed within ownership type. | `q1_rate - 1.5 × IQR`; Task 9. |
| `upper_bound` | Numeric; decimal proportion | Upper threshold for a potential outlier, computed within ownership type. | `q3_rate + 1.5 × IQR`; Task 9. |
| `potential_outlier` | Boolean | Whether a utility's disconnection rate falls below its ownership group's lower threshold or above its upper threshold. | `(disconnection_rate < lower_bound) OR (disconnection_rate > upper_bound)`; Task 9. A flag is not proof of erroneous data. |

## 3. Ownership-level summary variables (`group_summary`)

`group_summary` is indexed by `producer_type` and is built in **Task 8**, with `aggregate_rate` added in **Task 13**. All rate measures here are stored as **decimal proportions**, rather than percentages.

| Variable | Unit | Meaning / computation |
|---|---|---|
| `utility_count` | Utilities | Count of utilities in the ownership category (`utility_name` count). |
| `total_customers` | Customers | Sum of `customer_count` across utilities in the category. |
| `total_disconnections` | Events | Sum of `shutoffs` across utilities in the category. |
| `mean_rate` | Proportion | Arithmetic mean of utility-level `disconnection_rate` (each utility weighted equally). |
| `median_rate` | Proportion | Median utility-level `disconnection_rate`. |
| `std_rate` | Proportion | Sample standard deviation of utility-level rates. |
| `min_rate` | Proportion | Minimum utility-level rate. |
| `max_rate` | Proportion | Maximum utility-level rate. |
| `q1_rate` | Proportion | 25th percentile (first quartile) of utility-level rates. |
| `q3_rate` | Proportion | 75th percentile (third quartile) of utility-level rates. |
| `IQR` | Proportion | Interquartile range: `q3_rate - q1_rate`. |
| `aggregate_rate` | Proportion | Customer-weighted aggregate event rate: `total_disconnections / total_customers`. This is distinct from the equally weighted `mean_rate`. |

## 4. Final EDA summary table (`final_summary`)

Task 15 creates a presentation table from `group_summary`. **Unlike `group_summary`, its rate columns have been multiplied by 100**. Its index remains `producer_type`.

| Display column | Unit | Corresponding source variable |
|---|---|---|
| `Utilities` | Utilities | `utility_count` |
| `Total Customers` | Customers | `total_customers` |
| `Total Disconnections` | Events | `total_disconnections` |
| `Mean Rate (%)` | Events per 100 customers | `mean_rate × 100` |
| `Median Rate (%)` | Events per 100 customers | `median_rate × 100` |
| `Q1 (%)` | Events per 100 customers | `q1_rate × 100` |
| `Q3 (%)` | Events per 100 customers | `q3_rate × 100` |
| `IQR (percentage points)` | Percentage points | `IQR × 100` |
| `Aggregate Rate (%)` | Events per 100 customers | `aggregate_rate × 100` |

The notebook labels these columns with `%` for conventional presentation; for interpretation, the rate values denote **disconnection events per 100 customers**.

## 5. Other derived analysis objects

| Object / variable | Meaning |
|---|---|
| `producer_order` | Ordered list of ownership labels for tables and plots. |
| `quartiles` | Ownership-specific first and third quartiles, `q1_rate` and `q3_rate`, stored as proportions. |
| `outlier_limits` | Ownership-specific quartiles, IQRs, and lower/upper outlier thresholds. |
| `outliers` | Subset of `df` for which `potential_outlier` is true. |
| `outliers_by_type` | Count of potential outliers by ownership category. |
| `ownership_comparison` | Task 10 presentation table with utility counts, means, medians, quartiles, and IQRs; rates converted to percentages. |
| `mean_median` | Ownership-specific mean, median, and `mean_minus_median` difference; stored as proportions, converted to percentages for printed output. |
| `size_summary` | Ownership-specific customer-count statistics: count, mean, median, minimum, and maximum. |
| `state_counts` | Cross-tabulation of utility counts by state and ownership type. |
| `states_all_three` | States represented by all three ownership categories. |
| `state_compare` | Utilities belonging to those states. |
| `state_medians` | Median disconnection rate by state and ownership type, stored as a proportion. |
| `comparison` | Long-format table for comparing `Mean Utility Rate` with `Aggregate Customer-Weighted Rate` in the bar chart. Its `rate` field is a proportion. |
| `continuous_vars` | Three numeric analysis fields: `customer_count`, `shutoffs`, `disconnection_rate`. |
| `correlation_matrix` | Pearson correlation coefficients among the three continuous fields (unitless, ranging from −1 to 1). |
| `size_rate_corr` | Overall Pearson correlation between customer count and disconnection rate. |

## 6. Data preparation and quality checks

The notebook reads the `2024-IOUs`, `2024-MUNIS`, and `2024-COOPS` worksheets, renames their columns to a common schema, standardizes IOU rates to decimal proportions, assigns ownership labels, and concatenates the records. Task 6 checks missingness, possible duplicate utility identifiers (`state`, `utility_name`, `producer_type`), numeric conversion, nonpositive customer counts, negative disconnections or rates, and rates above 100%. Task 7 independently recalculates rates; Task 9 flags potential outliers **without deleting them**.

**Scope:** The dictionary describes fields used or created by the EDA Jupyter notebook. It is **not** a complete dictionary for every column in the original EIA/EPI workbook. The source workbook contains additional fields that this analysis does not retain.

**Source context:** [Energy & Policy Institute — New nationwide data reveals utility specific disconnection information](https://energyandpolicy.org/new-nationwide-data-reveals-utility-specific-disconnection-information/).
