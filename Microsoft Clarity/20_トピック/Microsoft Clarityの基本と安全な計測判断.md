# Microsoft Clarityの基本と安全な計測判断

## まず結論

Clarity は画面動画をそのまま録画する道具ではなく、ページの構造とユーザー操作を収集して再現・集計する行動分析ツールです。便利ですが、個人情報や一意トークンが見えるページでは、そのまま入れずにマスキングや除外判断が必要です。

## これは何か

Microsoft Clarity の基本的な計測の仕組み、ヒートマップや録画の見え方、カスタムタグの役割、計測してよいページかを判断するための考え方をまとめたトピックです。導入可否を素早く判断したいときの基礎メモとして使います。

## どこで使うか

- ヒートマップやセッション録画を導入したいとき
- 画像が出ない、見た目が崩れる理由を切り分けたいとき
- スタンプラリー、キャンペーン、LP などのページで計測可否を判断したいとき
- GA4 とは別に、行動の迷い方や UI の詰まり方を確認したいとき

## 全体像

- サイトへ Clarity の計測タグを入れると、Clarity は DOM とユーザー操作イベントを収集する
- 収集対象にはクリック、スクロール、マウス移動、ページ遷移、画面サイズ、JS エラーなどが含まれる
- セッション録画は動画保存ではなく、DOM と操作ログから後で再生画面を再構成する
- ヒートマップは複数セッションのクリックやスクロールを集計し、代表スクリーンショットへ重ねて表示する
- カスタムタグを使うと、標準項目だけでは切れない業務文脈を Clarity 側へ渡して分析軸にできる

## よくある疑問

### Q. Clarity は動画を録画しているの？

A. いいえ。Clarity は動画ファイルを保存するのではなく、DOM と操作ログから後で再生画面を再構成しています。

### Q. 入力欄がマスクされるなら安全？

A. それだけでは不十分です。URL パラメータ、一意トークン、会員識別子、応募コードのような情報は別途注意が必要です。

### Q. 画像が出ないのは実装バグ？

A. 実装バグとは限りません。代表スクリーンショット未採用、画像未読込、マスキング、CSS 読込制約などが原因になりえます。

### Q. カスタムタグと Identify API は同じ？

A. 同じではありません。カスタムタグは分析用の分類ラベル、Identify API は同一人物や同一セッションを寄せる識別子に近い役割です。

## 実務での使い方

- まず、計測したいページに個人情報、認証情報、決済情報、一意トークンが見えていないか確認する
- 次に、ログイン後ページ、申込完了ページ、予約確認ページ、マイページのような個人別表示ページを除外候補として洗う
- 画像がヒートマップに出ないときは、`Change screenshot`、画像の lazy load、マスキング設定、外部 CSS の取得条件を確認する
- キャンペーン分析では、`page_group` `campaign` `entry_type` などのカスタムタグを付けて録画やヒートマップを絞り込む
- スタンプラリーページでは、公開ページで個人入力や個別コードがなければ計測しやすいが、URL や表示内容に個別識別子がないかを必ず確認する

## 次回の確認

- [ ] 計測予定ページに個人情報、認証情報、決済情報、一意トークンがないか確認したか
- [ ] URL パラメータに `token` `code` `email` `userId` `entryId` などが含まれていないか確認したか
- [ ] マスキング対象と除外対象を分けて整理したか
- [ ] カスタムタグに入れる値が個人情報や秘密情報になっていないか確認したか
- [ ] 画像が見えないときの切り分け順を共有できる状態か

## 関連トピック

- [分析設計の基本プロセス](../../分析全般/20_トピック/分析設計の基本プロセス.md)
- [タグ未設置ページからGA4へ一時的にpage_viewを送る](../../GA4/20_トピック/タグ未設置ページからGA4へ一時的にpage_viewを送る.md)

## 参考リンク

- [Clarity data collection](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-data)
  - Clarity が DOM と操作イベントをどう収集するかの確認用
- [Frequently asked questions](https://learn.microsoft.com/en-gb/clarity/faq)
  - 録画の仕組み、画像が出ない理由、ページ除外の考え方の確認用
- [Masking content](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-masking)
  - マスキングモード、常時マスク対象、CSS 内データの注意点の確認用
- [Change screenshots](https://learn.microsoft.com/en-gb/clarity/heatmaps/heatmaps-screenshots)
  - ヒートマップの代表スクリーンショット切替の確認用
- [Clarity client API](https://learn.microsoft.com/en-ca/clarity/setup-and-installation/clarity-api)
  - `window.clarity("set", key, value)` によるカスタムタグ付与の確認用
- [Identify API](https://learn.microsoft.com/en-us/clarity/setup-and-installation/identify-api?source=recommendations)
  - custom ID と custom tags の違いの確認用

## 更新履歴

- 2026-06-02: QA型へ再構成
