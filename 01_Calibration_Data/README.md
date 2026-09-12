# Calibration data

## Files

- `01_Calibration_Data.xlsx`: preserved source workbook.
- `calibration_data.csv`: normalized, machine-readable export.

The workbook contains five replicate response measurements per calibration point in three blocks:

| Measurement mode | Wavelength | Calibration points |
| --- | ---: | ---: |
| Direct scattering | 405 nm | 6 |
| Indirect absorbance | 405 nm | 11 |
| Indirect absorbance | 520 nm | 10 |

## CSV columns

| Column | Meaning |
| --- | --- |
| `measurement_mode` | `direct_scattering` or `indirect_absorbance` |
| `wavelength_nm` | Measurement wavelength in nanometres |
| `pb_ii_amount_umol` | Pb(II) amount recorded by the source workbook, in μmol |
| `response_value_1` … `response_value_5` | Five recorded replicate responses |
| `response_mean` | Mean recorded in the source workbook |
| `response_sd` | Sample standard deviation recorded in the source workbook |
| `source_sheet`, `source_row` | Location of the source record in the workbook |

The exported mean and sample standard deviation were independently checked against the five replicates. Differences are below `5 × 10^-6`, consistent with rounding in the workbook.
