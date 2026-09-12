# JAT mini-program source

## Source of truth

The uni-app Vue 3 source in this directory is the canonical application implementation:

- `App.vue`, `main.js`, `manifest.json`, and `pages.json`: application entry and configuration
- `pages/`: image analysis, standard-curve fitting, concentration calculation, and history pages
- `common/`, `static/`, and `utils/`: shared styles, assets, and utilities

HBuilderX generates the native WeChat mini-program output under `unpackage/`. That generated directory is intentionally excluded from the repository.

## Requirements

- HBuilderX with uni-app support
- WeChat Developer Tools
- A WeChat mini-program AppID supplied by the user

The project uses Vue 3 and declares uni-app compiler version 3. Exact versions used for the original build were not recorded.

## Run

1. Open this directory in HBuilderX.
2. Enter your own AppID in `manifest.json` under `mp-weixin.appid`; the public repository intentionally leaves it blank.
3. Choose **Run > Run to Mini Program Simulator > WeChat Developer Tools**.
4. Grant camera or album access when prompted.

Generated output belongs in `unpackage/` and is excluded from Git.

## Data handling

The retained pages use local mini-program storage for RGB values, selection boxes, standard points, fitted coefficients, and history. They do not contain network requests or cloud-database operations. Do not add secrets directly to `manifest.json` or source files.
