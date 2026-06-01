# MHLW Monthly Source Manifest

厚労省月次source取得用のmanifestです。

このmanifestは公開sourceの取得だけを表します。downstreamの商品DB、match result、load SQL、価格、CSV列、顧客情報は含めません。

## Source row contract

`source_urls` の各rowは、旧GasoLead `pipeline_sources` の公開取得に必要な情報だけを持ちます。

```json
{
  "source_key": "kinki-medical-todokede",
  "pipeline_slug": "medical",
  "region": "近畿",
  "source_label": "届出受理_医科",
  "source_type": "todokede",
  "fetch_type": "xpath",
  "page_url": "https://...",
  "xpath": "//a[contains(@href, \"sisetukijun_ika\") and contains(@href, \".zip\")]",
  "download_subdir": "近畿/届出受理",
  "expected_filename": "kinki-medical-todokede.zip"
}
```

`expected_filename` が空の場合、collectorはresolved URLのbasenameを使います。

## Manifest source

Full manifestはGasoLead private repo側のread-only SQLで `pipeline_sources` から生成し、このrepoにPRで取り込みます。

このdirectoryの `source-manifest.json` はcollector canary用の最小manifestです。
