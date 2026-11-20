# PoC の準備と実行

## R2 側で事前に行うこと

1. PoC 用の R2 バケットを作成する。
2. バケット設定で Public Development URL (`r2.dev`) を有効にし、表示された URL を控える。
3. 対象バケットに限定した Object Read & Write の R2 API token を作成する。
4. Cloudflare ダッシュボードで S3 API endpoint のアカウント ID を確認する。

このワークフローはバケットを作成しない。Sleet には既存バケットに対する Object Read & Write の権限を設定する。
`r2.dev` を有効にすると、バケット内の全オブジェクトがインターネットから取得可能になる。PoC 専用バケットを使い、NuGet パッケージ以外の機密情報や他用途のオブジェクトを置かない。

## GitHub Environment と Secrets

GitHub リポジトリの **Settings → Environments** で `r2-poc` Environment を作成し、次の保護を設定する。

- Deployment branches and tags を `Selected branches and tags` にし、`main` のみを許可する。
- Required reviewers に承認者を追加する。
- `main` ブランチにはブランチ保護を設定し、Pull Request 経由の変更とレビューを必須にする。

そのうえで、次の値を `r2-poc` Environment の Environment secrets に登録する。リポジトリレベルの Actions secrets には登録しない。値はコード、ログ、通常のワークフロー入力に書かない。

| Secret | 内容 |
| --- | --- |
| `R2_ACCOUNT_ID` | Cloudflare アカウント ID（R2 S3 API endpoint のホスト名に使用） |
| `R2_BUCKET_NAME` | 事前作成した PoC バケット名 |
| `R2_ACCESS_KEY_ID` | バケット限定 R2 API token の Access Key ID |
| `R2_SECRET_ACCESS_KEY` | 同 token の Secret Access Key |
| `R2_PUBLIC_BASE_URI` | `r2.dev` の公開 URL。末尾を `/` にする |

読み取りは公開 URL から匿名で行うため、クライアント用の読み取り資格情報は設定しない。

## 実行

1. GitHub の Actions で `R2 Sleet NuGet PoC` を選ぶ。
2. `Run workflow` を選び、`main` ブランチを指定して手動実行する。
3. 実行ログで package、匿名 restore、Sleet delete、削除後の新しいキャッシュによる取得失敗の各段階を確認する。

R2 の認証情報は Environment 承認後、Sleet の登録・削除ステップにだけ渡される。

ワークフローは NuGet ソースを `R2_PUBLIC_BASE_URI` の `index.json` に限定する。削除後の確認では新しい `NUGET_PACKAGES` と `NUGET_HTTP_CACHE_PATH` を使い、`NU1101` による取得不可を確認する。

## 結果の記録

- R2 Secrets 未設定の状態では R2 疎通確認を行わない。
- 接続時には Sleet の書き込み先と、Sleet がフィードに記録した公開 `index.json` / パッケージ URL が `r2.dev` の実 URL で解決できることを確認する。
- `r2.dev` は開発用途の候補であり、PoC では本番向けの URL・キャッシュ設計を決めない。
- Sleet の削除はフィードからパッケージを取り除く。既存の端末キャッシュに残るコピーを消す操作ではない。
