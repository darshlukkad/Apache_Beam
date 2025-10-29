
# Apache Beam: End‑to‑End Sales ETL (Notebook)

This repository contains a single Jupyter/Colab notebook that demonstrates an end‑to‑end **ETL pipeline using [Apache Beam]** on a small, in‑memory sales dataset. The notebook walks through building clean Beam pipelines—starting from reading raw CSV lines, through parsing & cleaning, mapping & filtering, windowing, partitioning, and finally **writing multiple outputs** (high‑value vs regular sales).

> **Why this notebook?**  
> If you're new to Beam, this is a compact, runnable tour of core Beam concepts on the **Direct/Interactive runner** without external infra. You can run it entirely on your laptop or open it in Colab.

---

## What the notebook does

1. **Installs and bootstraps Apache Beam**
   - Ensures `apache-beam` is available and configures the **Interactive Beam** environment for quick iteration.

2. **Creates a sample dataset**
   - Defines a small CSV (customers, amounts, timestamps) and writes it to `sales.csv` for subsequent steps.

3. **Reads and prints raw records with a simple pipeline**
   - Uses `ReadFromText(..., skip_header_lines=1)` to ingest the CSV lines.
   - Verifies the runner and pipeline basics by printing elements.

4. **Parses, cleans, and enriches records (ParDo & Composite Transform)**
   - Implements a `DoFn` to parse CSV text into structured dicts: `{'order_id','customer','amount','timestamp'}`.
   - Filters invalid rows (e.g., non‑positive amounts).
   - Demonstrates a **composite transform** (e.g., `CleanSalesData`) to encapsulate reusable logic.

5. **Applies Map and Filter transforms**
   - Adds a derived field, e.g. `amount_usd` via a `Map` with a (sample) FX rate.
   - Filters to “significant” sales using a configurable threshold.

6. **Partitions data into high‑value vs regular**
   - Uses `beam.Partition` with a `partition_fn` (e.g., `amount > 500` → high‑value).
   - Writes each partition to its own sink:
     - `output/high_value_sales-0000*`
     - `output/regular_sales-0000*`

7. **(Optional) Windowing over event time**
   - Demonstrates attaching event timestamps and windowing (e.g., **Fixed Windows** around 1 minute).
   - Shows how you’d aggregate (e.g., per customer per window) before writing or further processing.

8. **Writes outputs and inspects files**
   - Uses `WriteToText` sinks and lists the produced files in `output/`.

---

## Key Apache Beam concepts covered

- **PCollections** and the Beam **transform model** (`ParDo`, `Map`, `Filter`, `Partition`, `GroupByKey`/`CombinePerKey` if you add aggregations).
- **Composite transforms** to group related operations.
- **Event time & windowing** using `TimestampedValue` and `FixedWindows` (optional cell).
- **Multiple outputs** via partitioning and separate sinks.
- Running on the **Direct/Interactive runner** locally or in Colab.

---

## Repository contents

```
.
├── Apache_beam.ipynb   # The notebook (run top-to-bottom)
└── README.md           # This file
```

> The dataset is generated inside the notebook and written to `sales.csv` in the working directory.

---

## Requirements

- Python 3.9+ (3.10 recommended)
- JupyterLab/Notebook **or** Google Colab
- The notebook itself installs:
  - `apache-beam` (as a pip dependency)

---

## How to run

### Option A: Google Colab (recommended for first run)
1. Upload `Apache_beam.ipynb` to a new Colab session, or open it from your GitHub fork.
2. Run all cells **in order** (top → bottom). The first code cell installs `apache-beam`.

### Option B: Local Jupyter
1. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   ```
2. Install dependencies (Beam is installed directly in the notebook; you can also preinstall):
   ```bash
   pip install apache-beam jupyter
   ```
3. Launch Jupyter and open the notebook:
   ```bash
   jupyter notebook
   ```
4. Run all cells in order. The **Direct/Interactive runner** is used—no extra services required.

---

## Notebook structure (high level)

- **Setup**
  - Install/import Beam; enable interactive mode.
- **Data prep**
  - Create `sales.csv` from an in‑memory CSV string.
- **Basic I/O**
  - `ReadFromText` → print lines.
- **Parse & clean**
  - `ParDo` (`DoFn`) → dict records; filter bad rows; composite transform wrapper.
- **Enrich, map & filter**
  - Add USD amounts (`Map`); filter significant rows (`Filter`).
- **Partitioning**
  - Split high‑value vs regular with `beam.Partition` and write to separate sinks.
- **(Optional) Windowing**
  - Attach event timestamps; apply `FixedWindows`; (optionally) aggregate per customer.
- **Outputs**
  - `WriteToText("output/...")` files for inspection.

---

## Configuration knobs

- **Significance threshold** (e.g., `amount > 100`) inside the filter step.
- **High‑value split** (e.g., `amount > 500`) inside `partition_fn`.
- **FX rate** for `amount_usd` computation (sample constant in the notebook).
- **Window size** and allowed lateness (in the optional windowing cell).

> Tweak these values in the respective cells to explore Beam behavior.

---

## Expected outputs

After a successful run you should see an `output/` directory containing Beam‑sharded text files like:

```
output/
├── high_value_sales-00000-of-00001
└── regular_sales-00000-of-00001
```

Each line is typically formatted as:
```
customer,amount,timestamp
```

If you enable the windowing/aggregation example, you may also see per‑window results (depending on how you wire the final sinks).

---

## Troubleshooting

- **`apache_beam` not found** → Re‑run the first cell that installs Beam, or `pip install apache-beam` in your environment.
- **Permission errors writing `output/`** → Ensure the working directory is writable; on Colab it is.
- **Windows timestamp parsing** → The sample uses ISO‑8601 timestamps; make sure your locale doesn’t alter parsing; the notebook uses `datetime.fromisoformat`.
- **Nothing written** → Confirm filters (thresholds) aren’t excluding all rows and that the write steps are executed.
- **Stale files** → Beam’s `WriteToText` creates sharded outputs. Delete any previous `output/` directory and re‑run if you change sinks.

---

## Extending the example

- Swap `ReadFromText` for a connector (e.g., GCS, Pub/Sub, BigQuery) with the appropriate runner.
- Replace the FX constant with a side input (e.g., broadcast config).
- Add **CombinePerKey** for per‑customer totals, or **GroupIntoBatches** for micro‑batching.
- Package the composite transform into a library module for reuse.
- Parameterize thresholds via `PipelineOptions`.

---

## License

This example notebook is provided under the MIT License. See `LICENSE` (add one if you plan to share publicly).
