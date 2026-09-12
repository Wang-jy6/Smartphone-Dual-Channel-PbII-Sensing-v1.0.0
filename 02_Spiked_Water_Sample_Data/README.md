# Spiked-water sample data

## Files

- `02_Spiked_Water_Sample_Data.xlsx`: preserved source workbook.
- `spiked_water_response.csv`: five response measurements for each sample and Pb(II) addition level.
- `spiked_water_measured_concentration.csv`: five measured concentration values for each sample and addition level.

The samples are lake water, river water, domestic sewage, and industrial wastewater. Each sample has measurements at Pb(II) additions of 0.4, 10, and 40 μM.

## Response CSV

`response_value_1` through `response_value_5` preserve the source observations. `response_mean` and `response_sd` are calculated from those five observations, using the sample standard deviation (`n - 1`).

## Measured-concentration CSV

`measured_value_1_uM` through `measured_value_5_uM` preserve the five reported concentration estimates. `measured_mean_uM` and `measured_sd_uM` are independently calculated from the five estimates. They agree with the workbook's two-decimal summary within its rounding precision.

Both CSV files fill the workbook's merged/blank sample labels down to every record, normalize the truncated source label `Industrial wastewate` to `Industrial wastewater`, and include `source_sheet` and `source_row` for traceability. The original workbook remains unchanged.
