# Civic Source Snapshotter Scaffold

PR infra-f用のscaffoldです。`civic-source-snapshotter` public repoへ移す初期ファイル案です。

PR infra-g時点で、ownerはpublic repo `civic-source-labs/civic-source-snapshotter` を作成済みです。このdirectoryは、次のrepo rootへ転記するtemplateとして扱います。

```text
https://github.com/civic-source-labs/civic-source-snapshotter
```

このdirectoryは **転記用template** です。このまま現在のrepo内でActionsを実行するためのものではありません。public repo作成、暗号鍵生成、GitHub secret登録、Actions実行、downstream DB writeはこのPRでは行いません。

## 目的

public collector repoは、公開sourceから取得し、後段のprivate normalizer / matcherへ渡せるartifactを作るだけを担当します。

扱うもの:

- 公開source manifest
- manual dispatch workflow template
- fetch / shard / artifact packaging contract
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

## 初期配置案

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
```

このscaffoldには実際のcollector実装はまだ含めません。`collectors/navii_detail/README.md` に、public repo側で実装するCLI contractだけを固定します。

## 移管時の使い方

1. ownerが `civic-source-labs/civic-source-snapshotter` public repoを作成する
2. このdirectory配下の中身をrepo rootへ転記する
3. 初回empty repo bootstrapだけはowner承認のうえで `main` へ直接pushする
4. `keys/owner-age-recipient.example.txt` を実際のage public recipient `keys/owner-age-recipient.txt` へ置き換える
5. `source-manifest.json` のsnapshot date / URLをownerが確認する
6. collector実装を追加する
7. `summary_only` canaryを実行する
8. `encrypted_full` canaryを実行し、owner localでdecrypt / checksum確認する
9. private normalizer intakeで読めることを確認する

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

- `docs/data/public-collector-repo-spec.md`
- `docs/data/public-collector-bootstrap-runbook.md`
- `docs/data/private-normalizer-load-gate-spec.md`
- `docs/data/navii-workflow-migration-prep.md`
