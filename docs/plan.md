# Cloudflare R2 + Sleet NuGet 静的フィード PoC 計画

## 目的

Cloudflare R2 と Sleet を使い、NuGet パッケージを静的フィードとして登録・匿名取得・削除できる最小の構成を検証する。PoC では構成の成立性を確認し、正式リポジトリ向けの詳細設計や品質要求は Phase 4 で検討する。

## 対象範囲と制約

- 代表的な正常系シナリオを1本だけ実行する。
- 異常系の網羅や堅牢化は行わない。必要最低限の失敗確認とログを残す。
- 環境固有の問題や、容易に解決できない問題には深入りしない。
- NuGet.org のような検索掲載は対象外とする。
- フィードの読み取り認証は設けず、匿名の `dotnet restore` を対象とする。
- 検証用のクラスライブラリは最小限とする。
- GitHub Actions は `workflow_dispatch` による手動実行とする。
- R2 接続情報は GitHub Secrets から読み、リポジトリへ値を保存しない。PoC 開始時点ではユーザーへ値を要求しない。
- 公開 URL は `r2.dev` を候補とする。Sleet が生成するフィード URL と公開 URL の一致、匿名アクセス、キャッシュ動作は実接続時に確認する。

## 成功条件

1. GitHub Actions がサンプルクラスライブラリの `.nupkg` を作成し、Sleet で R2 に登録する。
2. 匿名アクセスでサンプルパッケージを `dotnet restore` できる。
3. Sleet で対象パッケージの対象バージョンを削除できる。
4. NuGet パッケージキャッシュと HTTP キャッシュを新しい場所にした `dotnet restore` が、削除後に対象パッケージを取得できないことを確認する。

## 進め方

### Phase 1: PoC 計画

- 対象範囲、制約、成功条件、採用技術を本書に記録する。
- R2 の公開エンドポイント候補は `r2.dev` とし、Sleet の R2 プロバイダー設定で確認する。
- R2 は S3 互換 API を使い、読み取りは公開 URL、書き込みは S3 API とする。

### Phase 2: PoC 実装・検証

- C#/.NET SDK で最小のクラスライブラリと復元確認用コンシューマーを用意する。
- Sleet と Cloudflare R2 を GitHub Actions から操作する `workflow_dispatch` ワークフローを用意する。
- 1回の実行でパッケージ登録、匿名復元、Sleet 削除、独立キャッシュでの取得不可確認を行う。
- 接続情報を用いないビルド・静的確認・ローカル検証を先に行う。
- R2 疎通、実 URL、実際の登録・削除・匿名復元は Secrets 設定後の手動実行で検証する。

### Phase 3: ドキュメント整備

- R2 側の事前設定、GitHub Secrets、ワークフロー実行方法、成功条件を記録する。
- 実行結果と未検証事項を明記し、PoC 完了と R2 疎通確認完了を区別する。

### Phase 4: 正式リポジトリ構築計画

- PoC で得られた結果と制約を整理し、正式リポジトリで必要な構成・運用・品質要求・公開 URL 方針を計画する。
- 正式版の詳細設計や追加要件の実装はこの PoC に持ち込まない。

## 採用技術

- C# / .NET SDK
- Sleet
- Cloudflare R2（S3 互換 API と匿名公開 URL）
- GitHub Actions（手動実行）

## 参照

- [Sleet: Create a Cloudflare R2 feed](https://emgarten.github.io/Sleet/feed-type-cloudflare/)
- [Sleet: Publish from CI](https://emgarten.github.io/Sleet/ci-server/)
- [Cloudflare R2: Public buckets](https://developers.cloudflare.com/r2/buckets/public-buckets/)
