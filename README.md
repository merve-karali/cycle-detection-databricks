# Brake Cycle Detection Pipeline on Databricks

End-to-end cycle detection pipeline for brake testbench sensor data using the Databricks ML stack.

Detects **brake cycles** (braking operation segments) in time-series sensor data from Pressure Control Unit (PCU) testbenches, using Feature Engineering, MLflow, Model Registry, and Genie Spaces.

## Quick Start

### Prerequisites

- Databricks workspace with **Unity Catalog** enabled
- Cluster running **Databricks Runtime 15.4 LTS ML** or later (single node is sufficient)
- Permissions: `CREATE CATALOG`, `CREATE SCHEMA`, `USE CATALOG` on your catalog

### Setup

1. **Configure Git integration** in your Databricks workspace by following [Configure Git integration for Git folders](https://docs.databricks.com/en/repos/repos-setup.html)

2. **Clone this repo** into your workspace using [Git folders](https://docs.databricks.com/repos/git-operations-with-repos.html):
   - In the sidebar, click **Workspace** and navigate to your user folder
   - Click the dropdown arrow next to your name → **Create** → **Git folder**
   - Paste the repository URL, select **GitHub** as the provider, and click **Create Git folder**

3. **Edit `00_Config`** — update these three variables:

   ```python
   CATALOG = "your_catalog"
   SCHEMA = "cycle_detection"
   EXPERIMENT_NAME = "/Users/you@company.com/cycle-detection"
   ```

4. **Attach a cluster** (ML Runtime 15.4 LTS+) and run notebooks in order: `00` → `01` → `02` → `03` → `04`

> **Recommended**: Run the full pipeline with synthetic data first to verify that all permissions, cluster configuration, and Unity Catalog access are working correctly. Once everything runs end-to-end without errors, switch to your real data (see [Using Real Data](#using-real-data) below).

## Notebooks

| # | Notebook | What it does |
|---|----------|-------------|
| 00 | **Config** | Shared constants (catalog, schema, table names). **Edit this first.** |
| 01 | **Generate Synthetic Data** | Creates realistic brake testbench sensor data with ground truth labels. Replace with real data in production. |
| 02 | **Feature Engineering** | Computes windowed time-series features (rolling pressure stats, gradients, valve switches) and registers a Feature Table in Unity Catalog. |
| 03 | **Model Training & Inference** | Trains a LightGBM model, logs with `fe.log_model()` for feature lineage, sets production alias, runs batch inference on held-out test data. |
| 04 | **Genie Space Setup** | Extracts cycle boundaries from predictions and creates a Genie Space for natural language exploration. |

## Databricks Features Demonstrated

- **Feature Engineering Client** — `fe.create_table()`, `fe.create_training_set()`, `fe.log_model()`, `fe.score_batch()`
- **MLflow** — experiment tracking, model logging, dataset lineage
- **Unity Catalog Model Registry** — model versioning, production alias
- **Genie Spaces** — natural language SQL interface for business users

## Data Schema

The synthetic (or real) sensor data has this structure:

| Column | Type | Description |
|--------|------|-------------|
| `timestamp` | timestamp | Measurement time (100ms resolution) |
| `testbench_id` | string | Testbench identifier (e.g. TB-PCU-001) |
| `test_run_id` | string | Unique test run identifier |
| `pressure_bar` | double | Brake pressure in bar |
| `valve_state` | int | Magnet valve state (0=closed, 1=open) |
| `temperature_c` | double | Component temperature in Celsius |
| `flow_rate` | double | Hydraulic flow rate (l/min) |
| `cycle_status` | int | Ground truth — 1 if inside a brake cycle, 0 if idle |