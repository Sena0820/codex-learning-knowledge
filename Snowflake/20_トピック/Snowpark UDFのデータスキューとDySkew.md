# Snowpark UDFのデータスキューとDySkew

## まず結論

Snowpark UDF は、`行ごとの処理時間の差` と `Python / Java 実行の重さ` のため、少しのデータ偏りでも全体が大きく遅くなりやすいです。DySkew は、この偏りを実行中に観測しながら行単位で再配分し、必要なときだけ動的に負荷を均す仕組みです。

## これは何か

Snowflake の Snowpark UDF 実行で起きる data skew と、その対策として提案された `DySkew: Dynamic Data Redistribution for Skew-Resilient Snowpark UDF Execution` の考え方を整理するトピックです。静的な round-robin 再配分では足りない理由と、Snowpark 向けに何を最適化したのかを戻せるようにします。

## どこで使うか

- Snowpark UDF や Python UDF の実行が、ノード数を増やしても思ったほど速くならないとき。
- Query Profile や実行時間のばらつきから、straggler が起きていそうか考えるとき。
- `まず SQL を直すべきか` `UDF の偏りを疑うべきか` を切り分けるとき。
- Snowflake の内部実行最適化や Snowpark の性能改善事例を理解したいとき。

## 全体像

1. Snowpark UDF は、各行を Python / Java 実行環境へ渡して処理するため、ネイティブ SQL より 1 行の重さが大きいです。
2. そのため、`行数が均等` でも `処理時間が均等` とは限らず、重い行を引いたワーカーが straggler になりやすいです。
3. 以前の static round-robin は、最初に均等配分するだけなので、`行ごとの重さの違い` と `実行中の偏り` を見られません。
4. DySkew は runtime 中の状態を見て、必要なら行を他ワーカーへ再配分します。
5. ただし、大きすぎる行まで積極的に動かすと転送コストで逆に遅くなるため、`偏り` と `行サイズ` を両方見て再配分を制御します。

Snowpark 向けの要点は次の 3 つです。

- `Eager Redistribution`
  - Snowpark UDF は skew の影響が大きいため、最初から再配分を始める方が有利になりやすい。
- `Row Size Model`
  - 巨大 JSON や高解像度画像のような重い行では、転送コストが高すぎるため、batch density を見て再配分を止める。
- `Local Worker も使う`
  - リモート優先でローカルを避けると CPU が遊ぶため、Snowpark では local worker も行き先として使う。

## よくある疑問

### Q. なぜ `均等配分` だけでは足りない？

A. UDF では `1 行あたりの処理時間` が大きくぶれやすいからです。行数が同じでも、あるワーカーだけ重いレコードを多く引くと、そこが全体の完了時刻を決めてしまいます。

### Q. 何を見て skew を検知する？

A. 論文では主に 3 系統です。`行数の偏り` `他ワーカーが idle になっていないか` `同期処理時間の伸び方が他より速すぎないか` を使い、一定回数連続で偏りが出たときだけ再配分します。

### Q. Snowpark では、なぜ最初から再配分した方がよい？

A. Snowpark UDF はネイティブ SQL より 1 行の処理オーバーヘッドが大きく、わずかな偏りでも待ち時間が増えやすいからです。論文では、UDF では再配分の利益が転送コストを上回る場面が多いとしています。

### Q. いつ再配分が逆効果になる？

A. 行が非常に大きいときです。巨大オブジェクトをネットワーク越しに動かすと、処理短縮よりシリアライズと転送の方が高くつくことがあります。論文では、不要な再配分で最大 20 倍の性能劣化も観測しています。

### Q. これは `SQL が遅い問題` とどう違う？

A. これは `UDF 実行の偏り` に強く寄った話です。通常の SQL では scan や join、spill を見ることが多いですが、Snowpark UDF では `ユーザーコードの行ごとの重さ` まで性能差の原因になります。

## 実務での見方

1. Snowpark UDF が遅いときは、まず `普通の SQL 問題` と `UDF の skew 問題` を分けて考える。
2. 前者なら scan、join、spill、warehouse を見る。
3. 後者なら `一部ワーカーだけ極端に遅くなっていないか` `入力行の性質にばらつきがないか` を疑う。
4. 特に semi-structured data、画像、巨大 JSON、条件分岐の多い UDF は skew の影響を受けやすい。
5. `並列度を上げたのに伸びない` ときは、均等分散ではなく straggler が支配している可能性を考える。

## 次回の確認

- [ ] UDF の遅さは、scan や join ではなく行ごとの処理差が原因か。
- [ ] 行数の均等と処理時間の均等を混同していないか。
- [ ] 大きい行を動かしすぎると逆効果になる点を覚えているか。
- [ ] Snowpark では eager redistribution が効きやすい理由を説明できるか。
- [ ] SQL の遅さと UDF skew の遅さを別の論点として切り分けられるか。

## 関連トピック

- [Snowflakeのスピル](./Snowflakeのスピル.md)
- [Snowflake Notebookの頻出スクリプト](./Snowflake%20Notebookの頻出スクリプト.md)
- [Snowflakeウェアハウスの世代](./Snowflakeウェアハウスの世代.md)

## 参考リンク

- [DySkew: Dynamic Data Redistribution for Skew-Resilient Snowpark UDF Execution](https://arxiv.org/pdf/2604.13034)
  - Snowpark UDF の skew 問題と DySkew の設計、評価結果を確認する論文。
- [Snowpark: Performant, Secure, User-Friendly Data Engineering and AI/ML Next To Your Data](https://doi.org/10.1109/ICDCSW63273.2025.00042)
  - Snowpark の実行モデルと、なぜ UDF 実行が独自の性能課題を持つかの前提を押さえる。
- [Creating Automated Optimizations for Python User-Defined Functions with Snowpark's Parallel Execution](https://www.snowflake.com/en/engineering-blog/snowpark-parallel-python-udf-optimization/)
  - DySkew 以前の並列 Python UDF 最適化と round-robin 再配分の背景を確認する。

## 更新履歴

- 2026-06-12: 初版作成。
