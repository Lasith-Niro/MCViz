# Monitoring Campaign Visualizer

`monitoring-visualizer.html` is a standalone browser-based tool for exploring
monitoring campaign data. Upload a CSV file, choose how indicators should be
labelled, select campaign metrics, and build an interactive Plotly chart.

This README applies only to `monitoring-visualizer.html`. The Streamlit files
in this repository are separate applications and are not required to use the
HTML visualizer.

## Open the visualizer

1. Open `monitoring-visualizer.html` in a modern browser.
2. Drop a CSV file onto the upload area, or click the area to select a file.
3. Review the data preview and detected column types.
4. Select indicators, metrics, and a visualization.

The file can be opened directly from the filesystem. It loads Papa Parse and
Plotly from public CDNs, so an internet connection is normally required when
the page loads. No uploaded data is sent to a server by the visualizer.

## CSV structure

Each row should represent one indicator for a monitoring context. A typical
file has columns like these:

| DYNAMO | SOLUTION | CODE | INDICATOR | BASELINE | TARGET | MC1 | MC2 |
|---|---|---|---|---:|---:|---:|---:|
| D04 | Solution 1 | TExT12.1 | Touristic routes development | 0 | 10 | 2 | 4 |

### Text columns

Text columns identify and describe the indicator. Common examples are:

- `DYNAMO`: project or area identifier
- `SOLUTION`: solution or intervention group
- `CODE`: indicator code
- `INDICATOR`: human-readable indicator name

The **Label indicators by** selector uses these columns. The **Group / colour
by** selector uses another text column to add context to each label and to
group some chart types.

### Numeric columns

Numeric columns contain measurements and become selectable campaign metrics.
The visualizer automatically prefers columns named `MC1`, `MC2`, and similar
`MC<number>` columns as the default metrics.

- `BASELINE`: value before monitoring
- `TARGET`: intended value
- `MC1`, `MC2`, ...: monitoring campaign results

Campaign values should follow one consistent convention. If `MC2` is an
accumulated result, enter the accumulated value in `MC2`; do not add `MC1`
again when preparing the CSV.

### Missing values

The following values are treated as missing for numeric detection and chart
values:

- Empty cells
- `-`
- `TBD`
- `N/A`
- `NA`

Missing numeric values are not automatically converted to zero. Charts that
need a numeric value may leave the corresponding point empty or omit it.

The CSV header should not contain duplicate column names. Empty or unnamed
columns are ignored by the visualizer.

## Using the controls

### Indicators

- **Label indicators by** changes the text shown for each row.
- **Group / colour by** adds group context to labels and controls grouping in
  applicable charts.
- Use the indicator search box to filter the list.
- Use **Select all** or **Select none** to change the visible selections.
- Changing the label or group column preserves the selected rows.

### Metrics

Select one or more numeric campaign columns in **Monitoring campaigns /
metrics**. The available columns are detected from the uploaded CSV.

For accumulated comparisons, select the campaign columns whose contributions
should be displayed. Whether campaign columns are independent measurements or
already accumulated values is determined by the data supplied in the CSV.

### Baseline and Target toggles

The **Baseline** and **Target** checkboxes control whether those reference
columns are shown. They are enabled automatically when matching columns are
present in the CSV, and disabled when the columns are absent.

The references are displayed on indicator-based comparison charts, including
the standard bar charts, stacked bar chart, lollipop chart, and the dedicated
stacked comparison chart.

Both references can be shown together, shown individually, or turned off.

## Visualization types

### Comparison

- **Bar**: horizontal bars for selected metrics
- **Column**: vertical bars for selected metrics
- **Grouped Bar**: metrics grouped by the selected group column
- **Stacked Bar**: selected metrics stacked for each indicator
- **Lollipop**: marker-based comparison by indicator
- **Stacked vs Baseline/Target**: selected campaign metrics stacked together,
  with optional Baseline and Target reference markers

### Trend

- **Line**: plots each selected indicator across the selected metrics
- **Area**: line chart with filled areas

Trend charts require at least two selected metrics.

### Part-to-whole

- **Pie**
- **Donut**
- **Treemap**
- **Sankey**

These charts use the first selected metric when a single metric is required.

### Distribution

- **Histogram**
- **Box Plot**
- **Violin Plot**

### Relationship

- **Scatter**: requires at least two selected metrics; the first is the x-axis
  and the second is the y-axis
- **Bubble**: requires at least three selected metrics; the first two are the
  axes and the third controls bubble size
- **Heatmap**

