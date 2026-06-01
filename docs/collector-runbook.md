# Collector Runbook

`civic-source-snapshotter` public repoでsource snapshotを取得するためのrunbookです。

このrunbookはpublic collector側だけを扱います。downstream DB candidate出力、match inventory、product config / CSV反映、owner loadは扱いません。

## Supported collectors

| collector | workflow | role |
| --- | --- | --- |
| Navii detail | `Source Snapshot: Navii Detail` | ナビィopen dataからdetail pageを取得し、人員・在宅/連携・予約/診療時間系tableを抽出する |
| MHLW monthly | `Source Snapshot: MHLW Monthly` | 厚労省地方厚生局の月次source ZIP/XLSXを取得し、checksum付きraw source packageを作る |

## 初回セットアップ

完了済み:

- ownerがpublic repo `civic-source-labs/civic-source-snapshotter` を作成する
- scaffoldをrepo rootへ転記し、初回empty repo bootstrapを `main` へpushする
- collector実装 `collectors/navii_detail/collect.py` を追加する

未完了:

1. `sources/navii/source-manifest.json` のsource snapshot dateとURLを確認する
2. `summary_only` canaryを実行する
3. `keys/owner-age-recipient.example.txt` を `keys/owner-age-recipient.txt` に置き換える
4. `encrypted_full` canaryを実行する

private keyはpublic repoへ置きません。GitHub secretにも登録しません。

collector実装追加とcanary成功は別です。canary結果を確認するまでは、取得導線が本番運用可能とは扱いません。

## Navii Canary 1: summary only

目的は、public repo側のworkflow、source manifest、shard matrix、summary artifactが動くかだけを確認することです。

推奨input:

| input | value |
| --- | --- |
| `execute` | `false` |
| `confirm` | 空 |
| `source_snapshot_date` | `2025-12-01` |
| `run_label` | `collector-navii-detail-20251201-canary` |
| `artifact_mode` | `summary_only` |
| `shard_count` | `1` |
| `max_parallel` | `1` |
| `max_pages_per_shard` | `400` |

確認:

- workflowがmanual dispatchでだけ起動する
- source ZIPをdownloadできる
- `collector-run-manifest.json` が作られる
- `SHA256SUMS` が作られる
- workflow logにraw HTMLや施設単位full dataが出ていない
- `encrypted/README.txt` 以外のraw full artifactが平文でuploadされていない

## Navii Canary 2: encrypted full

目的は、full artifactを暗号化し、owner localで復号できることを確認することです。

推奨input:

| input | value |
| --- | --- |
| `execute` | `true` |
| `confirm` | `owner-approved-public-source-snapshot` |
| `artifact_mode` | `encrypted_full` |
| `shard_count` | `1` |
| `max_parallel` | `1` |
| `workers` | `2` |
| `pause_seconds` | `0.3` |
| `jitter_seconds` | `0.1` |
| `max_pages_per_shard` | `400` |

確認:

- `encrypted/raw-artifacts-shard-*.tar.zst.age` がある
- `SHA256SUMS` とartifactが一致する
- owner localで復号できる
- 復号済みartifactのrow countがmanifest / metricsと一致する
- private normalizer intakeに渡せる
- 暗号化前の `*.tar.zst` やraw CSV / JSONL / HTMLがGitHub artifactへ残っていない

## MHLW Monthly Canary

MHLW monthlyは、まずcanary manifestの一部だけでdry-runします。

推奨input:

| input | value |
| --- | --- |
| `execute` | `false` |
| `confirm` | 空 |
| `source_manifest` | `sources/mhlw_monthly/source-manifest.json` |
| `run_label` | `collector-mhlw-monthly-202604-canary` |
| `artifact_mode` | `summary_only` |
| `max_sources` | `1` |
| `insecure_skip_tls_verify` | `false` |

確認:

- workflowがmanual dispatchでだけ起動する
- `mhlw-source-file-inventory.csv` が作られる
- `source-coverage-summary.csv` が作られる
- `SHA256SUMS` が作られる
- summary_onlyでraw ZIP/XLSXがuploadされていない

MHLW monthlyでraw source fileを取得する場合は、`execute=true`、`confirm=owner-approved-public-source-snapshot`、`artifact_mode=encrypted_full` にします。
厚労省側のTLS chain問題でsummary canaryが証明書検証のみ失敗する場合だけ、owner判断で `insecure_skip_tls_verify=true` を使います。

## Full run

full runはcanaryが成功してからowner判断で行います。

推奨input:

| input | value |
| --- | --- |
| `artifact_mode` | `encrypted_full` |
| `shard_count` | `32` |
| `max_parallel` | `32` |
| `workers` | `2` |
| `pause_seconds` | `0.3` |
| `jitter_seconds` | `0.1` |
| `max_pages_per_shard` | `0` |

実行前確認:

- GitHub Actions minutesの見込み
- source側負荷
- encrypted artifact retention
- owner local decrypt作業時間
- private normalizer intake path

## Handoff

private側へ渡す最小package:

- `manifest/source-snapshot-manifest.json`
- `manifest/collector-run-manifest.json`
- `metrics/fetch-metrics.json`
- `metrics/shard-summary.json`
- `metrics/coverage-summary.csv`
- `metrics/mhlw-source-file-inventory.csv` (MHLW monthly)
- `metrics/source-coverage-summary.csv` (MHLW monthly)
- `checksums/SHA256SUMS`
- `encrypted/raw-artifacts-shard-*.tar.zst.age`
- `encrypted/raw-mhlw-source-files.tar.zst.age` (MHLW monthly)
- GitHub run URL

private側へ渡さないもの:

- GitHub token
- owner private key
- downstream DB candidate CSV
- downstream match result
- downstream DB load SQL
- production secret

## Stop Conditions

- source manifestのURLが404 / 内容不一致
- `fetch_error_rate` がowner許容値を超える
- encrypted artifactが復号できない
- checksum不一致
- workflow logへraw full dataが出ている
- private normalizer intakeがmanifestを読めない
