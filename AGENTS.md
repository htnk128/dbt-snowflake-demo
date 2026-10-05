# AGENTS.md

## プロジェクト概要

「Jaffle Shop」の売上分析基盤。dbt (profile: `dbt_snowflake_demo`) でステージング〜マートのデータパイプラインを構築する。

## 接続情報

- connection (profile): `dbt_snowflake_demo`
- database: `DEMO_DBT`
- warehouse: `DEMO_DBT_WH`
- schema (開発): `DEV`
- schema (ソース): `DEV_RAW` (`{{ target.schema }}_raw`)

## セットアップコマンド

- 依存インストール: `dbt deps`
- テスト実行: `dbt test`
- ビルド (全体): `dbt build`
- ビルド (staging のみ): `dbt build --select staging`
- ドキュメント生成: `dbt docs generate && dbt docs serve`

## ソーステーブル (DEMO_DBT.DEV_RAW, source: `ecom`)

| テーブル | 概要 | 主キー | 備考 |
|---|---|---|---|
| raw_customers | 1つ以上の商品を購入した人物ごとに1レコード | id | |
| raw_orders | 注文ごとに1レコード(1つ以上の注文商品で構成) | id | `loaded_at_field: ordered_at` |
| raw_items | 注文に含まれる商品 | id | |
| raw_stores | 店舗マスタ | id | `loaded_at_field: opened_at` |
| raw_products | 店舗で販売される商品のSKUごとに1レコード | id | |
| raw_supplies | 店舗で販売される商品のSKUごとの資材 | - | |

## モデル構成

- staging: `stg_customers` / `stg_locations` / `stg_order_items` / `stg_orders` / `stg_products` / `stg_supplies` (`+materialized: view`)
- marts: `customers` / `locations` / `order_items` / `orders` / `products` / `supplies` (`+materialized: table`)

## 命名規約

- Staging: `stg_{table}.sql` (例: `stg_customers.sql`)
- Marts: モデルが表す名詞そのまま (例: `orders.sql`, `customers.sql`)
- カラム名: snake_case、日本語禁止
- ブール値カラム: `is_` または `has_` プレフィックス (例: `is_food_order`, `is_repeat_buyer`)
- 日付カラム: `_date` サフィックス、タイムスタンプは `_at` サフィックス
- 金額カラム: cents 単位は `_cents` サフィックス、ドル換算後は無サフィックス

## コーディングスタイル

- CTEベースで記述 (サブクエリのネスト禁止)
- CTE名は処理内容を表す名前にする (例: `renamed`, `joined`, `order_items_summary`)
- 最終 `select * from <最後のCTE>` で終える
- JOINは常に `on` 句で結合条件を明記 (`using` 禁止)
- SQLキーワードは小文字 (`select`, `from`, `where`, `with`)
- インデント: スペース4つ
- cents→dollars変換は `{{ cents_to_dollars(column_name) }}` マクロを使用 (独自実装禁止)

## テスト方針

- 全てのマート主キー: `unique` + `not_null` テスト必須
- 外部キー (例: `customer_id`): `relationships` テストで参照整合性を確認
- 全てのstagingモデル: `schema.yml` にドキュメントとテストを定義
- マート間の整合性は `dbt_utils.expression_is_true` で検証 (例: `orders.order_items_subtotal = orders.subtotal`)
- 複雑なロジック (例: フード/ドリンク判定) には `unit_tests` を追加
- モデル追加・変更時は必ず `dbt build --select <model_name>` で検証してから完了とする

## 品質基準

- NULL率 > 5% のカラムはレビュー対象としてフラグ
- 金額カラム (subtotal, tax_paid, order_total): 負の値は異常値として検出
- 日付の未来値 (`> CURRENT_DATE`) は異常値として検出

## セキュリティ考慮事項

- `.snowflake/keys/` 以下の秘密鍵・パスフレーズをコードやコミットに含めない
- `profiles.yml` の認証情報をハードコードしない (`~/.dbt/profiles.yml` を使用)
- 本番環境への直接書き込み禁止 (CI/CDフローを経由すること)

## 禁止事項

- `SELECT *` は使用禁止 (CTE内の `source as (select * from ...)` を除く)
- モデル内でのデータベース名・スキーマ名のハードコード禁止
- `{{ source(...) }}` / `{{ ref(...) }}` 以外での他モデル参照禁止
- マジックナンバー禁止 (dbt varsまたはマクロで名前付き定数として定義)
