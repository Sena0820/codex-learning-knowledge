# Snowflake CTE再評価の参照メモ

## まず結論

CTEの定義と再帰CTEの評価方法を確認し、非再帰CTEの性能は実行計画で判断するための参照メモです。

## 用途

- CTEとtemp tableの違いを説明する。
- 公式説明と、Query Profileで確認すべき解釈を分ける。

## 参照先

- [Working with CTEs](https://docs.snowflake.com/en/user-guide/queries-cte)
  - CTEがnamed subqueryであり、一時ビューのように扱われることを確認する。
  - 再帰CTEが反復評価されることを確認する。
- [Working with Subqueries](https://docs.snowflake.com/en/user-guide/querying-subqueries.html)
  - サブクエリの評価単位を確認する。

## 解釈上の注意

- 非再帰CTEが必ず一度だけ物理計算されるとは限らない。
- 実際の性能は、Query Profileと比較実験で確認する。

## 関連トピック

- [SnowflakeのCTE再評価](../20_トピック/SnowflakeのCTE再評価.md)

## 更新履歴

- 2026-06-01: 参照メモの標準形式へ再整理。
- 2026-06-01: 初版作成。
