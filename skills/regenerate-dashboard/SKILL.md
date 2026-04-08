---
name: regenerate-dashboard
description: "Regenerate Dashboard: Query Datadog for fresh build duration metrics and update the dashboard index.html with new data. Use when asked to regenerate, refresh, or update the pipeline dashboard."
---

# Regenerate Pipeline Dashboard

$ARGUMENTS

## Overview

This command queries Datadog for fresh `aws.codebuild.build_duration` metrics across 6 pipeline groups and updates both the `data/*.json` files and `index.html` with the new values. Run this from the dashboard project directory (containing `index.html`).

## Prerequisites — REQUIRED before proceeding

Check if the user provided a **date range** in `$ARGUMENTS`.

- **If a date range is provided**: use those dates. Formats accepted: `D/M/YYYY`, `DD/MM/YYYY`, `YYYY-MM-DD`, or natural language like "1/2/2026 until now". If the end date is omitted or specified as "now"/"today", use today's date. Convert to `YYYY-MM-DD` format.
- **If NO date range is provided** (empty arguments): **ask the user for a start date**. Do NOT proceed without at least a start date.

The default Datadog dashboard ID is `hq7-tai-dma` (full URL: `https://gotx-nprd.datadoghq.com/dashboard/hq7-tai-dma`). Only change if the user explicitly provides a different dashboard ID.

## Step 1: Export metrics from Datadog (MANDATORY)

**You MUST call `/export-datadog-metrics` first.** Do NOT skip this step. Do NOT reuse existing/cached JSON files. Always query Datadog for fresh data.

Pass the user's date range to `/export-datadog-metrics` along with these parameters:

- **Date range**: the start and end dates from the prerequisites
- **Dashboard ID**: `hq7-tai-dma` (or user-provided)
- **Metric**: `aws.codebuild.build_duration` (same for all 6 groups)
- **Interval**: weekly (604800s rollup)
- **Unit conversion**: seconds → minutes (divide by 60, round to 2 decimals)
- **Output directory**: `./data/`

### Pipeline groups to query (6 total)

Query each group separately:

**AppPipeline** (Commit Check of AppPipeline):

| # | Label | Pipeline names | Output file |
|---|-------|---------------|-------------|
| 1 | AppPipeline Android | `gosa-android-app-feature-build`, `sanlam-android-app-feature-build`, `goph-android-app-feature-build` | `app_pipeline_android_build_duration.json` |
| 2 | AppPipeline iOS | `gosa-ios-app-feature-build`, `sanlam-ios-app-feature-build`, `goph-ios-app-feature-build` | `app_pipeline_ios_build_duration.json` |
| 3 | AppPipeline KMP | `gosa-kmp-shared-feature-build`, `slsa-kmp-shared-feature-build`, `goph-kmp-shared-feature-build` | `app_pipeline_kmp_build_duration.json` |

**FeaturePipeline** (Commit Check of FeaturePipeline):

| # | Label | Pipeline names | Output file |
|---|-------|---------------|-------------|
| 4 | FeaturePipeline Android | `android*` (wildcard) | `feature_pipeline_android_build_duration.json` |
| 5 | FeaturePipeline iOS | `ios*` (wildcard) | `feature_pipeline_ios_build_duration.json` |
| 6 | FeaturePipeline KMP | `kmp*` (wildcard) | `feature_pipeline_kmp_build_duration.json` |

### Datadog query format

```
avg:aws.codebuild.build_duration{projectname:<name1> OR projectname:<name2> OR projectname:<name3>}.rollup(avg, 604800)
```

For wildcard groups:
```
avg:aws.codebuild.build_duration{projectname:android*}.rollup(avg, 604800)
```

## Step 2: Generate index.html

After all 6 JSON files are saved in `./data/`, generate `index.html` using the **exact template** below. The dashboard must render **6 separate charts** (one per pipeline), summary cards, and a status banner.

### 2a. DASHBOARD_DATA — replace with fresh data

Replace only the `DASHBOARD_DATA` array contents. Each entry maps a label + color to the JSON file contents:

