# Snowflakeウェアハウスの世代

## まず結論

Snowflakeの `gen 1 / gen 2` はwarehouseのサイズではなく、標準warehouseが使う計算基盤の世代です。サイズと世代を別の改善軸として考えます。

## これは何か

`GENERATION = '1'` または `GENERATION = '2'` で、標準warehouseが使う compute resources の世代を指定する考え方を整理したトピックです。

## どこで使うか

- warehouseの性能改善候補を整理するとき。
- サイズ変更と世代変更を比較するとき。
- 新しいwarehouseを作成または変更するとき。

## 全体像

- `gen 1`
  - 従来世代の標準warehouse。
- `gen 2`
  - より新しい基盤ハードウェアとソフトウェア最適化を使う標準warehouse。
- `SMALL / LARGE`
  - 計算資源の規模。
- `gen 1 / gen 2`
  - 同じ規模の中で使う基盤世代。

## よくある疑問

### Q. `gen 1 / gen 2` と `SMALL / LARGE` は何が違う？

A. `SMALL / LARGE` は計算資源の規模、`gen 1 / gen 2` はその規模の中で使う基盤世代です。規模を上げる話と、同じ規模で基盤を変える話を分けて考えます。

### Q. 遅いなら、まず `gen 2` にすればよい？

A. それだけでは弱いです。まず SQL やデータ量の問題かを確認し、そのうえで同サイズの世代変更で改善余地があるかを見ます。

### Q. どのwarehouseでも `gen 2` を使える？

A. いいえ。標準warehouseが対象で、リージョンやサイズに制約があります。事前確認が必要です。

### Q. `gen 2` にすると必ずコスト効率が上がる？

A. 必ずではありません。効果は workload ごとに比較します。

## 実務での見方

1. 遅さがSQLやデータ量に起因するか確認する。
2. サイズ不足か、同サイズの世代変更で改善余地があるかを分ける。
3. 対応リージョンとサイズを確認する。
4. 同じworkloadで比較する。

```sql
create warehouse example_wh
  warehouse_size = 'LARGE'
  warehouse_type = 'STANDARD'
  generation = '2';
```

## 次回の確認

- [ ] サイズと世代を分けて検討したか。
- [ ] 対象が標準warehouseか。
- [ ] リージョンとwarehouseサイズが対応しているか。
- [ ] 同じworkloadで効果を比較したか。

## 関連トピック

- [Snowflakeのスピル](./Snowflakeのスピル.md)
- [SnowflakeのCTE再評価](./SnowflakeのCTE再評価.md)

## 参考リンク

- [Snowflake generation 2 standard warehouses](https://docs.snowflake.com/en/user-guide/warehouses-gen2)
  - generation 2 の基本仕様と制約を確認する。
- [CREATE WAREHOUSE](https://docs.snowflake.com/en/sql-reference/sql/create-warehouse)
  - `GENERATION` 句の指定方法を確認する。
- [Behavior change: warehouse commands use GENERATION](https://docs.snowflake.com/en/en/release-notes/bcr-bundles/2026_02/bcr-2225)
  - generation 指定まわりの挙動変更を確認する。

## 更新履歴

- 2026-06-02: QA中心で復習しやすい構成へ再整理。
- 2026-05-31: 初版作成。
