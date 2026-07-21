# タグ未設置ページからGA4へ一時的にpage_viewを送る

## まず結論

タグ未設置ページから 1 回だけ GA に疎通確認したいなら、`analytics.js` より `gtag.js` を一時的にブラウザで読み込んで `page_view` を送る方が現実的です。`api_secret` を取れない場合、GA4 の `Measurement ID` だけで試せる方法として使いやすいです。

## これは何か

Google Tag Manager や既存タグが入っていないページで、GA4 に一時的な `page_view` を送って後続連携の疎通を確認するための実務メモです。Measurement Protocol と違い `api_secret` は不要ですが、正式な本番実装ではなくブラウザでの一時検証向けです。

## どこで使うか

- 対象ドメインに GTM や GA タグが入っていないが、GA4 連携が動くかだけ確認したいとき。
- ベンダーや後続システムが「GA4 に `page_view` が入るか」を前提に動くか切り分けたいとき。
- Measurement Protocol の `api_secret` を取得できず、ブラウザ側だけでまず試したいとき。

## 全体像

- `analytics.js` は Universal Analytics 向けの旧方式で、現在の GA4 前提の検証には向きにくい。
- GA4 では `gtag.js` を読み込み、`config` で初期化し、`event` で `page_view` を送る。
- タグ未設置ページでも、ブラウザの Console から一時的に `gtag.js` を注入すれば疎通確認はできる。
- Measurement Protocol は HTTP 直送だが `api_secret` が必要なので、権限がない場面では採りにくい。

## よくある疑問

### Q. `Measurement ID` と `api_secret` は何が違う？

A. `Measurement ID` は送信先のWebストリーム識別子、`api_secret` は Measurement Protocol 用の認証情報です。今回の方法では `api_secret` は使いません。

### Q. 一時確認なら、なぜ `analytics.js` ではなく `gtag.js` なの？

A. `analytics.js` は UA 向けの旧方式で、現在の GA4 前提の疎通確認には向きにくいからです。

### Q. `204` が返れば、全部成功？

A. まだ十分ではありません。`Network` で送信できても、GA4 の `Realtime` や後続システムでの反映確認が必要です。

### Q. この方法をそのまま本番実装にしてよい？

A. よくありません。これは一時検証向けで、本番では GTM / gtag / Measurement Protocol のどれを正式採用するか別途決めます。

## 実務での見方

1. 対象の GA4 Web ストリームの `Measurement ID` を確認する。
2. 対象ページをブラウザで開き、開発者ツールの Console を開く。
3. 下のコードを貼り付けて実行する。
4. `Network` タブで `g/collect` が飛んでいるか確認する。
5. GA4 の `Realtime` と後続システム側で反映を確認する。

```javascript
(function () {
  const measurementId = 'G-XXXXXXXXXX';

  window.dataLayer = window.dataLayer || [];
  window.gtag = window.gtag || function(){ dataLayer.push(arguments); };

  const s = document.createElement('script');
  s.async = true;
  s.src = 'https://www.googletagmanager.com/gtag/js?id=' + measurementId;

  s.onload = function () {
    gtag('js', new Date());

    gtag('config', measurementId, {
      send_page_view: false
    });

    gtag('event', 'page_view', {
      page_location: location.href,
      page_title: document.title,
      page_path: location.pathname + location.search + location.hash
    });

    console.log('page_view sent to', measurementId);
  };

  document.head.appendChild(s);
})();
```

## 次回の確認

- [ ] 対象の `Measurement ID` が正しいか。
- [ ] ブラウザの `Network` で `g/collect` が飛んだか。
- [ ] GA4 の `Realtime` で `page_view` が見えたか。
- [ ] 後続システムの取り込み条件が `page_view` のみか、追加条件があるか。
- [ ] 本番実装では GTM / gtag / Measurement Protocol のどれを正式採用するか。

## 関連トピック

- [分析設計の基本プロセス](./分析設計の基本プロセス.md)

## 参考リンク

- [Measure pageviews | Google Analytics](https://developers.google.com/analytics/devguides/collection/ga4/views)
  - GA4 で `page_view` を送る基本と、手動 pageview 時の重複注意を確認する。
- [Configuration | Google Analytics](https://developers.google.com/analytics/devguides/collection/ga4/reference/config)
  - `send_page_view`、`page_location`、`page_title` などの設定項目を確認する。
- [UA→GA4 Tips for switching from analytics.js to gtag.js](https://support.google.com/analytics/answer/10271001)
  - `analytics.js` と `gtag.js` の役割差分を確認する。
- [Measurement Protocol | Google Analytics](https://developers.google.com/analytics/devguides/collection/protocol/ga4)
  - `api_secret` が必要な別方式であることを確認する。

## 更新履歴

- 2026-06-02: QA中心で復習しやすい構成へ再整理。
- 2026-06-01: 初版作成。