```js
const DASHBOARD_DATA = [
  { label: "AppPipeline Android",     color: "#34D399", data: <contents of app_pipeline_android_build_duration.json> },
  { label: "AppPipeline iOS",         color: "#60A5FA", data: <contents of app_pipeline_ios_build_duration.json> },
  { label: "AppPipeline KMP",         color: "#A78BFA", data: <contents of app_pipeline_kmp_build_duration.json> },
  { label: "FeaturePipeline Android", color: "#F59E0B", data: <contents of feature_pipeline_android_build_duration.json> },
  { label: "FeaturePipeline iOS",     color: "#F472B6", data: <contents of feature_pipeline_ios_build_duration.json> },
  { label: "FeaturePipeline KMP",     color: "#FB923C", data: <contents of feature_pipeline_kmp_build_duration.json> }
];
```

### 2b. BASELINES — keep unchanged unless user requests

```js
const BASELINES = {
  "AppPipeline": 14.5,      // ≤ 14.5 min
  "FeaturePipeline": 5.0    // ≤ 5.0 min
};
```

### 2c. Full HTML template

The `index.html` MUST follow this exact structure. Do NOT deviate. Copy this template verbatim, only replacing the `DASHBOARD_DATA` array contents with the fresh JSON data from step 1.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mobile CI/CD Pipeline Performance</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-annotation@3.1.0/dist/chartjs-plugin-annotation.min.js"></script>
  <style>
    body { font-family: 'Inter', system-ui, -apple-system, sans-serif; background: #f1f5f9; }
    .chart-card { transition: box-shadow 0.2s; }
    .chart-card:hover { box-shadow: 0 8px 30px rgba(0,0,0,0.08); }
    .stat-value { font-variant-numeric: tabular-nums; }
  </style>
</head>
<body class="min-h-screen">
  <!-- Header -->
  <header class="bg-gradient-to-r from-slate-900 to-slate-800 text-white py-8 px-6 shadow-lg">
    <div class="max-w-7xl mx-auto">
      <h1 id="dashTitle" class="text-3xl font-bold tracking-tight"></h1>
      <p id="dashSubtitle" class="text-slate-300 mt-2 text-lg"></p>
      <p id="dateRange" class="text-slate-400 mt-1 text-sm"></p>
    </div>
  </header>

  <!-- Status Banner -->
  <div class="max-w-7xl mx-auto px-6 -mt-4 mb-4">
    <div id="statusBanner" class="rounded-xl px-6 py-4 text-center font-semibold text-lg shadow-sm"></div>
  </div>

  <!-- Summary Stats -->
  <div class="max-w-7xl mx-auto px-6">
    <div id="summaryCards" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4"></div>
  </div>

  <!-- Charts Grid — 6 SEPARATE charts -->
  <main class="max-w-7xl mx-auto px-6 py-8">
    <div id="chartsGrid" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6"></div>
  </main>

  <!-- Footer -->
  <footer class="bg-slate-800 text-slate-400 text-center py-4 text-sm mt-8">
    <span id="footerText"></span>
  </footer>

  <script>
    const DASHBOARD_TITLE = "Mobile CI/CD Pipeline Performance";
    const DASHBOARD_SUBTITLE = "Average CodeBuild Duration (Weekly)";
    const DASHBOARD_DATA = [
      // === REPLACE THIS ARRAY CONTENTS WITH FRESH JSON DATA ===
    ];
    const BASELINES = {
      "AppPipeline": 14.5,
      "FeaturePipeline": 5.0
    };

    function getBaseline(label) {
      for (const [prefix, target] of Object.entries(BASELINES)) {
        if (label.startsWith(prefix)) return target;
      }
      return null;
    }

    function formatDate(dateStr) {
      const d = new Date(dateStr + 'T00:00:00');
      return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
    }

    function calcStats(data) {
      const values = data.map(d => d.value);
      const avg = values.reduce((a, b) => a + b, 0) / values.length;
      return { avg: avg.toFixed(1), min: Math.min(...values).toFixed(1), max: Math.max(...values).toFixed(1) };
    }

    function createSummaryCard(label, color, stats, metricData) {
      const baseline = getBaseline(label);
      const dataPoints = metricData.data;
      const latestValue = dataPoints[dataPoints.length - 1].value;
      const withinTarget = dataPoints.filter(d => d.value <= baseline).length;
      const totalWeeks = dataPoints.length;
      const compliancePct = Math.round((withinTarget / totalWeeks) * 100);
      const latestWithin = latestValue <= baseline;
      const borderColor = latestWithin ? '#10b981' : '#ef4444';
      const statusBadge = latestWithin
        ? `<span class="inline-flex items-center gap-1 text-xs font-medium px-2 py-0.5 rounded-full bg-emerald-50 text-emerald-700">✅ ${latestValue.toFixed(1)} min</span>`
        : `<span class="inline-flex items-center gap-1 text-xs font-medium px-2 py-0.5 rounded-full bg-rose-50 text-rose-600">❌ ${latestValue.toFixed(1)} min</span>`;
      return `
        <div class="chart-card bg-white rounded-xl shadow-sm p-5 border-2" style="border-color:${borderColor}">
          <div class="flex items-center justify-between mb-3">
            <div class="flex items-center gap-3">
              <div class="w-3 h-3 rounded-full" style="background:${color}"></div>
              <h3 class="font-semibold text-slate-700 text-sm uppercase tracking-wide">${label}</h3>
            </div>
            ${statusBadge}
          </div>
          <div class="grid grid-cols-3 gap-4 text-center">
            <div><p class="text-xs text-slate-400 mb-1">Average</p><p class="stat-value text-xl font-bold text-slate-800">${stats.avg}<span class="text-xs text-slate-400 ml-0.5">min</span></p></div>
            <div><p class="text-xs text-slate-400 mb-1">Min</p><p class="stat-value text-xl font-bold text-emerald-600">${stats.min}<span class="text-xs text-slate-400 ml-0.5">min</span></p></div>
            <div><p class="text-xs text-slate-400 mb-1">Max</p><p class="stat-value text-xl font-bold text-rose-500">${stats.max}<span class="text-xs text-slate-400 ml-0.5">min</span></p></div>
          </div>
          <div class="mt-3 pt-3 border-t border-slate-100">
            <div class="flex items-center justify-between text-xs mb-1.5">
              <span class="text-slate-500">Target: <span class="font-semibold text-slate-700">≤ ${baseline} min</span></span>
              <span class="font-semibold" style="color:${compliancePct >= 75 ? '#10b981' : compliancePct >= 50 ? '#f59e0b' : '#ef4444'}">${withinTarget}/${totalWeeks} weeks (${compliancePct}%)</span>
            </div>
            <div class="w-full bg-slate-100 rounded-full h-2">
              <div class="h-2 rounded-full transition-all" style="width:${compliancePct}%; background:${compliancePct >= 75 ? '#10b981' : compliancePct >= 50 ? '#f59e0b' : '#ef4444'}"></div>
            </div>
          </div>
        </div>`;
    }

    function createChartCard(id, label, color) {
      return `
        <div class="chart-card bg-white rounded-xl shadow-sm p-5 border border-slate-100">
          <div class="flex items-center justify-between mb-4">
            <div class="flex items-center gap-2">
              <div class="w-3 h-3 rounded-full" style="background:${color}"></div>
              <h3 class="font-semibold text-slate-700">${label} Build Duration</h3>
            </div>
            <span id="trend-${id}" class="text-xs font-medium px-2 py-1 rounded-full"></span>
          </div>
          <div class="relative" style="height:220px">
            <canvas id="chart-${id}"></canvas>
          </div>
          <div id="stats-${id}" class="grid grid-cols-3 gap-2 mt-3 pt-3 border-t border-slate-100 text-center text-xs text-slate-500"></div>
        </div>`;
    }

    function renderChart(id, label, color, metricData) {
      const labels = metricData.data.map(d => formatDate(d.week_start));
      const values = metricData.data.map(d => Math.round(d.value * 100) / 100);
      const stats = calcStats(metricData.data);
      const baseline = getBaseline(label);

      const trendEl = document.getElementById(`trend-${id}`);
      const firstVal = values[0], lastVal = values[values.length - 1];
      const pctChange = ((lastVal - firstVal) / firstVal * 100).toFixed(0);
      if (pctChange < 0) {
        trendEl.textContent = `↓ ${Math.abs(pctChange)}%`;
        trendEl.className = 'text-xs font-medium px-2 py-1 rounded-full bg-emerald-50 text-emerald-600';
      } else {
        trendEl.textContent = `↑ ${pctChange}%`;
        trendEl.className = 'text-xs font-medium px-2 py-1 rounded-full bg-rose-50 text-rose-500';
      }

      document.getElementById(`stats-${id}`).innerHTML = `
        <div><span class="text-slate-400">Avg</span> <span class="font-semibold text-slate-700">${stats.avg} min</span></div>
        <div><span class="text-slate-400">Min</span> <span class="font-semibold text-emerald-600">${stats.min} min</span></div>
        <div><span class="text-slate-400">Max</span> <span class="font-semibold text-rose-500">${stats.max} min</span></div>`;

      const pointColors = values.map(v => v <= baseline ? '#10b981' : '#ef4444');
      const ctx = document.getElementById(`chart-${id}`).getContext('2d');
      const gradient = ctx.createLinearGradient(0, 0, 0, 220);
      gradient.addColorStop(0, color + '30');
      gradient.addColorStop(1, color + '05');

      const annotationConfig = {};
      if (baseline !== null) {
        annotationConfig.baselineLine = {
          type: 'line', yMin: baseline, yMax: baseline,
          borderColor: '#ef4444', borderWidth: 2, borderDash: [6, 4],
          label: { display: true, content: `Target: ${baseline} min`, position: 'end',
            backgroundColor: 'rgba(239, 68, 68, 0.85)', color: '#fff',
            font: { size: 10, weight: 'bold' }, padding: { top: 2, bottom: 2, left: 6, right: 6 }, borderRadius: 4 }
        };
        annotationConfig.dangerZone = {
          type: 'box', yMin: baseline, yMax: undefined,
          backgroundColor: 'rgba(239, 68, 68, 0.06)', borderWidth: 0
        };
      }

      new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [{
            label: `${label} (min)`, data: values,
            borderColor: color, backgroundColor: gradient, borderWidth: 2.5,
            fill: true, tension: 0.35, pointRadius: 5,
            pointBackgroundColor: pointColors, pointBorderColor: pointColors,
            pointBorderWidth: 2, pointHoverRadius: 7, pointHoverBackgroundColor: pointColors,
          }]
        },
        options: {
          responsive: true, maintainAspectRatio: false,
          plugins: {
            legend: { display: false },
            tooltip: {
              backgroundColor: '#1e293b', titleFont: { size: 12 }, bodyFont: { size: 13, weight: 'bold' },
              padding: 10, cornerRadius: 8,
              callbacks: {
                title: (items) => items[0].label,
                label: (item) => {
                  const val = item.parsed.y;
                  const diff = val - baseline;
                  const status = val <= baseline ? '✅ Within target' : '❌ Above target';
                  return [` ${val.toFixed(2)} minutes`, ` ${status} (${diff > 0 ? '+' : ''}${diff.toFixed(2)} min)`];
                }
              }
            },
            annotation: { annotations: annotationConfig }
          },
          scales: {
            x: { grid: { display: false }, ticks: { font: { size: 11 }, color: '#94a3b8' } },
            y: { grid: { color: '#f1f5f9' }, ticks: { font: { size: 11 }, color: '#94a3b8', callback: v => parseFloat(v.toFixed(2)) + ' min' }, beginAtZero: false }
          }
        }
      });
    }

    function init() {
      document.getElementById('dashTitle').textContent = DASHBOARD_TITLE;
      document.getElementById('dashSubtitle').textContent = DASHBOARD_SUBTITLE;
      document.title = DASHBOARD_TITLE;

      const results = DASHBOARD_DATA.map(entry => ({ label: entry.label, color: entry.color, metricData: entry.data }));

      // Date range
      const allDates = results.flatMap(r => r.metricData.data.map(d => d.week_start));
      const minDate = formatDate(allDates.sort()[0]);
      const maxDate = formatDate(allDates.sort().reverse()[0]);
      document.getElementById('dateRange').textContent = `${minDate} — ${maxDate} · ${results[0].metricData.interval} data`;

      // Status banner
      const pipelinesWithinTarget = results.filter(r => {
        const baseline = getBaseline(r.label);
        const latest = r.metricData.data[r.metricData.data.length - 1].value;
        return latest <= baseline;
      }).length;
      const totalPipelines = results.length;
      const bannerEl = document.getElementById('statusBanner');
      if (pipelinesWithinTarget === totalPipelines) {
        bannerEl.className = 'rounded-xl px-6 py-4 text-center font-semibold text-lg shadow-sm bg-emerald-50 text-emerald-800 border border-emerald-200';
        bannerEl.innerHTML = `✅ All ${totalPipelines} pipelines within target this week`;
      } else if (pipelinesWithinTarget >= totalPipelines / 2) {
        bannerEl.className = 'rounded-xl px-6 py-4 text-center font-semibold text-lg shadow-sm bg-amber-50 text-amber-800 border border-amber-200';
        bannerEl.innerHTML = `⚠️ ${pipelinesWithinTarget} of ${totalPipelines} pipelines within target this week`;
      } else {
        bannerEl.className = 'rounded-xl px-6 py-4 text-center font-semibold text-lg shadow-sm bg-rose-50 text-rose-800 border border-rose-200';
        bannerEl.innerHTML = `❌ ${pipelinesWithinTarget} of ${totalPipelines} pipelines within target this week`;
      }

      // Summary cards (6 cards in 3-column grid)
      document.getElementById('summaryCards').innerHTML = results.map(r =>
        createSummaryCard(r.label, r.color, calcStats(r.metricData.data), r.metricData)
      ).join('');

      // Chart cards (6 SEPARATE charts in 3-column grid)
      document.getElementById('chartsGrid').innerHTML = results.map((r, i) =>
        createChartCard(i, r.label, r.color)
      ).join('');

      // Render all 6 charts
      results.forEach((r, i) => renderChart(i, r.label, r.color, r.metricData));

      // Footer
      const latestDate = allDates.sort().reverse()[0];
      document.getElementById('footerText').textContent = `Data source: Datadog · Last updated: ${formatDate(latestDate)}`;
    }

    init();
  </script>
