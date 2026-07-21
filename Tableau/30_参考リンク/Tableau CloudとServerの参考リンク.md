# Tableau CloudとTableau Serverの参照メモ

## まず結論

Tableau公式ヘルプから、CloudとServerの違い、およびCloudの階層構造を確認するための参照メモです。

## 用途

- CloudとServerの運用責任を比較する。
- `tenant` `site` `project` の関係を確認する。
- 接続や移行の制約を確認する。

## 参照先

- [Tableau Product Overview](https://help.tableau.com/current/tableau/en-gb/tableau_product_overview.htm)
  - Cloudはhosted、Serverはself-hostedであることを確認する。
- [Use Tableau Cloud Manager](https://help.tableau.com/current/online/en-us/cloud_manager_intro.htm)
  - 最上位のtenantとsitesの関係を確認する。
- [Add, Delete, or Activate Sites](https://help.tableau.com/current/online/en-us/cloud_manager_sites.htm)
  - tenant配下で複数siteを持てることを確認する。
- [Use Projects to Manage Content Access](https://help.tableau.com/current/server/en-us/projects.htm)
  - projectが整理と権限管理の単位であることを確認する。
- [What Can I Do with a Tableau Site?](https://help.tableau.com/current/pro/desktop/en-us/web_author_overview.htm)
  - siteが利用者とコンテンツをまとめる作業空間であることを確認する。
- [Technical Considerations for Migrating from Tableau Server to Tableau Cloud](https://help.tableau.com/current/server/en-us/migrate_server_to_cloud_overview.htm)
  - CloudとServerの差分を比較する。

## 関連トピック

- [Tableau CloudとTableau Server、クラウド基盤の関係](../20_トピック/Tableau CloudとServerとCloud階層の違い.md)

## 更新履歴

- 2026-06-01: 参照メモの標準形式へ再整理。
- 2026-06-01: 初版作成。
