# DP2-Commissioning Notebooks

This directory contains tutorial notebooks from the [Rubin Observatory tutorial-notebooks repository](https://github.com/lsst/tutorial-notebooks) related to the LSSTCam commissioning and Science Validation (SV) surveys. These notebooks were originally designed to run on the Rubin Science Platform (RSP) but can be adapted to run locally.

---

## Notebooks

### 101 — LSSTCam Visits Database (`101_lsstcam_visits_database.ipynb`)

**Last verified:** 2026-01-28 | **LSST Science Pipelines:** v29.2.0

**Learning objective:** How to query and retrieve data from the commissioning visits database file.

This notebook demonstrates how to explore the SQL-formatted table of LSSTCam commissioning visits contained in the file `lsstcam_20250930.db`. This static database file is a snapshot of the Science Validation (SV) survey as of 30 September 2025, aggregated from the Consolidated Database (ConsDB) with per-visit summary values from summit quicklook processing.

**Key content:**
- Connect to the SQLite database using `sqlite3` and explore its schema (a single `observations` table with 217 columns)
- Understand key columns: `fieldRA`, `fieldDec`, `band`, `airmass`, `exp_midpt_mjd`, `observation_reason`, `target_name`, `s_ra`, `s_dec`, `seeing_zenith_500nm`, `sky_bg`, `sky_source`, `t_eff`, `visit_id`
- Query visits by filter band, airmass, and observation type
- Visualize the survey footprint on sky
- Reproduce tables and figures from the [SV survey summary webpage](https://survey-strategy.lsst.io/progress/sv_status/sv_20250930.html)

**Context:** Science imaging with LSSTCam began on 04 April 2025. The SV area follows the ecliptic plane and includes 4 Deep Drilling Fields (DDFs). The database contains 21,647 commissioning visits (excluding bad visits but spanning a wide range of data quality).

**Required packages:** `sqlite3` (stdlib), `rubin_sim` (MAF module), `numpy`, `pandas`, `matplotlib`, `astropy`, `tabulate`, `lsst.utils.plotting`

**Data file:** `lsstcam_20250930.db` (included in this directory)

---

### 102 — Rubin Schedule Viewer (`102_rubin_schedule_viewer.ipynb`)

**Last verified:** 2026-02-18 | **LSST Science Pipelines:** v29.2.0

**Learning objective:** How to programmatically query the Rubin Schedule Viewer.

This notebook demonstrates how to interact with the **Rubin Schedule Viewer**, a public web service hosted at the US Data Facility (USDF): [https://usdf-rsp.slac.stanford.edu/obsloctap/static/viewer.html](https://usdf-rsp.slac.stanford.edu/obsloctap/static/viewer.html). The service provides both a graphical user interface and a programmatic API following the **IVOA ObsLocTAP** (Observation Locator Table Access Protocol) standard.

**Key content:**
- Check the health of the Schedule Viewer service
- Query the service metadata (header, description, capabilities) via the ObsLocTAP API using `requests`
- Understand the `ivoa.obsplan` table schema (28 columns), including `t_min`, `t_max`, `s_ra`, `s_dec`, `em_min`, `em_max`, `target_name`, `execution_status`
- Retrieve both historical observations and forecasted future observations
- Convert wavelength columns (`em_min`, `em_max`) to LSST filter/band names
- Visualize scheduled observations interactively with `bokeh` and `holoviews`

**Required packages:** `requests`, `astropy`, `bokeh`, `holoviews`

**Note:** This notebook requires internet access to reach the USDF Schedule Viewer service. No local data file is needed.

---

## Local Setup

These notebooks were originally designed for the RSP environment. The following adaptations are needed to run them locally.

### Environment variables

```python
import os
# Path to rubin_sim auxiliary data (downloaded with rs_download_data)
os.environ['RUBIN_SIM_DATA_DIR'] = os.path.expanduser('~/rubin_sim_data')

# Scratch directory for output files
os.environ['SCRATCH_DIR'] = os.path.expanduser('~/Desktop/RubinLSSTDP1Data/scratch')
```

### Required installations

```bash
conda activate conda_py313

# Core packages
pip install rubin-sim rubin-scheduler lsst-utils
pip install requests bokeh holoviews tabulate

# Download rubin_sim auxiliary data
scheduler_download_data
rs_download_data --update --dirs maf,throughputs,skybrightness
```

### `lsst.utils.plotting` workaround

If `lsst-utils` is not available via pip, replace the import in notebook 101 with:

```python
def get_multiband_plot_colors():
    return {'u': '#56b4e9', 'g': '#008060', 'r': '#ff4000',
            'i': '#850000', 'z': '#6600cc', 'y': '#000000'}

def get_multiband_plot_linestyles():
    return {'u': '-', 'g': '--', 'r': '-.',
            'i': ':', 'z': '-', 'y': '--'}
```

---

## Data Files

| File | Description |
|------|-------------|
| `lsstcam_20250930.db` | SQLite database of LSSTCam commissioning visits (snapshot: 30 Sep 2025, 21,647 visits) |

---

## References

- [Rubin tutorial-notebooks GitHub repository](https://github.com/lsst/tutorial-notebooks)
- [Science Validation survey summary (2025-09-30)](https://survey-strategy.lsst.io/progress/sv_status/sv_20250930.html)
- [rubin_sim documentation](https://rubin-sim.lsst.io)
- [rubin_scheduler documentation](https://rubin-scheduler.lsst.io)
- [Rubin Schedule Viewer (USDF)](https://usdf-rsp.slac.stanford.edu/obsloctap/static/viewer.html)
- [IVOA ObsLocTAP standard](https://www.ivoa.net/documents/ObsLocTAP/)
