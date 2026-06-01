# Artifact Schema

`civic-source-snapshotter` からprivate normalizer / matcherへ渡すartifact contractです。

public repoのartifactは第三者に見られる前提で扱います。summary / metricsは平文でよい一方、施設単位のfull artifactは暗号化します。

## Package Layout

```text
run-{source_id}-{source_snapshot_date}-{run_label}/
  manifest/
    source-snapshot-manifest.json
    collector-run-manifest.json
  metrics/
    fetch-metrics.json
    shard-summary.json
    coverage-summary.csv
  checksums/
    SHA256SUMS
  encrypted/
    raw-artifacts-shard-000.tar.zst.age
```

`summary_only` canaryでは `encrypted/raw-artifacts-shard-*.tar.zst.age` は作らず、summary / metrics / manifest / checksumだけを確認します。

## `collector-run-manifest.json`

```json
{
  "schema_version": "1.0",
  "collector_repo": "civic-source-snapshotter",
  "source_id": "navii_detail",
  "source_snapshot_date": "2025-12-01",
  "run_label": "collector-navii-detail-20251201-canary",
  "github_run_id": "123456789",
  "github_run_url": "https://github.com/civic-source-labs/civic-source-snapshotter/actions/runs/123456789",
  "completed_at": "2026-06-01T00:00:00Z",
  "artifact_mode": "summary_only",
  "shards": {
    "shard_count": 1,
    "max_parallel": 1,
    "workers_per_shard": 2
  }
}
```

## `fetch-metrics.json`

```json
{
  "schema_version": "1.0",
  "source_id": "navii_detail",
  "source_snapshot_date": "2025-12-01",
  "candidate_count": 400,
  "fetch_ok_count": 400,
  "fetch_error_count": 0,
  "fetch_error_rate": 0,
  "retry_count": 2,
  "timeout_seconds": 30,
  "pause_seconds": 0.3,
  "jitter_seconds": 0.1,
  "user_agent_mode": "rotate-transparent"
}
```

## `shard-summary.json`

```json
{
  "schema_version": "1.0",
  "source_id": "navii_detail",
  "source_snapshot_date": "2025-12-01",
  "shard_count": 1,
  "shards": [
    {
      "shard_index": 0,
      "candidate_count": 400,
      "fetch_ok_count": 400,
      "fetch_error_count": 0,
      "artifact_dir": "raw-artifacts/shard-000"
    }
  ]
}
```

## Raw Artifact Contents

暗号化前のfull artifactには、少なくとも次を含めます。

```text
raw-artifacts/
  shard-000/
    candidates.csv.gz
    page-coverage.csv.gz
    table-rows.csv.gz
    summary.csv.gz
    coverage-summary.csv.gz
    run-metrics.json
```

shard job間の中間artifactもpublic repo上では見える前提になるため、`encrypted_full` ではshardごとに暗号化したartifactだけをuploadします。未暗号化のraw shard directoryはpackage artifactへ含めません。

## Public Log Boundary

logに出してよい:

- source id
- source snapshot date
- run label
- shard index
- candidate count
- fetch ok / error count
- checksum
- artifact file name

logに出さない:

- raw HTML本文
- raw full CSV / JSONL本文
- 施設単位の全件table rows
- 復号済みartifact本文
- owner local path
- downstream DB candidate CSV
- match result
- secret / token
