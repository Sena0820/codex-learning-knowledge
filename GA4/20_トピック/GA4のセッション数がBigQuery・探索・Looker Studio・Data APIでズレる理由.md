# まず結論

GA4 のセッション数がズレる主因は、`どの画面や接続元が元データをどう集計しているか` が違うためです。特に BigQuery は生データから厳密集計しやすく、探索レポートや Data API は GA4 のレポート面に近く、Looker Studio は接続先によって BigQuery 側にも GA4 側にも寄ります。

## これは何か

GA4 の `セッション` 指標を、BigQuery、探索レポート、Looker Studio、Google Analytics Data API で見たときに、なぜ数値が少しずれるのかを整理するトピックです。

## どこで使うか

- GA4 UI と BigQuery の数字が合わず、原因を切り分けたいとき
- Looker Studio のダッシュボードで、接続元ごとの違いを説明したいとき
- API 取得値と画面表示の差を、仕様差として説明したいとき
- セッション数の比較前に、同じ集計面を見ているか確認したいとき

## 全体像

- まず `BigQuery は生データ寄り`、`探索レポートと Data API は GA4 レポート面寄り` と分ける
- 次に `Looker Studio は独自計算ではなく接続元に従う` と押さえる
- そのうえで、差異の原因を `HLL++ による近似` `Google Signals や modeling` `attribution` `sampling` `高カーディナリティ` `データ反映タイミング` に分けて考える
- セッション数の差は `どちらが壊れているか` ではなく、まず `集計方法が違う` と見る

## 理解用イラスト

この図では、`どの面が GA4 の集計済みレポートに近いか` と `どの面が生データに近いか` を一目で戻せます。Looker Studio が単体ではなく、接続元しだいで立ち位置が変わる点も思い出しやすくしています。

![GA4のセッション数がBigQuery・探索・Looker Studio・Data APIでズレる理由](../40_図解/GA4のセッション数がBigQuery・探索・Looker%20Studio・Data%20APIでズレる理由-全体像.png)

## よくある疑問

### Q. BigQuery と探索レポートは、どちらが正しいのか

A. まず `正しさの勝負` ではなく `集計面の違い` と考えるのが安全です。BigQuery は生データから厳密に数えやすく、探索レポートは GA4 のレポート面で使われる近似や付加処理の影響を受けます。

### Q. Data API は探索レポートより BigQuery に近いのか

A. 近いのは BigQuery ではなく、探索レポートや標準レポート側です。Data API は GA4 の reporting surfaces の一部なので、HLL++ や modeling などの影響を受けうる前提で見ます。

### Q. Looker Studio は独自にセッション数を計算しているのか

A. 基本はそう考えません。Looker Studio は表示の器であり、GA4 コネクタなら GA4 レポート面寄り、BigQuery コネクタなら BigQuery 寄りの数値になります。

### Q. 2021 年 10 月の変更で、何が変わったのか

A. よく使う指標の一部で、厳密一意件数ではなく HLL++ による高精度な推定が使われるようになりました。目的は、集計効率や安定性を上げ、エラー率を下げることです。

### Q. どのくらいズレることがあるのか

A. Google の開発者向け説明では、セッション数の近似精度について 95% 信頼区間でおおむね `±1.63%` 程度の差がありえます。完全一致を前提にせず、小さな差は仕様差として扱うのが基本です。

## 実務での見方

- 比較の最初に `同じ日付範囲` `同じタイムゾーン` `同じフィルタ` `同じ接続元` かを確認する
- Looker Studio は `GA4 コネクタか` `BigQuery コネクタか` を最初に確認する
- BigQuery と API/探索の差は、まず `HLL++` `modeling` `Google Signals` `sampling` `反映遅延` を疑う
- 社内説明では `BigQuery は生データ置き場` `Data API は GA4 の取り出し口` `Looker Studio は表示の器` と言い換えると伝わりやすい
- 厳密値が必要な検証や独自ロジック分析は BigQuery、日常的なレポート閲覧やダッシュボード確認は GA4/UI や API 側を使い分ける

## 次回の確認

- [ ] Looker Studio の数字を見る前に、GA4 接続か BigQuery 接続かを確認したか
- [ ] BigQuery と Data API を比べる前に、近似集計と付加処理の差を切り分けたか
- [ ] セッション差異を `異常` ではなく `集計面の差` として説明できるか

## 関連トピック

- [GA4のdirectセッションをBigQueryで調査する](./GA4のdirectセッションをBigQueryで調査する.md)
- [GA4のevent_paramsを生データへ直接クエリするときの考え方](./GA4のevent_paramsを生データへ直接クエリするときの考え方.md)
- [GA4とBigQueryのグラフ分析でユーザー導線を読む](./GA4とBigQueryのグラフ分析でユーザー導線を読む.md)

## 参考リンク

- [Bridge the gap between the Google Analytics UI and BigQuery export](https://developers.google.com/analytics/blog/2023/bigquery-vs-ui)
  - GA4 UI、探索、Data API、BigQuery export の差異要因と HLL++ の説明を確認するためのリンク
- [Google Analytics Data API](https://developers.google.com/analytics/devguides/reporting/data/v1)
  - Data API が GA4 のレポート面の一つであることを確認するためのリンク

## 更新履歴

- 2026-07-07: 初版作成
