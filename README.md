# JAT Pb(II) image-analysis repository

中文简介：本仓库汇集 JAT 图像分析小程序、Pb(II) 校准与加标水样数据、代表性原始图像，以及检测装置的 SolidWorks 模型。仓库用于公开研究材料和支持结果复现。

This repository contains the JAT image-analysis mini program together with calibration data, spiked-water measurements, representative images, and device design files used for Pb(II) analysis.

## Repository contents

| Path | Contents |
| --- | --- |
| `01_Calibration_Data/` | Original calibration workbook and a machine-readable CSV export |
| `02_Spiked_Water_Sample_Data/` | Original spiked-water workbook and two machine-readable CSV exports |
| `03_Representative_Raw_Images/` | Representative JPEG images for 405 nm and 520 nm measurements |
| `04_Code/` | Canonical uni-app source for the WeChat mini program |
| `05_Device_Files/` | SolidWorks part files for the upper and lower device assemblies |

Each directory contains a README with format-specific notes.

## Run the mini program

The canonical application source is the uni-app project in `04_Code/`.

1. Install HBuilderX with uni-app support and WeChat Developer Tools.
2. Import `04_Code/` as a uni-app project.
3. Set your own WeChat mini-program AppID under `mp-weixin.appid` in `04_Code/manifest.json`.
4. In HBuilderX, run the project to WeChat Developer Tools.
5. Select an image, define experimental and control regions, read RGB values, fit a standard curve, and calculate the sample concentration.

The original HBuilderX and WeChat base-library versions were not recorded. The manifest specifies Vue 3 and uni-app compiler version 3. Record tested tool versions before a formal archived release.

## Data formats and provenance

The `.xlsx` workbooks are the preserved source records. The CSV files are normalized exports for analysis in R, Python, MATLAB, and other tools. CSV transformations are documented in the data-directory READMEs and retain source sheet and row references.

The CSV exports standardize units in ASCII column names (`umol` and `uM`), fill merged sample labels down to individual records, and calculate transparent replicate summaries where the source table did not provide separate numeric columns. Original measurements are not overwritten.

## Application calculation

The app reads mean RGB values from experimental and control image regions and forms channel-specific ratios. Standard points are fitted with a linear model:

```text
ratio = k × concentration + b
```

For an unknown sample, the concentration is calculated as:

```text
concentration = (ratio - b) / k
```

The app stores images, calibration settings, and result history locally through the mini-program storage APIs. The retained application pages do not make network or cloud-database requests.

## Device files

The current device release contains native `.SLDPRT` files. It does not yet include STL/3MF print files, STEP exchange files, tolerances, materials, or validated printing parameters. See `05_Device_Files/README.md` before fabrication.

## Citation

Use `CITATION.cff` as the machine-readable citation template. Replace the contributor placeholder and add the associated paper DOI and repository archive DOI before publication.

## License

No open-source or open-data license has been selected by the rights holders. The current `LICENSE` therefore reserves all rights. Replace it only after the authors agree on licenses for the code, data/images, and hardware files. Third-party platforms remain subject to their own terms.
