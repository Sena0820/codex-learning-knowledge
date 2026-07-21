# Snowflakeの復習サマリー

## まず結論

Snowflake を復習するときは、`どう触るか` `どこを読み飛ばせるか` `どこで遅くなるか` `UDFでどこが偏るか` `SQLの書き方をどう誤解しやすいか` `基盤側で何を変えられるか` の順で戻すと整理しやすいです。細かい仕様を個別に追うより、`Notebook -> pruning -> spill -> UDF skew -> CTE -> warehouse` の流れで一周すると、実務の判断に戻しやすくなります。

## この話題で先に押さえること

- まず `Snowflake Notebookの頻出スクリプト` で日常操作の流れを戻す
- 次に `micro-partition と pruning` で scan 量を減らす見方を押さえる
- その後に `スピル` で遅さの兆候を確認する
- その次に `Snowpark UDF の skew` で 並列でも遅くなる理由を押さえる
- `CTE再評価` で SQL の書き方の誤解をほどく
- 最後に `warehouse の世代` で基盤側の改善余地を見る

## 全体像

- Notebook は `何をどう実行するか` の入口です。
- pruning は `どのデータを読まなくて済むか` の入口です。
- スピルは `中間データが大きすぎる` ときの兆候です。
- Snowpark UDF の skew は `行数が均等でも処理時間は均等とは限らない` という注意点です。
- CTE は `見た目を整理しただけで速くなるとは限らない` という注意点です。
- warehouse は `SQL を直した後でも残る負荷をどう吸収するか` の話です。

復習の順番:
1. Notebook で操作の流れを思い出す。
2. pruning で scan 削減の考え方を戻す。
3. スピルで Query Profile の見方を戻す。
4. Snowpark UDF の skew で straggler の見方を戻す。
5. CTE で SQL の誤解を外す。
6. warehouse で基盤側の選択肢を整理する。

## よくある疑問

### Q. Snowflake が遅いとき、最初に何から見るべきか

A. まず `読みすぎていないか` と `中間データが膨らんでいないか` を見ます。前者は pruning、後者はスピルが入口です。

### Q. SQL を直すのと warehouse を変えるのは、どちらが先か

A. 先に SQL と中間データ量を見ます。不要な scan や膨張した JOIN を残したまま基盤だけ強くしても、根本原因が残りやすいです。

### Q. Snowpark UDF だけ遅いときは、何を疑うべきか

A. 普通の SQL 問題に加えて、`行ごとの処理時間の偏り` を疑います。並列度を上げても一部ワーカーだけが重くなるなら、UDF skew が支配している可能性があります。

### Q. Snowflake の論点が多くて、どこから復習すればよいか分からない

A. まずこのページで順番を戻し、その後に必要な詳細だけ個別トピックへ進むのが早いです。分からない論点だけ深掘りすると、復習負荷を下げやすいです。

## 詳細を開く場所

- [Snowflake Notebookの頻出スクリプト](../20_トピック/Snowflake%20Notebookの頻出スクリプト.md)
- [Snowflakeのmicro-partitionとpruning](../20_トピック/Snowflakeのmicro-partitionとpruning.md)
- [Snowflakeのスピル](../20_トピック/Snowflakeのスピル.md)
- [Snowpark UDFのデータスキューとDySkew](../20_トピック/Snowpark%20UDFのデータスキューとDySkew.md)
- [SnowflakeのCTE再評価](../20_トピック/SnowflakeのCTE再評価.md)
- [Snowflakeウェアハウスの世代](../20_トピック/Snowflakeウェアハウスの世代.md)

## 図解

- [Snowflake Notebook頻出スクリプトの全体像](../40_図解/Snowflake-Notebookの頻出スクリプト-全体像.png)
- [Snowflakeのmicro-partitionとpruning](../40_図解/Snowflakeのmicro-partitionとpruning.png)

## 更新履歴

- 2026-06-09: 初版作成
- 2026-06-12: Snowpark UDF のデータスキュー論点を追加
