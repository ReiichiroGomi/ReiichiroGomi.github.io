# CLAUDE.md

このリポジトリで作業する Claude Code 向けの共有ドキュメント（コミット済み・所有者/共同編集者の両環境で読まれる）。個人環境固有の作業メモは `CLAUDE.local.md`（`.gitignore` 済み）へ。

## このリポジトリについて

作曲・編曲家 **五味礼一郎（Reiichiro Gomi）** のポートフォリオサイト。バンド WELL DONE SABOTAGE の活動と、楽曲提供／編曲のクライアントワークを紹介する。

- 所有者: **@ReiichiroGomi**（アーティスト本人・非エンジニア）／共同編集者: **@Ryota-Mochizuki-04**（開発担当）。両者とも Claude Code で作業する
- HTML 内の大量の日本語コメント（`▼ ここを編集`、`【◯◯】` 見出し、`貼るだけ見本` など）は「非エンジニアの所有者が触る編集 UI」。**絶対に削らないこと**
- リポジトリ名は GitHub Pages User site の規約名。`main` への push が即 `https://reiichirogomi.github.io/` に反映される

## 技術スタック・ビルド・実行

ビルドシステム・フレームワーク・パッケージマネージャは一切無い。素の HTML/CSS/JS で各ページ 1 ファイル完結（CSS は `<style>`、JS は `<script>` でインライン）。外部依存は cdnjs の GSAP 3.12.5 + ScrollTrigger（アニメーション用 CDN）と Google Fonts（Playfair Display / Shippori Mincho）のみ。

- **プレビュー**: HTML ファイルをブラウザで直接開く（dev サーバ不要）
- **公開**: `main` へ push → GitHub Pages が自動配信（Jekyll 非経由・反映は数十秒〜数分）
- テスト・Lint・CI は無い。コミット前にブラウザで目視確認する

## ファイル構成

| パス | 役割 |
| --- | --- |
| `index.html` | 本番ページ。HTML/CSS/JS 全部入りの単一ファイル（約 2100 行） |
| `404.html` | Not Found ページ（スタンドアロン） |
| `favicon.svg` / `apple-touch-icon.png` | アイコン類。Web 規約上ルート直下固定（iOS は `/apple-touch-icon.png` を自動で探しに来る） |
| `images/rei.jpg` | ポートレート写真（構造化データ・About・ヒーロー背景で参照） |
| `images/ogp.jpg` | SNS シェア用 OGP 画像（1200×630）。HTML テンプレを Playwright スクショで生成（テンプレは共同編集者ローカル） |
| `.github/ISSUE_TEMPLATE/feedback.md` | 非エンジニア向け Issue テンプレ（@メンション入り） |

コンテンツ画像は `images/` 配下のみに置く。

## アーキテクチャの要点

行番号は編集でズレるため、以下のキーワードで grep して該当箇所を探すこと。

### グレースフル劣化 + FOUC 対策（`js-loading`）
`<html class="js-loading">` が付いている間だけ `.reveal` / `.h-anim` を CSS で隠し、JS が外すと同時に GSAP アニメを開始する。フェイルセーフが 3 重: (1) IIFE 内の各分岐で `js-loading` を除去 (2) `</footer>` 直後の 2 秒 `setTimeout` 保険 (3) `<noscript>` で hidden 打ち消し。CDN 障害・JS 無効でも本文は必ず表示される設計。この構造（除去タイミングの順序含む）を壊さない。

### テーマカラー・フォントの単一管理
`<style>` 冒頭の `:root { … }` 内 CSS カスタムプロパティ（`--bg`, `--ink`, `--accent`, `--hero-1/2`, `--font-serif/display` …）が配色・タイポの唯一の入口。ハードコードの `#xxxxxx` を散らさない。

### モバイル対応のクセ
- **ヘッダー**: 900px 以下では `header.scrolled` 時の `backdrop-filter` を外している（`backdrop-filter: none` 付近の英語コメント参照）。`backdrop-filter` は包含ブロックを作り `position: fixed` の全画面メニューを壊すため。安易に戻さない
- **MV カルーセル**: 900px 以下で `.embed-row` を横スクロール + `scroll-snap-type: x mandatory` に切替。`#mvDots` は JS が iframe 本数から自動生成。**MV を増減しても JS は触らない**
- **楽曲提供セクション**: 901px 以上で `:has(.credit-embed)` により 2 カラム化。`:has()` 非対応ブラウザは縦並びフォールバック
- `prefers-reduced-motion: reduce` は JS 冒頭の判定と CSS `@media` の両方で対応済み

