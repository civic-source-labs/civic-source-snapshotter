# Civic Source Snapshotter

公開sourceから取得したsnapshot artifactを作るpublic collector repoです。downstreamの商品DB、決済、deploy、owner loadとは分離します。

```text
https://github.com/civic-source-labs/civic-source-snapshotter
```

このrepoはpublicです。secret、downstream DB candidate、match result、load SQL、復号private keyは置きません。

## 目的

public collector repoは、公開sourceから取得し、後段のprivate normalizer / matcherへ渡せるartifactを作るだけを担当します。

扱うもの:

- 公開source manifest
- manual dispatch workflow
- fetch / shard / artifact packaging contract
- source collector implementation
- summary / metrics
- checksum
- encrypted full artifactの受け渡し形

扱わないもの:

- downstreamの商品名、価格、販売戦略
- downstream DB schemaやcandidate CSV
- downstream itemとのmatch result
- downstream product config / CSV列への採用判断
- downstream DB load SQL
- DB password、service role、payment / email / deploy secret
- 復号private key

## 構成

```text
civic-source-snapshotter/
  README.md
  .gitignore
  .github/
    workflows/
      navii-detail-snapshot.yml
  docs/
    artifact-schema.md
    collector-runbook.md
  keys/
    README.md
    owner-age-recipient.example.txt
  sources/
    navii/
      source-manifest.json
      README.md
  collectors/
    navii_detail/
      README.md
      collect.py
```

## 実行の流れ

1. `source-manifest.json` のsnapshot date / URLをownerが確認する
2. `summary_only` canaryを実行する
3. `keys/owner-age-recipient.example.txt` を実際のage public recipient `keys/owner-age-recipient.txt` へ置き換える
4. `encrypted_full` canaryを実行し、owner localでdecrypt / checksum確認する
5. private normalizer intakeで読めることを確認する

## 安全境界

- public repoでは `workflow_dispatch` のみから始める
- summary / metricsは平文でよいが、施設単位のfull artifactは暗号化する
- private keyはowner localだけに置く
- public repoのActionsからproduction DBへ接続しない
- public repoへdownstream DB candidate、match result、load SQLを置かない
- downstream側legacy workflowは、public collector canary完了まで削除しない
- empty repo bootstrap後のpublic repo変更は、branch + PRで行う
- `keys/owner-age-recipient.txt` に置くのはpublic recipientだけ。private keyはpublic repoにもGitHub secretにも置かない

## 関連docs

- `docs/artifact-schema.md`
- `docs/collector-runbook.md`
- `collectors/navii_detail/README.md`
- `sources/navii/source-manifest.json`
