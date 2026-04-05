# Data Analysis & Visualization Skill

## Overview
This skill guides Claude in producing enterprise-grade data analysis and visualizations across all data sources (Snowflake/SQL, CSV/Excel uploads, API/JSON), output types (interactive charts, static summaries, data tables, executive narratives), and environments (Claude.ai artifacts, Claude Code). 

Always read this skill fully before beginning any analysis or visualization task.

---

## Phase 1 — Understand Before You Build

Before writing any code or producing any output, establish:

1. **Data source type**: SQL result set / uploaded file / API JSON / pasted data?
2. **Business question**: What decision does this analysis support? Who is the audience?
3. **Output destination**: Claude.ai React artifact / Claude Code file / both?
4. **Grain and shape**: How many rows? What dimensions vs. metrics? Time series or snapshot?
5. **Governance context**: Are there known definitions, business rules, or metric definitions to respect?

If the data source, question, or audience is unclear — ask before proceeding. A wrong assumption in analysis is worse than a delayed answer.

---

## Phase 2 — Data Ingestion Patterns

### Snowflake / SQL
- Always inspect column names and types before summarizing. Never assume column names.
- Apply date filters on large tables (tickets, work orders, invoices, communications).
- Use 2-part names (`edw.ticket`) never 3-part (`PROD.EDW.ticket`).
- Filter on `_desc` columns for enums, not raw integer codes.
- Validate SQL before executing when operating in Claude Code.
- If operating via MCP (e.g., DMG Data MCP), call `get_table_schema()` first, then `build_query_context()` before generating SQL.

### Uploaded CSV / Excel
- Use `papaparse` (React) or `pandas` (Python/Claude Code) to parse.
- Check for: header row presence, encoding issues, blank rows, merged cells (Excel).
- Infer data types — don't treat numeric IDs as measures.
- Report row count, column count, and null rates before proceeding to analysis.
- For Excel: use SheetJS (`import * as XLSX from 'xlsx'`) in React artifacts.

### API / JSON
- Flatten nested structures before analyzing.
- Identify the grain (what does one record represent?).
- Check for pagination — partial data produces misleading analysis.
- Validate that field names match expected schema before computing metrics.

### Pasted / Inline Data
- Parse structure carefully — detect delimiters (comma, tab, pipe).
- Confirm field interpretation with the user if ambiguous.

---

## Phase 3 — Analysis Patterns

### Always lead with the business answer, not the data
Structure every analysis as:
1. **The headline** — direct answer to the question in plain English
2. **The evidence** — supporting metrics, trends, comparisons
3. **The signal** — what stands out, anomalies, outliers
4. **The implication** — what action or decision this supports

### Metric computation rules
- Rates = numerator / denominator — always show both, not just the rate
- Averages are misleading without distribution context — pair with median or range when stakes are high
- Period-over-period comparisons require equal window lengths
- Percentages should sum to a meaningful total — show the denominator
- Never mix grain levels in a single aggregation without explicit intent

