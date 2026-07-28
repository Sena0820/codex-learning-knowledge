# Universal LinksとAndroid App Linksの仕組みとアプリ起動計測

## まず結論

Universal LinksとAndroid App Linksは、対応するHTTPSリンクがタップされたときに、OSがWebとアプリのどちらで開くかを判定する仕組みです。ページ訪問をきっかけにWebサーバーがアプリを強制起動する仕組みではありません。

アプリ起動をWeb版GA4だけで正確に数えるのは難しいため、アプリ側で `リンク受信` と `目的画面表示` をイベント計測するのが基本です。アプリを改修できない範囲だけ、クリック数とWeb到達数の差から推計します。

## これは何か

このトピックでは、次の2点を一続きで整理します。

- iOSのUniversal LinksとAndroid App Linksが、HTTPSリンクをアプリへ渡す仕組み
- GA4などでアプリ起動を実測し、観測できない部分を推計する方法

`ディープリンク` はアプリ内の特定画面を直接開く仕組みの総称です。Universal LinksとAndroid App Linksは、その中でもWebサイトと正規アプリの関係を検証したHTTPSディープリンクです。

## どこで使うか

- Webからアプリへの送客数をKPIにするとき
- Universal LinksやApp Linksの実装要件を整理するとき
- Web版GA4のpage_viewだけでは見えない起動数を分析するとき
- アプリ改修前に、既存データで起動数の概算を作るとき

## 全体像

### リンクを開く流れ

```text
対応するHTTPSリンクをユーザーがタップ
        |
        v
OSがドメインとアプリの関連付け、対象パス、ユーザー設定を確認
        |
        +-- アプリへ振り分け --> URL受信 --> 目的画面を表示
        |
        +-- Webへ振り分け ----> ページ表示 --> GA4のpage_view
```

アプリへ振り分けられた場合、通常はWebページが表示されず、Webページ上のJavaScriptも実行されません。そのため、Webデータストリームの `page_view` だけではアプリ起動を観測できません。

### OS別の関連付け

| 項目 | iOS Universal Links | Android App Links |
|---|---|---|
| Web側ファイル | `apple-app-site-association` | `.well-known/assetlinks.json` |
| アプリ側設定 | Associated Domains | Intent Filterと`android:autoVerify` |
| 主な照合情報 | Team ID、Bundle ID、対象パス | Package名、署名証明書、対象URL |
| アプリ側の受信 | Universal LinkとしてURLを受信 | IntentとしてURLを受信 |
| 主な例外 | Safariで同一ドメイン内を移動するとWebを継続する場合がある | ユーザーが「対応リンクを開く」を無効化できる |

どちらも「アプリがインストール済みなら必ず起動する」とは限りません。関連付けの検証状態、対象パス、リンク元アプリ、ブラウザやWebView、ユーザー設定などの影響を受けます。

### 実測する3段階

```text
deep_link_click
リンクがクリックされた
        |
        v
deep_link_received
アプリがURLを受け取った
        |
        v
deep_link_content_view
目的画面が表示された
```

`deep_link_received` はOSからアプリへの引き渡し成功、`deep_link_content_view` はユーザーが目的のコンテンツまで到達したことを表します。両方を分けると、アプリは開いたがルーティングやログイン要求で目的画面を表示できなかったケースを切り分けられます。

## 理解用イラスト

この図では、リンクタップ後のOSによる振り分けと、実測・推計に使う観測点を一枚で確認できます。

![Universal LinksとAndroid App Linksの仕組みとアプリ起動計測](../40_図解/Universal%20LinksとAndroid%20App%20Linksの仕組みとアプリ起動計測-全体像.png)

## よくある疑問

### Q. 対象ページを訪問すると自動的にアプリが起動するのか

A. 原則は違います。対応リンクをユーザーがタップしたときにOSが振り分けます。ブラウザ内の移動やユーザー設定によっては、アプリが入っていてもWebで開きます。

### Q. Web版GA4のpage_viewがない分を、すべてアプリ起動としてよいか

A. できません。クリック後離脱、通信失敗、計測拒否、広告ブロックなどもpage_view欠損に含まれるためです。

### Q. GA4のsession_startやfirst_openで代用できるか

A. できません。`session_start` は通常起動とディープリンク起動を区別できず、`first_open` はインストールまたは再インストール後の初回利用だけを表します。ディープリンク専用のカスタムイベントを送る方が明確です。

### Q. 関連付けファイルのサーバーアクセス数は起動数になるか

A. なりません。AASAや`assetlinks.json`へのアクセスは、OSによるドメイン検証やキャッシュ更新を含み、リンクタップごとに発生するものではありません。

### Q. Firebase Dynamic Linksの自動イベントを使えばよいか

A. 新規設計では依存しない方が安全です。GA4の `dynamic_link_app_open` などはFirebase Dynamic Links向けであり、同サービスの終了に伴って段階的に廃止されています。