</body>
</html>
```

### 2d. Key visualization requirements

The template above produces:

1. **Header** — dark gradient banner with title, subtitle, date range
2. **Status banner** — green/amber/red based on how many pipelines are within target
3. **6 summary cards** — 3-column grid, each card shows: label, latest value badge (✅/❌), avg/min/max stats, compliance progress bar with target
4. **6 separate line charts** — 3-column grid, one chart per pipeline, each with:
   - Trend badge (↓ green or ↑ red percentage change)
   - Line chart with gradient fill
   - Red dashed baseline/target line with annotation label
   - Red danger zone shading above baseline
   - Points colored green (≤ target) or red (> target)
   - Tooltip showing value + within/above target status
   - Avg/Min/Max stats below chart
5. **Footer** — data source and last updated date

### 2e. Rules

- **Light mode ONLY** — no dark mode, no `prefers-color-scheme: dark`, no dark backgrounds
- **Do NOT combine pipelines into a single chart** — each pipeline gets its own chart
- **Do NOT change** DASHBOARD_TITLE, DASHBOARD_SUBTITLE, or BASELINES unless user explicitly requests it
- **Do NOT modify** the HTML structure, CSS, or JavaScript logic — only replace the `DASHBOARD_DATA` array contents
- Use Tailwind CSS via CDN, Chart.js 4.x, and chartjs-plugin-annotation 3.x exactly as shown

## Step 3: Verify and report

1. Confirm all 6 JSON files exist in `./data/`
2. Confirm `index.html` has been generated with the fresh DASHBOARD_DATA
3. Report a summary: date range used, number of data points per group, any groups with no data
4. Remind the user: "Open `index.html` in a browser to see the updated dashboard — no server needed."
