# pr-vi-history

Reusable GitHub workflow set for pull-request VI history analysis.

## Latest release

- Current release: [`v1.1.6`](https://github.com/svelderrainruiz/pr-vi-history/releases/tag/v1.1.6)
- Mutable major tag: `v1` now points to `v1.1.6`

Recommended caller pin:

- Stabilization: pin exact tag (`@v1.1.6`)
- Broad adoption: pin major (`@v1`)

## What this repository provides

- `.github/workflows/pr-vi-history.yml`
  reusable PR history workflow (Windows compare + Linux smoke lane)
- `.github/workflows/vi-history-linux-compare.yml`
  hosted Linux compare workflow callable by name
- `tools/*`
  local scripts used by both reusable workflows

## PR history reusable workflow

Usage from caller repository:

```yaml
name: PR VI History Analysis

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review, labeled]
    paths:
      - '**/*.vi'

permissions:
  contents: read
  pull-requests: write

jobs:
  vi-history:
    if: ${{ !github.event.pull_request.draft }}
    uses: svelderrainruiz/pr-vi-history/.github/workflows/pr-vi-history.yml@v1.1.6
    with:
      pr_number: ${{ github.event.pull_request.number }}
      fetch_depth: '20'
      max_pairs: '6'
      compare_modes: 'default,attributes'
      include_merge_parents: false
      windows_runner: windows-2025
      windows_image: nationalinstruments/labview:2026q1-windows
      linux_runner: ubuntu-latest
      linux_image: nationalinstruments/labview:2026q1-linux
      enable_linux_smoke: true
      upload_artifact: true
      post_comment: true
    secrets: inherit
```

Key inputs:

- `windows_image` (default `nationalinstruments/labview:latest-windows`)
- `linux_image` (default `nationalinstruments/labview:latest-linux`)
- `windows_runner` (default `windows-2025`)
- `linux_runner` (default `ubuntu-latest`)
- `enable_linux_smoke` (default `true`)

Outputs:

- Existing outputs remain stable:
  `summary_path`, `results_root`, `manifest_path`, `pair_count`,
  `target_count`, `completed_count`, `diff_count`, `markdown_path`,
  `artifact_name`
- Additive lane outputs:
  `windows_lane_status`, `linux_lane_status`

Lane behavior:

- Windows lane is hard-gated.
- Linux smoke lane runs in parallel path and reports status; current finalize
  behavior keeps Linux smoke informational.

## Hosted Linux compare workflow (by name)

Use this for direct hosted Linux compare/report execution without the full PR
history graph.

```yaml
jobs:
  linux-compare:
    uses: svelderrainruiz/pr-vi-history/.github/workflows/vi-history-linux-compare.yml@v1.1.6
    with:
      base_vi: fixtures/vi/VI1.base.vi
      head_vi: fixtures/vi/VI1.head.vi
      linux_image: nationalinstruments/labview:2026q1-linux
      report_path: tests/results/vi-history-linux/compare-report.html
      timeout_seconds: '420'
      upload_artifact: true
```

Linux compare contract:

- Job timeout is `10` minutes.
- Image pull step is bounded to `300` seconds.
- Compare timeout default is `420` seconds.
- Artifact includes report + capture + stdout/stderr.

Linux compare outputs:

- Paths: `report_path`, `capture_path`, `stdout_path`, `stderr_path`
- Status: `compare_status`, `result_class`, `is_diff`, `lane_status`
- Diff metrics: `diff_detail_count`, `diff_image_count`

## Troubleshooting hosted stalls

- If Linux pull stalls, inspect the `Pull NI Linux image` step first.
- If compare fails, inspect `ni-linux-container-capture.json` and stderr.
- If caller run is waiting on Windows queue time, use Linux compare workflow as
  first fail-fast gate in caller workflows.

## Release runbook

See [docs/RELEASE.md](docs/RELEASE.md) for the deterministic release sequence
(`tag -> release -> major-tag move -> caller repin`).
