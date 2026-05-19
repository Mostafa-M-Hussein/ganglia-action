# ganglia-action

Structural PR impact reports — powered by Ganglia.

## What it does

Ganglia builds a call graph of your codebase at index time. When a PR opens, this Action downloads the `gng` binary and runs `gng analyze --pr`, which diffs the two commit SHAs, identifies every symbol touched by the diff, and walks the call graph to surface downstream callers at configurable depth. The result is a precise, deterministic map of blast radius — not an AI guess.

The analysis runs entirely in your runner. No code leaves your environment, no LLM is invoked, and the report is ready in under 30 seconds on most repositories. Output is written as both JSON (machine-readable, `schema_version: 1`) and Markdown (appended to the workflow step summary automatically).

## Quickstart

```yaml
name: Ganglia
on:
  pull_request:
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: ganglia-tools/ganglia-action@v1
```

`fetch-depth: 0` is required so both base and head SHAs are available to `gng`.

## Inputs

| Input | Default | Description |
|---|---|---|
| `command` | `analyze` | Ganglia subcommand. Only `analyze` is supported in v1. |
| `base` | PR base SHA | Base commit SHA for the diff. |
| `head` | PR head SHA | Head commit SHA for the diff. |
| `format` | `json` | Primary output format (`json` or `markdown`). Both formats are always written. |
| `gng-version` | `v0.9.95` | Version of the `gng` binary to download. Must be >= v0.9.95. |
| `max-callers` | `5` | Maximum callers reported per symbol. |
| `impact-depth` | `2` | Call-graph depth for impact traversal. |
| `upload-artifact` | `true` | Upload JSON and Markdown reports as workflow artifacts. |
| `comment-pr` | `false` | Reserved for future use (Ganglia GitHub App handles PR comments). |

## Outputs

| Output | Description |
|---|---|
| `report-json-path` | Absolute path to `ganglia-report.json` on the runner. |
| `report-md-path` | Absolute path to `ganglia-report.md` on the runner. |
| `risk-level` | Risk level from the JSON report: `low`, `medium`, or `high`. |

## Example output

```json
{
  "schema_version": 1,
  "base": "a1b2c3d",
  "head": "e4f5a6b",
  "risk": "medium",
  "touched_symbols": [
    {
      "name": "parse_config",
      "file": "src/config.rs",
      "change": "modified",
      "callers": ["main", "reload_config", "test_parse"],
      "impact_depth": 2
    }
  ],
  "summary": {
    "touched": 3,
    "total_callers": 11,
    "risk_score": 42
  }
}
```

## How it compares

| | Ganglia | CodeRabbit | Sourcegraph |
|---|---|---|---|
| Analysis method | Call-graph traversal | LLM review | Semantic search |
| Deterministic | Yes | No | Partial |
| PR noise | Step summary only | Inline comments | Varies |
| Runs in your runner | Yes | No (SaaS) | No (SaaS) |
| Speed | < 30s | 1–3 min | Varies |
| Cost | Binary download | Per-seat SaaS | Per-seat SaaS |

## Platform support

v1 supports `ubuntu-latest` runners only. macOS and Windows support is planned for v2.

## License

MIT — see [LICENSE](LICENSE).
