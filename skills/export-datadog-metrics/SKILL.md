---
name: export-datadog-metrics
description: "Export Datadog Metrics: Query Datadog build metrics (aws.codebuild.build_duration) and save as JSON files. Use when asked to export, fetch, or query pipeline build duration data."
---

# Export Datadog Metrics to JSON

$ARGUMENTS

## When to use

When the user asks to export, fetch, or query Datadog build metrics and save them as JSON files. Triggered by requests like:

- "export datadog metrics to json"
- "fetch build duration data and save to json"
- "query pipeline metrics and export"

## Instructions

### 1. Gather parameters from the user

**REQUIRED — you MUST ask for these if not provided:**

- **Date range**: The time range for querying metrics. The user provides a start date and optionally an end date. Formats accepted: `D/M/YYYY`, `DD/MM/YYYY`, `YYYY-MM-DD`, or natural language like "1/2/2026 until now". If the end date is omitted or specified as "now"/"today", use today's date. Convert the provided dates to `YYYY-MM-DD` format. **Do NOT proceed without at least a start date.**
- **Dashboard ID** *(optional)*: Datadog dashboard ID. Defaults to `hq7-tai-dma`. The full dashboard URL is constructed as `https://gotx-nprd.datadoghq.com/dashboard/<dashboard_id>`.

**Optional parameters** (use defaults if not provided):

- **Metric name**: defaults to `aws.codebuild.build_duration` (BUILD phase duration). Other available metrics: `aws.codebuild.duration` (total build time), `aws.codebuild.provisioning_duration`, `aws.codebuild.queued_duration`
- **Pipeline names**: list of CodeBuild `projectname` values to query (grouped by category)
- **Interval**: `weekly` (604800s rollup), `daily` (86400s rollup), or `biweekly`
- **Unit conversion**: whether to convert (e.g. seconds to minutes, default: yes)
- **Output directory**: defaults to `./data/`

### 2. Query the metrics

Use `mcp__datadog-mcp__get_datadog_metric` with:

```
queries: ["avg:aws.codebuild.build_duration{projectname:<name1> OR projectname:<name2> OR ...}.rollup(avg, <interval_seconds>)"]
from: "<start_date>T00:00:00Z"
to: "<end_date>T23:59:59Z"
raw_data: true
```

- Query each metric group separately (max 3 queries per call for reliability).
- If Datadog overrides the rollup interval (e.g. returns 2-week intervals when you asked for weekly), note this in the output JSON.
- If a query returns no data, verify tag names using `mcp__datadog-mcp__get_datadog_metric_context` with `tag_filter` and `include_tag_values: true`.

### 3. Write JSON files

Save each metric to a separate JSON file in the output directory. Always use this single flat schema:

```json
{
  "metric": "<datadog_metric_name>",
  "description": "<human-readable description>",
  "unit": "<converted unit, e.g. minutes>",
  "pipeline_names": ["<projectname1>", "<projectname2>"],
  "source_url": "https://gotx-nprd.datadoghq.com/dashboard/<dashboard_id>",
  "datadog_site": "gotx-nprd.datadoghq.com",
  "date_range": { "from": "<YYYY-MM-DD>", "to": "<YYYY-MM-DD>" },
  "interval": "<weekly|daily|biweekly>",
  "data": [
    { "week_start": "<YYYY-MM-DD>", "value": <number> }
  ]
}
```

One file per metric per platform. No nested structures or phase groupings.

**Naming convention**: `<pipeline_type>_<platform>_<metric_short_name>.json`

- Use `app_pipeline_` prefix for [AppPipeline] metrics (e.g. `app_pipeline_android_build_duration.json`)
- Use `feature_pipeline_` prefix for [FeaturePipeline] metrics (e.g. `feature_pipeline_ios_build_duration.json`)
- Description format: `[AppPipeline][<Platform>] <metric description>` or `[FeaturePipeline][<Platform>] <metric description>`

**Value conversion**: If converting seconds to minutes, divide by 60 and round to 2 decimal places.

**Date key in data array**: Use `week_start` for weekly, `date` for daily, `period_start` for biweekly.

### 4. Report results

Summarize what was exported: file paths, number of data points, any metrics that returned no data, and any interval overrides by Datadog.

## Known project name mappings

These are verified `projectname` tag values for mobile build pipelines:

**[AppPipeline]** — Commit Check of AppPipeline (metric: `aws.codebuild.build_duration`):

- **Android**: `gosa-android-app-feature-build`, `sanlam-android-app-feature-build`, `goph-android-app-feature-build`
- **iOS**: `gosa-ios-app-feature-build`, `sanlam-ios-app-feature-build`, `goph-ios-app-feature-build`
- **KMP**: `gosa-kmp-shared-feature-build`, `slsa-kmp-shared-feature-build` (not sanlam), `goph-kmp-shared-feature-build`

**[FeaturePipeline]** — Commit Check of FeaturePipeline (metric: `aws.codebuild.build_duration`):

- **Android**: wildcard `projectname:android*`
- **iOS**: wildcard `projectname:ios*`
- **KMP**: wildcard `projectname:kmp*`

### Naming patterns

- Feature builds: `<app>-<platform>-app-feature-build` or `<app>-kmp-shared-feature-build`
- Apps: `gosa`, `sanlam` (or `slsa` for KMP), `goph`
