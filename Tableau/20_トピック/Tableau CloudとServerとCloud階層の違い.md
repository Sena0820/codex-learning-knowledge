# Tableau CloudとTableau Server、クラウド基盤の関係

## まず結論

Tableau CloudはTableauが運用するSaaS、Tableau Serverは利用者側で運用する自己管理版です。Cloudの階層は `tenant > site > project > content` と捉えると整理しやすくなります。

## これは何か

Tableau製品の提供方式と管理単位を、運用責任と階層構造の2軸で整理したトピックです。

## どこで使うか

- Tableau環境の構成を説明するとき。
- CloudとServerのどちらが向くか考えるとき。
- `site` と `project` の役割を整理するとき。

## 全体像

- Tableau Cloud
  - Tableauがホスティング、アップグレード、基盤運用を担う。
  - 利用者は主に `site` 単位で管理する。
- Tableau Server
  - 利用者側がサーバー全体を運用する。
  - インフラ、認証、接続、カスタマイズを広く制御する。
- Tableau Cloudの階層
  - 管理階層: `tenant > site`
  - コンテンツ階層: `site > project > nested project > workbook / data source / flow / view`

## よくある疑問

### Q. Tableau Cloud と Tableau Server の違いは何？

A. 本質的な違いは、基盤の運用責任を誰が持つかです。Cloud は Tableau 側、Server は利用者側が広く持ちます。

### Q. `site` と `project` は何が違う？

A. `site` は大きな作業空間の分離単位、`project` はその中の整理棚であり権限モデルの中心です。

### Q. Cloudなら、接続まわりも全部気にしなくてよい？

A. そうではありません。Cloudでも、プライベートネットワーク上のデータ接続には Bridge が必要になる場合があります。

### Q. Server はオンプレミス専用？

A. 専用ではありません。パブリッククラウド上でも運用できます。重要なのは設置場所より、誰が運用責任を持つかです。

## 実務での見方

- Cloudを優先しやすい場面
  - 基盤運用を軽くし、標準機能で素早く展開したい。
- Serverを検討する場面
  - 認証、ネットワーク、接続方式を細かく制御したい。
- Cloud設計で先に考えること
  - `site` を増やす前に、`project` の階層と権限を設計する。

## 次回の確認

- [ ] CloudとServerの違いを、運用責任で説明できるか。
- [ ] `tenant` `site` `project` `content` の順序を説明できるか。
- [ ] Bridgeが必要になる場面を確認したか。

## 関連トピック

- [データ可視化の分野まとめ](../10_分野まとめ.md)
- [Tableau CloudとTableau Serverの参照メモ](../30_参考リンク/Tableau CloudとServerの参考リンク.md)

## 参考リンク

- [Tableau CloudとTableau Serverの参照メモ](../30_参考リンク/Tableau CloudとServerの参考リンク.md)

## 更新履歴

- 2026-06-02: QA中心で復習しやすい構成へ再整理。
- 2026-06-01: 旧SVG図への参照を削除。
- 2026-06-01: 初版作成。
