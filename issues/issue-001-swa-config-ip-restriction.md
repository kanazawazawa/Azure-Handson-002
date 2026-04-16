# Issue #001: staticwebapp.config.json の IP 制限により Lab02 Step 7 の動作確認が失敗する

## 発見箇所
- **Lab**: Lab02 Step 7 (Linked Backend 経由の動作確認)
- **ファイル**: `src/web/staticwebapp.config.json`

## 現象
SWA 経由で `/api/health` にアクセスすると **403 Forbidden** が返る。

```
curl -s "https://victorious-sky-01b725100.7.azurestaticapps.net/api/health"
→ 403: Forbidden (You don't have permissions for this page.)
```

## 原因
`src/web/staticwebapp.config.json` に以下の IP 制限が設定されている:

```json
"networking": {
  "allowedIpRanges": ["10.0.0.0/24"]
}
```

この設定により VNet 内（10.0.0.0/24）からのアクセスのみ許可され、パブリックインターネットからのアクセスが拒否される。

## 手順書との矛盾
Lab02 の Step 1 に以下の記載がある:

> この時点では SWA はパブリックアクセス可能です。Lab03 で Application Gateway + WAF + Private Endpoint を構成し、ネットワークレベルでのアクセス制限を追加します。

Lab02 時点ではパブリックアクセス可能であるべきだが、`staticwebapp.config.json` に最初から IP 制限が入っているため矛盾している。

## 修正案
以下のいずれかの対応が必要:

1. **推奨**: Lab02 用の `staticwebapp.config.json` では `networking` セクションを含めず、Lab03 で IP 制限を追加する手順にする
2. Lab02 の手順書に「動作確認前に `networking` セクションをコメントアウトまたは削除する」旨を追記する

## 影響範囲
- Lab02 Step 7: SWA 経由の API 動作確認が不可
- Lab02 Step 7: ブラウザでのフロントエンド表示確認が不可
