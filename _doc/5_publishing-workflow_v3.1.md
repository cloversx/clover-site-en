* * *

publishing-workflow.md
======================

Clover Publishing Workflow（公開・バージョン運用規定）
----------------------------------------

本ドキュメントは、以下の正典サイト（GitHub Pages）における公式な公開・更新フローを定義する。

*   clover-site-en
*   clover-site-ja

本運用は、思想（および創作物を含む書き物）の長期保存・透明性・整合性を最優先とする。

本サイトのURLは「表示名（Library 等）」から独立しており、将来表示名を変更してもURLは維持される。

* * *

1.  基本原則  
    ===========
2.  正典URL（`/0/<dir-name>/`）は常に最新バージョン（現行版）を指す。
3.  過去バージョン（アーカイブ：`/0/<dir-name>/vN/`）は絶対に変更しない。
4.  バージョン番号（`version`）は「思想段階（段階的進化）」を表す。
5.  軽微修正ではバージョン番号を上げない（Minor updates として記録する）。
6.  思想的に別物となった場合は新しいディレクトリ（新 `<dir-name>`）で公開する（Revisited）。

補足（識別子の分離）：

*   URLはディレクトリ構造（`page.url`）で決まる。
*   front matter の `content_id` は **URL生成には使わない論理識別子**。
*   運用上の混乱を避けるため、通常は `content_id` を `<dir-name>` と一致させる（推奨）。
*   アーカイブ化しても `content_id` は変更しない（全バージョンを貫く論理ID）。

* * *

2.  コンテンツ・ライフサイクル  
    =====================

2.1 新規公開（Version 1）
-------------------

パス（現行版）：

```
/0/<dir-name>/
```

例：

```
/0/ai-evaluation-structure/
```

front matter（最小要点）：

*   layout: content
*   is_archive: false
*   version: 1
*   content_id: （推奨：dir-nameと一致）
*   first_published: YYYY-MM-DD
*   last_updated: YYYY-MM-DD（初回は first_published と同一）

初回公開時は `first_published` と `last_updated` を同日にする。

* * *

2.2 軽微修正（バージョンは繰り上げない）
----------------------

対象例：

*   誤字修正
*   文言の微調整
*   レイアウト修正
*   意味を変えない補足
*   リンク修正（内容の思想段階が変わらない）

ルール：

*   アーカイブを作成しない
*   version を変更しない
*   last_updated のみ更新する
*   Changelog（本文内セクション）に Minor updates として記録する

Changelog記載例：

```
Changelog:

Version 1 (2026-02-06): First published
- 2026-03-01: Minor typo corrections
```

Version番号は変更しない。

Changelogは、コンテンツの思想段階（Version）と、
その段階内で行われた軽微修正（Minor updates）を記録するための履歴である。

Version行は思想段階の確定点を示し、
Minor updatesはその段階内で行われた非本質的修正を時系列で記録する。

Version番号は front matter の version と常に一致させる。

* * *

2.3 重大更新（バージョンを繰り上げる）
---------------------

対象例：

*   論点の追加
*   構造再編
*   新セクション追加
*   分析の大幅拡張
*   思想の再整理（同一テーマのまま深掘り）
*   （創作の場合）同一作品としての「段階的な大改稿」

この場合、現行版を凍結保存してから現行版を更新する。

* * *

### 手順1 — 現行版をアーカイブ化（凍結保存）

コピー：

```
/0/<dir-name>/index.md
↓
/0/<dir-name>/vN/index.md
```

※ N は「これまでの現行版の version」  
例：現行が version: 1 なら v1 を作る。

アーカイブ側 front matter 修正（最低限）：

*   is_archive: true
*   version: 変更しない（例：1のまま）
*   last_updated: 変更しない（凍結した日付ではなく、その版の最終状態の日付を保持）
*   content_id: 変更しない

canonical_url について：

*   原則、`canonical_url` は **空/未指定のままでよい**（テンプレートで `page.url` から自動生成され、`/vN/` を含む canonical になる）。
*   例外（ドメイン移転・構造変更・SEO統合など）の場合のみ `canonical_url` を手入力する。

