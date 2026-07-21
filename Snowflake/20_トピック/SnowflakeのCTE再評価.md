# SnowflakeのCTE再評価

## まず結論

SnowflakeのCTEは、一時テーブルではなく、その文の中で使う名前付きサブクエリです。重いCTEを複数回参照するときは、`一度だけ計算されるはず` と思い込まず、実行計画を確認します。

## これは何か

`WITH` で定義したCTEが、固定結果として必ず保存されるわけではなく、実行計画の中で複数回計算相当に扱われる可能性があることを整理するトピックです。

## どこで使うか

- 重いCTEを複数箇所から参照するとき。
- Query Profileで似た重いノードが複数見えるとき。
- scan、JOIN、集約、ソート、スピルが想定より多いとき。

## 全体像

- 非再帰CTE
  - 可読性を高める名前付きサブクエリ。
  - 一度だけ計算して保存する保証はない。
- 再帰CTE
  - アンカー句の後、再帰句を反復評価する。
  - 反復は仕様そのもの。
- temp table
  - 一度保存し、保存した結果を後で読む。

## よくある疑問

### Q. CTEに切り出したら、一度だけ計算される？

A. そうとは限りません。Snowflakeの非再帰CTEは、まず `名前付きサブクエリ` と捉えます。複数参照時は Query Profile で重複ノードを確認します。

### Q. temp table と何が違う？

A. temp table は `一度保存した結果` を後で読む形です。CTEは保存を保証するものではなく、主目的は可読性や構造化です。

### Q. CTEを使うと速くなる？

A. 自動では速くなりません。見やすくなることはありますが、重い処理を複数回たどる実行計画になるなら遅くなることもあります。

### Q. 何でも temp table にすればよい？

A. それも単純化しすぎです。書き込みコスト、保守対象、再利用価値を含めて比較します。重複計算が重く、temp table化で明確にノードと時間が減るときに検討します。

## 実務での見方

1. 同じベース表scanが複数回出ていないかを見る。
2. 同じJOIN、集約、ソートが複数系列にないかを見る。
3. `operator time` や scan bytes が重複していないかを見る。
4. 必要なら temp table 化し、ノードと実行時間が減るか比較する。
5. 書き込みコストと保守対象の増加も含めて判断する。

```sql
with base as (
  select customer_id, sum(amount) as total_amount
  from orders
  group by customer_id
)
select a.customer_id, a.total_amount, b.total_amount
from base a
join base b
  on a.customer_id = b.customer_id;
```

## 次回の確認

- [ ] 重いCTEを複数回参照していないか。
- [ ] Query Profileで重複ノードを確認したか。
- [ ] 一時テーブル化の効果とコストを比較したか。
- [ ] スピル増加との関係を確認したか。

## 関連トピック

- [Snowflakeのスピル](./Snowflakeのスピル.md)
- [Snowflakeウェアハウスの世代](./Snowflakeウェアハウスの世代.md)

## 参考リンク

- [Working with CTEs](https://docs.snowflake.com/en/user-guide/queries-cte)
  - SnowflakeでのCTEの基本的な位置づけを確認する。
- [Working with Subqueries](https://docs.snowflake.com/en/user-guide/querying-subqueries.html)
  - サブクエリとの関係を確認する。

## 更新履歴

- 2026-06-02: QA中心で復習しやすい構成へ再整理。
- 2026-06-01: 初版作成。
