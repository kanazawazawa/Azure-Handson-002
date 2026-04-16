# Issue #002: Azure CLI の API バージョン不整合で Application Gateway / Log Analytics コマンドが失敗する

## 発生箇所
- Lab03 Step 6 (確認コマンド): `az network application-gateway show / rule show / rule list`
- Lab03 Step 7: `az network application-gateway show`, `az monitor log-analytics workspace show`
- Lab03 Step 9 (CLI 参考): `az monitor log-analytics workspace show`

## 症状
`az network application-gateway show` が `InvalidApiVersionParameter` エラーで失敗する。

```
(InvalidApiVersionParameter) The api-version '2023-11-01' is invalid.
The supported versions are '2025-04-01,2025-03-01,...,2023-07-01,...'
```

`az monitor log-analytics workspace show` も同様に `api-version '2025-02-01'` が invalid。

## 原因
Azure CLI 2.85.0 が使用する API バージョン (`2023-11-01` for network, `2025-02-01` for monitor) が、
当該サブスクリプション/リージョンでサポートされていない。CLI のバージョンと Azure RP のサポート API バージョンの
不整合と思われる。

## 影響範囲
- **リソースの作成は成功する** (create コマンドは動作)
- **show / list / update コマンドが失敗する場合がある**
- Lab03 Step 7 の手順書コマンド (`az network application-gateway show --query id`, `az monitor log-analytics workspace show --query id`) がそのまま実行できない

## 回避策
リソース ID を手動構成するか、`az rest` で明示的に API バージョンを指定する:

```bash
# リソースIDを手動構成
SUB_ID=$(az account show --query id -o tsv)
AGW_ID="/subscriptions/${SUB_ID}/resourceGroups/${RG_NAME}/providers/Microsoft.Network/applicationGateways/agw-${PREFIX}"
LAW_ID="/subscriptions/${SUB_ID}/resourceGroups/${RG_NAME}/providers/Microsoft.OperationalInsights/workspaces/law-${PREFIX}-dev"

# az rest で明示的にAPIバージョン指定
az rest --method get --url "https://management.azure.com${LAW_ID}?api-version=2023-09-01" --query "properties.customerId" -o tsv
```

## 推奨対応
- Lab03 Step 7, Step 9 の手順に「`az network application-gateway show` が API バージョンエラーになる場合の回避策」を追記する
- または Azure CLI を最新版にアップデートすることを前提条件に追加する

## 環境
- Azure CLI: 2.85.0
- OS: Windows (Git Bash)
- サブスクリプション: ME-MngEnvMCAP062166-tkanazawa-1
