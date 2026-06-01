# Navii Source Manifest

`source-manifest.json` は、ナビィ / 医療情報ネットの公式open data snapshotを表します。

このmanifestには公開URL、snapshot date、取得対象kindだけを置きます。downstream DB candidate、match result、product config判断は含めません。

更新時は次を確認します。

- snapshot date
- URLが公式ページ由来であること
- expected filename
- checksumはworkflow実行時にartifact側で生成すること
- source terms URL
