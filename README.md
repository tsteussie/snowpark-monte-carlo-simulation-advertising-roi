# Monte Carlo Sales Forecast Simulation Workbook

## Overview

This Snowpark workbook implements a Monte Carlo simulation to estimate sales performance across multiple marketing channels and customer segments. By sampling from defined probability distributions for lead generation, conversion rates, revenue per sale, and profit margins, the simulation provides insights into expected outcomes, variability (confidence intervals and percentiles), convergence behavior, and ROI.

## Features

* **Configurable simulation parameters**: number of batches, batch size, and convergence blocks.
* **Multiple marketing channels**: A–E, each with its own spend and segment ratios.
* **Customer segments**: V–Z, with dynamic ratio sampling summing to 1.
* **Distributions for**:
  * Marketing‐spend‐to‐lead ratios
  * Conversion rates per channel/segment
  * Sales revenue per unit
  * Profit margins per sale
* **Summary statistics**: mean, standard deviation, 95% confidence intervals, and key percentiles (5th, 25th, 50th, 75th, 95th) for leads, orders, revenue, and profit.
* **Convergence analysis**: tracks how batch estimates stabilize over blocks.
* **ROI calculation**: compares average profit against total marketing spend.
* **Visualization**: convergence plots saved to disk and displayed.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setup](#setup)
3. [Simulation Parameters](#simulation-parameters)
4. [Distributions](#distributions)
5. [Generating a Batch](#generating-a-batch)
6. [Running the Simulation](#running-the-simulation)
7. [Summary Statistics](#summary-statistics)
8. [Convergence Analysis](#convergence-analysis)
9. [ROI Calculation](#roi-calculation)
10. [Visualization](#visualization)
11. [Usage](#usage)
12. [Contributing](#contributing)
13. [License](#license)

---

## Prerequisites

* Python 3.8+ with the following packages:
  * `snowflake-snowpark-python`
  * `matplotlib`
  * `pandas`
* Access to a Snowflake account with `SYSADMIN` role and a warehouse.

## Setup

1. Clone this repository.
2. Install dependencies:

   ```bash
   pip install snowflake-snowpark-python matplotlib pandas
   ```
3. Ensure `MPLCONFIGDIR` is writable:

   ```python
   import os
   os.environ['MPLCONFIGDIR'] = '/tmp/matplotlib'
   ```
4. Configure connection parameters in `main()` (role, database, schema).

## Simulation Parameters

* `num_batches`: Number of independent simulation batches.
* `batch_size`: Number of simulations per batch.
* `block_count`: Number of blocks for convergence analysis.

These can be modified at the top of the notebook to control scale and precision.

## Distributions

1. **Marketing Spend**: Fixed spend per channel (A–E).
2. **Segment Ratios**: For each channel, sampling ratios for segments V–Y from normal distributions; segment Z is the residual to sum ratios to 1.
3. **Spend-to-Lead Ratio**: Normal distribution per channel to convert spend into lead count.
4. **Conversion Rates**: Normal distribution per channel/segment to convert leads into orders.
5. **Revenue per Unit**: Normal distribution per channel/segment.
6. **Profit Margins**: Normal distribution per channel/segment.

## Generating a Batch

Implemented in `generate_batch()`:
* Creates a Snowpark DataFrame with a `sim_idx` column.
* Injects spend and samples lead ratios to compute leads.
* Samples segment ratios and allocates leads across segments.
* Samples conversion rates to compute orders.
* Samples revenue and profit margins to compute total revenue and profit.
* Tags the batch ID.

## Running the Simulation

The `run_simulation()` function:
1. Iterates over batches:
   * Calls `generate_batch()`.
   * Calculates total leads, orders, revenue, and profit by summing across channels and segments.
   * Records convergence blocks for each metric.
   * Computes summary stats (mean, std, 95% CI, percentiles) using Snowpark and pandas conversion.
   * Prints description of each calculation step for clarity.
2. Returns convergence data and a list of summary statistics.

## Summary Statistics

For each batch and metric (leads, orders, revenue, profit):
* Mean and standard deviation.
* 95% confidence interval (±1.96·σ/√N).
* Percentiles at 5%, 25%, 50%, 75%, and 95%.

## Convergence Analysis

* Splits simulations into `block_count` equally sized blocks.
* Computes block‐wise average for each metric.
* Tracks cumulative mean across blocks to assess stability.

## ROI Calculation

* Computes ROI = (mean total profit − total marketing spend) / total marketing spend.
* Prints ROI percentage.

## Visualization

Convergence plots are generated for each metric and saved under `/tmp`:
* `leads_convergence.png`
* `orders_convergence.png`
* `revenue_convergence.png`
* `profit_convergence.png`

These are displayed inline if running in a notebook environment, or saved for inspection otherwise.

## Usage

Run the notebook within Snowflake's snowpark environment, adjusting the top‐level parameters to scale simulations or tweak distributions.

## Contributing

Feel free to submit issues or pull requests to:

* Add more sophisticated distributions (e.g., log‐normal).
* Parallelize batch generation for performance.
* Extend to other key performance metrics.

## License

This project is licensed under the [MIT License](LICENSE).