alt_url / summary について（アーカイブ時）：

*   alt_url は **基本的に変更しない**（テンプレ側で `is_archive == true` のとき hreflang/相互リンクを出力しない前提）。
    *   明快さ優先で「アーカイブは alt_url を空」に統一してもよい（どちらでも整合は取れる）。
*   summary も **基本的に変更しない**。
    *   運用上の理由で区別したい場合は `"Archived snapshot of Version N."` などに変更してもよい。

アーカイブは今後一切編集しない。

* * *

### 手順2 — 現行版を更新（Version を繰り上げる）

`/0/<dir-name>/index.md` 側：

*   is_archive: false のまま
*   version: N+1
*   last_updated: 更新日
*   first_published: 変更しない
*   content_id: 変更しない

Changelogに追加：

```
Changelog:

Version 1 (2026-02-06): First published
- 2026-03-01: Minor typo corrections

Version 2 (2027-05-10): Expanded argument and added dialogue logs
- 2027-06-01: Minor typo corrections
```

原則：

*   各 Version の下に、その Version 期間内の軽微修正（Minor updates）をぶら下げる。
*   Minor updates の記述は、versionを上げない限り増えていく。

* * *

3.  Revisited（思想的再構成 / 別物化）  
    ================================

思想的に別物となった場合（同一テーマでも結論・焦点が大きく変わる等）：

*   新しいディレクトリ（新 `<dir-name>`）を作成
*   Version 1 から開始
*   旧コンテンツは保持（過去の思想状態として価値がある）
*   必要に応じて相互リンク（「Revisited版はこちら」等）

Revisited は履歴の上書きではない。

* * *

4.  日付規則  
    ===========
*   first_published = その `<dir-name>`（論理的同一コンテンツ）の最初の公開日（固定）
*   last_updated = 現行版の最終更新日（更新ごとに変更）
*   アーカイブの first_published / last_updated は凍結（以後更新しない）

Version 2以降でも first_published は変更しない。

* * *

5.  言語運用  
    ===========
*   EN/JA はリポジトリで分離される。
*   alt_url は **現行版のみ**意味を持つ（アーカイブでは出力しない運用）。
*   hreflang の self（自己言語）は **repo の site.lang が真実**であり、テンプレートは `site.lang` を信頼する。
    *   これにより front matter の lang 記載ミスで hreflang が壊れる事故を防ぐ。
*   front matter の lang は推奨：
    *   将来の検索/フィルタ/データ整合のための「ページ属性」として有用。
    *   未指定の場合は表示ロジック等で `site.lang` にフォールバック可能。

* * *

6.  canonical 規則  
    =================
*   現行版 canonical → `/0/<dir-name>/`
*   アーカイブ canonical → `/0/<dir-name>/vN/`
*   Revisited canonical → 新しい `/0/<new-dir-name>/`

canonical_url は通常は空/未指定でよい（自動生成）。  
例外（移転・統合等）の場合のみ手入力する。

* * *

7.  公開前チェックリスト  
    ================
*   `<dir-name>` の確認（小文字・ハイフン推奨・一意）
*   content_id の確認（推奨：dir-nameと一致、かつ全バージョンで不変）
*   version 確認
*   first_published 確認（固定）
*   last_updated 確認（更新日）
*   重大更新時：
    *   `/vN/` のアーカイブ作成済み
    *   アーカイブ側 is_archive: true
    *   アーカイブ側 version が凍結（Nのまま）
*   canonical の確認（現行/アーカイブそれぞれ self canonical）
*   hreflang の確認（現行のみ）
*   RSS feed 反映確認
*   sitemap 反映確認

* * *

8.  運用思想  
    ===========

本サイトは完成品の展示ではない。

思考と制作の進化を保存するための正典アーカイブである。

Version は「思想の層（段階）」を表し、  
軽微修正はその層の内部（Minor updates）で記録される。

* * *

評価：

✔ SEO的に安定  
✔ 長期保存に強い  
✔ 思想進化が可視化される  
✔ 将来の自分が迷わない

* * *