## 実務での見方

### 1. 最初に「起動」の定義を決める

次の3つを混同しないようにします。

- OSがアプリへURLを渡した
- アプリが前面に表示された
- アプリ内の目的画面が表示された

業務KPIとしては、単なる前面表示より `目的画面が表示された回数` を主指標にし、`URL受信回数` を診断指標にするのが分かりやすい設計です。

### 2. 推奨イベントとパラメータ

| イベント | 意味 | 主なパラメータ |
|---|---|---|
| `deep_link_click` | 管理できるリンク元でのクリック | `platform_hint`、`campaign_id`、`link_id` |
| `deep_link_received` | アプリがURLを受信 | `platform`、`link_path_type`、`app_state`、`route_result` |
| `deep_link_content_view` | 目的画面を表示 | `platform`、`content_type`、`campaign_id`、`link_id` |

`link_id` は個人情報を含まない一時的な識別子にします。メールアドレスや会員情報を含む生URLを、そのままGA4へ送らないようにします。

### 3. 基本KPI

```text
アプリ引き渡し率
= deep_link_received / deep_link_click

目的画面表示成功率
= deep_link_content_view / deep_link_received

Webフォールバック率
= Web到達数 / deep_link_click
```

管理できない外部リンクでは全クリックを取得できないため、媒体、キャンペーン、OS、日付などの集計単位で比較します。Webとアプリの個人単位の接続が必要な場合は、同意とプライバシー要件を満たした `user_id` などを別途検討します。

### 4. アプリを改修できない場合の推計

最も単純な推計は次の形です。

```text
推定アプリ振り分け数
≈ 対象リンククリック数 - Web到達数
```

ただし、クリック後離脱やGA4の計測欠損も差分に入るため、そのまま確定値にはできません。

より良い方法は、アプリを開かない通常URLをコントロールとして用意し、通常時のWeb到達率を求めることです。

```text
通常Web到達率 r
= コントロールURLのWeb到達数 / コントロールURLのクリック数

期待Web到達数
= App Linkクリック数 x r

推定アプリ振り分け数
= 期待Web到達数 - App Link先の実Web到達数
```

推計はiOSとAndroidを分け、さらにブラウザ、アプリ内ブラウザ、流入元、アプリ保有が分かる会員群などで層別すると精度を上げやすくなります。

### 5. 導入順序

1. アプリ起動KPIを `URL受信` と `目的画面表示` に分ける。
2. 対象ドメイン、パス、除外パス、遷移先画面の対応表を作る。
3. iOSとAndroidで同じ意味のアプリイベントを実装する。
4. 管理できるリンク元にクリックイベントと共通の`campaign_id`を付ける。
5. GA4またはBigQueryでクリック、受信、画面表示を集計する。
6. 観測できない流入だけ、コントロール方式で補完推計する。

## 次回の確認

- [ ] KPIの「アプリ起動」がURL受信と目的画面表示のどちらか決まっているか
- [ ] iOSとAndroidの対象ドメイン・パスが一覧化されているか
- [ ] cold startとwarm startの両方でイベントが送られるか
- [ ] URL受信後のルーティング失敗を計測できるか
- [ ] Webとアプリで共通の`campaign_id`または安全な`link_id`を使えるか
- [ ] 推計値と実測値を同じ名前で扱っていないか

## 関連トピック

- [GA4のsession_startと他イベントパラメータの見え方](./GA4のsession_startと他イベントパラメータの見え方.md)
- [GA4のセッション数がBigQuery・探索・Looker Studio・Data APIでズレる理由](./GA4のセッション数がBigQuery・探索・Looker%20Studio・Data%20APIでズレる理由.md)

## 参考リンク

- [Apple: Supporting associated domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)
  - iOSアプリとWebサイトの関連付け、AASA、更新タイミングの確認に使う公式資料。
- [Apple: Allowing apps and websites to link to your content](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content/)
  - Universal Linksの基本動作とSafariでの同一ドメイン遷移の例外を確認する公式資料。
- [Android Developers: About App Links](https://developer.android.com/training/app-links/about)
  - Android App Linksの仕組み、検証、Dynamic App Linksを確認する公式資料。
- [Android Developers: Verify App Links](https://developer.android.com/training/app-links/verify-applinks)
  - `assetlinks.json`と端末上の検証状態を確認する公式資料。
- [Google Analytics: Automatically collected events](https://support.google.com/analytics/answer/9234069)
  - `first_open`、`session_start`、`screen_view`、旧Dynamic Links関連イベントの定義確認に使う公式資料。
- [Google Analytics: Event parameters](https://support.google.com/analytics/answer/13675006)
  - カスタムイベントへパラメータを付与して分析する方法を確認する公式資料。

## 更新履歴

- 2026-07-28: 初版作成