### お問い合わせは `mailto:`（バックエンド無し）
`#mailBtn` は `<a href="mailto:gomi.wds@gmail.com?subject=...">` として HTML に直書き（JS 不通でも動く）。宛先変更は `href` 内を書き換える。

### SEO / OGP
`<head>` に `og:*`、`twitter:card`、JSON-LD（Schema.org Person）あり。`og:image` は `images/ogp.jpg`、JSON-LD の `image` は `images/rei.jpg` の絶対 URL。`<title>` の検索キーワード（作曲・編曲・楽曲提供）を安易に削らない。

### コンテンツ編集パターン
- 楽曲提供アーティスト追加: `.credit-list` 内の `<div class="credit-row">` を丸ごとコピー（`reveal` クラス無し。GSAP が `.credit-row` を直接アニメ対象にする）。曲は `.credit-songs` に `<li>曲名<span class="role">役割</span></li>`
- バンド作品（リリース）追加: `.works-list` に `<li class="work-item">` を 1 行コピー
- 試聴埋め込み: YouTube / Spotify / Apple Music の公式埋め込み `<iframe>` を既存と差し替え（`style` / `allow` 属性は既存に合わせる）

## 作業時の注意

- 日本語コメントは剥がさない。HTML 構造を編集する前に付近のコメントを必ず確認（コメントの存在＝「触り方注意」のシグナル）
- `class="reveal"` / `class="h-anim"` は GSAP 連携の目印。不用意に外さない
- `desktop.ini` / `.DS_Store` / `Thumbs.db` は OS 生成ファイル。`.gitignore` 済みなので触らない
- `main` への push は即本番公開。コミット前にブラウザで確認

## レビュー・公開フロー（PR ベース）

`main` へ直接 push せず、**ブランチ → Pull Request → 所有者レビュー → マージ**が基本。

- **見た目が変わる変更**: PR 本文に BEFORE / AFTER スクショを表で埋め込む。画像はマージしない専用ブランチ `compare/<トピック>` に置き `raw.githubusercontent.com` の URL で参照（レビュー完了後にブランチ削除可）
- **見た目が変わらない内部改善**: 「何が変わる／効果／見た目への影響（無し）」を箇条書きし、やる・やらないを所有者に判断してもらう
- PR・Issue は非エンジニアが読む前提の平易な日本語で書く。専門用語にはひと言補足。各項目に「👀 ここを見て（判定ポイント）」を付けるとレビューしやすい

## Issue / PR (GitHub) 運用

- `gh` CLI は @Ryota-Mochizuki-04 のアカウントで認証されている
- 所有者に確認してほしい Issue / PR には **@ReiichiroGomi メンション必須**（しないと通知が届かない、と本人が明言済み）
- カジュアル口調（「〜したよ」など）OK。過去コメントの口調に合わせる
- チェックリスト記号: `- [x]`＝対応完了 / `- [ ]`＝未対応（pending） / `- [ ] ~~項目~~ — 理由`＝検討の上やらないと決定（取り消し線で意図を明示。所有者からの指摘で確立した運用）

## デプロイ完了の自動検知

`main` への push 後は、「今回 push で追加した特徴的な文字列」（旧版に無いことが確実なもの。新フレーズ・新 iframe title・新 CSS クラス名など）を選び、`curl -sf https://reiichirogomi.github.io/ | grep -q "文字列"` を 6 秒間隔でループポーリングする。`run_in_background: true` の Bash で実行すれば完了通知が自動で来る。反映時間の目安は **12 秒〜数分**。

## コミットメッセージ作法

- `<type>: <description>` 形式（feat / fix / docs / refactor など）、50 文字以内、動詞で始める、句読点なし
- 個人名や主観的表現（「デザイナー視点」「いい感じ」など）は入れない。「検討の上／反映／決定」など客観的表現を使う
- body には**「何を変えたか」と「なぜか」をペアで**書く（後から `git blame` で経緯をたどれるように）
- 「Item N は検討の上やらないと決定」のような**やらなかった判断も body に記録**する