## Included sample data

The repository includes:

- `sample_data/D04_mc2.csv`: an example dataset containing missing values
- `sample_data/monitoring_no_nulls.csv`: a small complete dataset with no
  empty or null-like values

For a quick test, open the HTML file and upload
`sample_data/monitoring_no_nulls.csv`. It contains `BASELINE`, `TARGET`, `MC1`,
and `MC2`, so all comparison features can be tested immediately.

## Publishing only the HTML

To publish the visualizer, provide this file:

```text
monitoring-visualizer.html
```

No Python environment, Streamlit server, build step, or backend is required.
Keep the CDN script URLs in the HTML, or replace them with local copies of
Papa Parse and Plotly if the published environment cannot access the internet.

## Limitations

- CSV files are parsed in the browser and are not uploaded by the visualizer.
- Numeric detection classifies a column as numeric when at least 60% of its
  non-empty values can be parsed as numbers.
- Values containing commas or percent signs are parsed as numbers after those
  characters are removed.
- The visualizer does not save uploaded data or chart settings between page
  loads.
# Monitoring Campaign Data Visualization Platform

A local Streamlit app for exploring EU-project monitoring-campaign data
(BASELINE / TARGET / MC1 / MC2 per indicator), with 18 chart types and an
optional local-LLM assistant (Ollama + `qwen2.5:7b`) for chart suggestions
and quick Q&A.

## 1. Setup

```bash
cd dashboard
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Optional: local AI assistant (Ollama)

The AI Assistant tab is optional — the rest of the dashboard works without it.

```bash
# install Ollama: https://ollama.com/download
ollama pull qwen2.5:7b
ollama serve            # usually already running as a background service
```

The app talks to `http://localhost:11434` by default (editable in the
sidebar). Nothing about your data is sent anywhere except your own machine.

## 2. Run

```bash
streamlit run app.py
```

Then open the URL Streamlit prints (usually http://localhost:8501).

## 3. Expected CSV format

The app expects the same layout every time:

| DYNAMO | SOLUTION | CODE | INDICATOR | *(blank col)* | BASELINE | TARGET | MC1 | MC2 |
|---|---|---|---|---|---|---|---|---|

- `MC1` = value recorded at Monitoring Campaign 1
- `MC2` = **accumulated** value recorded at Monitoring Campaign 2 (already
  includes MC1's contribution)
- Cells containing `-`, `TBD`, `N/A`, or left blank are treated as missing
  data, not zero
- A bundled sample file (`sample_data/D04_mc2.csv`) is available from the
  sidebar for a quick test drive

## 4. What's inside

```
dashboard/
├── app.py                 # Streamlit UI (filters, tabs, AI assistant)
├── utils/
│   ├── data_utils.py       # CSV loading, cleaning, derived stats
│   ├── charts.py            # One function per chart type (Plotly)
│   └── ollama_utils.py       # Local Ollama client
├── sample_data/D04_mc2.csv
└── requirements.txt
```

### Chart types, by category

- **Comparison**: Bar, Column, Grouped Bar, Lollipop
- **Trend / Time**: Line (Baseline→MC1→MC2→Target), Area, Progress Gantt,
  Candlestick
- **Part-to-Whole**: Pie, Donut, Treemap
- **Distribution**: Histogram, Box Plot, Violin Plot
- **Relationship**: Scatter, Bubble, Heatmap, Sankey

Two chart types don't map naturally onto this data and are explicitly
adapted rather than faked, with a note in the UI:
- **Progress Gantt**: since there are no dates, bars run 0 → Target and
  fill to the latest recorded value instead of showing a schedule.
- **Candlestick**: Baseline/Target/observed values are mapped onto
  Open/High/Low/Close per indicator purely as a visual reuse of the form —
  it is not a real price series.

### Derived fields

`data_utils.load_monitoring_csv` adds a few computed columns you'll see in
the data table and use across charts:

- `LATEST_VALUE` / `LATEST_CAMPAIGN`: MC2 if present, else MC1
- `PROGRESS_PCT`: `(latest − baseline) / (target − baseline) × 100`
- `STATUS`: `Target reached` / `On track` (≥50%) / `Behind` (<50%) /
  `In progress` / `No target` / `No data`

## 5. Extending

- Add a new chart: write a function in `utils/charts.py` returning a
  Plotly figure, then wire it into the matching tab in `app.py`.
- Swap the LLM model: change the "Model" field in the sidebar to any model
  you've pulled with `ollama pull <name>`.
