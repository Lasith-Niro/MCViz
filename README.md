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


## Limitations

- CSV files are parsed in the browser and are not uploaded by the visualizer.
- Numeric detection classifies a column as numeric when at least 60% of its
  non-empty values can be parsed as numbers.
- Values containing commas or percent signs are parsed as numbers after those
  characters are removed.
- The visualizer does not save uploaded data or chart settings between page
  loads.