### Common analysis types and when to use them
| Analysis Type | When to Use |
|---|---|
| Trend over time | Volume, rate, or metric changing across dates |
| Ranking / leaderboard | Comparative performance across entities |
| Distribution | Understanding spread, outliers, concentration |
| Cohort / segmentation | Behavior differs by group |
| Funnel | Sequential drop-off across stages |
| Correlation | Two metrics moving together (flag, don't assert causation) |
| Composition | Parts-of-whole, mix shift |

---

## Phase 4 — Visualization Selection

### Chart type decision guide
| Data Shape | Recommended Chart |
|---|---|
| Time series (continuous) | Line chart |
| Time series (discrete periods) | Bar chart (grouped or stacked) |
| Ranking (top N) | Horizontal bar chart |
| Part-of-whole | Donut / pie (≤6 slices) or stacked bar |
| Distribution | Histogram or box plot |
| Two metric correlation | Scatter plot |
| Multi-metric comparison | Radar / spider chart |
| Funnel / conversion | Funnel chart or waterfall |
| Geographic | Map (choropleth or bubble) |
| Dense multi-dim data | Heatmap |
| KPI summary | Metric cards with trend indicators |

### Visualization design rules
- **One chart, one question.** Never cram two stories into one chart.
- Always include: chart title, axis labels, units, data source note.
- Use color purposefully — highlight the signal, mute the context.
- Avoid 3D charts, dual axes (unless unavoidable), and pie charts with >6 slices.
- Truncated Y-axes misrepresent magnitude — start at zero for bar charts.
- Color palettes: use sequential for continuous data, diverging for +/- data, categorical for groups. Maximum 7 distinct colors before using a legend.
- Accessibility: never rely on color alone — use labels, patterns, or shapes as secondary encoding.

---

## Phase 5 — Output Formats

### Interactive React Artifact (Claude.ai)

**When to use**: Exploratory analysis, stakeholder-facing dashboards, filterable/drillable views.

**Stack**:
- `recharts` for standard charts (line, bar, area, pie, scatter)
- `d3` for custom / advanced visualizations
- Tailwind utility classes for layout (no custom CSS files)
- `lucide-react` for icons
- `papaparse` for CSV parsing in-browser
- `SheetJS (xlsx)` for Excel parsing

**Structure every React artifact as**:
```jsx
// 1. Imports
// 2. Data / constants (or file upload handler)
// 3. Helper functions (formatters, calculators)
// 4. Sub-components (chart panels, metric cards)
// 5. Main component with layout
// 6. export default
```

**Always include**:
- Loading and empty states
- Error handling for malformed data
- Responsive layout (works at 800px+ width)
- A summary/insight panel alongside charts — never charts alone
- Filters or toggles when data has multiple segments

**Aesthetic direction for data dashboards**:
- Dark or deep neutral backgrounds outperform white for data-dense views
- Use a strong accent color for the primary metric/series
- Metric cards above the fold, detail charts below
- Generous spacing — data density ≠ cramming
- Avoid generic purple gradient + white — commit to a distinct palette

**Sample metric card pattern**:
```jsx
const MetricCard = ({ label, value, delta, deltaLabel }) => (
  <div className="bg-gray-900 rounded-xl p-5 border border-gray-800">
    <p className="text-gray-400 text-sm">{label}</p>
    <p className="text-white text-3xl font-bold mt-1">{value}</p>
    <p className={`text-sm mt-1 ${delta >= 0 ? 'text-emerald-400' : 'text-red-400'}`}>
      {delta >= 0 ? '▲' : '▼'} {Math.abs(delta)}% {deltaLabel}
    </p>
  </div>
);
```

---

### Static Visual Summary (Claude Code — Python)

**When to use**: Batch reports, file outputs, scheduled analysis, PDF-ready charts.

**Stack**:
- `matplotlib` + `seaborn` for styled static charts
- `pandas` for all data manipulation
- `plotly` (with `kaleido`) for interactive HTML exports

**Always**:
- Set figure size explicitly (`figsize=(12, 6)` for widescreen)
- Use `tight_layout()` to prevent label clipping
- Save to `/mnt/user-data/outputs/` with descriptive filename
- Include a title, subtitle, and source annotation on every chart

**Seaborn theme setup**:
```python
import seaborn as sns
import matplotlib.pyplot as plt
sns.set_theme(style="darkgrid", palette="muted")
plt.rcParams['figure.dpi'] = 150
```

---

### Executive Narrative + Visuals

**When to use**: Leadership summaries, board updates, strategic reviews.

**Structure**:
```
## [Topic] — [Time Period]

**Bottom line**: [1-2 sentence answer to the business question]

**What happened**: [Trend narrative — 2-3 sentences]

**What to watch**: [1-2 emerging signals or risks]

**Recommended action**: [Optional — only if clearly supported by data]
```

**Rules**:
- Lead with the conclusion, not the methodology
- Numbers in narrative: round to meaningful precision (61.8K not 61,814)
- Every number cited must appear in the underlying data
- Avoid hedging language when the data is clear — "revenue declined 12%" not "revenue may have experienced some decline"
- Pair with 1-2 supporting charts maximum — don't dump all visuals

---

### Data Tables + Insights (Claude.ai or Claude Code)

**Rules**:
- Sort by the most meaningful dimension by default (usually the primary metric descending)
- Highlight top and bottom performers with color or badges
- Add a "% of total" column for any volume metric
- Cap at 20 rows in artifact — add pagination or a filter for larger sets
- Always include a totals/summary row
- Format numbers consistently: K/M suffixes, consistent decimal places, % symbol

---

## Phase 6 — Quality Checks Before Delivering

Run through this before presenting any output:

- [ ] Does the headline directly answer the business question?
- [ ] Are all metrics defined (not assumed)?
- [ ] Are date ranges and filters stated explicitly?
- [ ] Do totals cross-check (subtotals sum to grand total)?
- [ ] Are outliers acknowledged, not hidden?
- [ ] Is the chart type appropriate for the data shape?
- [ ] Are axis labels, units, and titles present?
- [ ] Does the narrative lead with the finding, not the method?
- [ ] Is the output appropriately scoped for the audience (analyst vs. executive)?
- [ ] Are null/empty/error states handled in any interactive artifact?

---

## Phase 7 — Domain-Specific Rules (Enterprise Data Context)

When working with enterprise operational data:

### Semantic Layer / Business Glossary
- Always respect established metric definitions — do not recompute KPIs from scratch if a pre-aggregated semantic table exists
- Check if a `semantic.*` schema or pre-built mart is available before writing raw joins
- If a metric name is ambiguous (e.g., "completion rate"), clarify which definition applies before computing

### Governance Rules
- Never expose PII in visual outputs (names, emails, phone numbers in charts or tables)
- Flag if a query touches sensitive domains (HR, compensation, individual performance)
- Date grain must match the reporting period convention for the domain

### Common Enterprise Metric Patterns
- **Completion rate**: Completed / (Completed + Cancelled + In Progress) — clarify denominator
- **Cancellation rate**: Cancelled / Total Created
- **Auto-assignment rate**: Auto-Assigned / Total Assigned
- **Margin %**: (Revenue - Cost) / Revenue
- **MTTR**: Mean time from incident open to resolved

---

## Anti-Patterns to Avoid

- Never produce a chart without a business insight attached
- Never present averages without context about distribution or volume
- Never generate SQL that touches >1M rows without a date filter
- Never hardcode sample/fake data in interactive artifacts — use real data passed in or uploaded
- Never use pie charts for time series
- Never assert causation from correlation
- Never skip the quality checklist under time pressure

---

## File Naming Conventions (Claude Code outputs)

```
analysis_[domain]_[metric]_[YYYYMMDD].py       # Analysis scripts
chart_[type]_[topic]_[YYYYMMDD].png            # Static chart exports
dashboard_[topic]_[YYYYMMDD].html              # Interactive HTML exports
report_[topic]_[YYYYMMDD].md                   # Narrative reports
```

---

## Quick Reference — Library Imports

**React Artifact**:
```jsx
import { useState, useEffect, useMemo } from "react";
import { LineChart, BarChart, PieChart, AreaChart, ScatterChart,
         Line, Bar, Pie, Area, Scatter,
         XAxis, YAxis, CartesianGrid, Tooltip, Legend,
         ResponsiveContainer, Cell } from "recharts";
import * as d3 from "d3";
import { TrendingUp, TrendingDown, AlertCircle, Filter } from "lucide-react";
```

**Python (Claude Code)**:
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px
import plotly.graph_objects as go
from pathlib import Path
```
