# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

このファイルはリポジトリにコミットされており、所有者・共同編集者どちらの Claude Code も読む共有ドキュメント。個人環境固有の作業メモは `CLAUDE.local.md`（`.gitignore` 済み）に書く。

## このリポジトリについて

作曲・編曲家 **五味礼一郎（Reiichiro Gomi）** のポートフォリオサイト。バンド WELL DONE SABOTAGE のボーカル／ギターとしての活動と、楽曲提供／編曲のクライアントワークを紹介する。

- 所有者: **@ReiichiroGomi**（アーティスト本人・非エンジニア）
- 共同編集者: **@Ryota-Mochizuki-04**（望月さん・開発担当）
- 2026-06 以降は両者とも Claude Code を使って作業する

HTML 内に大量に埋め込まれた日本語コメント（`▼ ここを編集`、`【◯◯】` 見出し、`貼るだけ見本` など）が「非エンジニアの所有者が触る編集 UI」になっている。**コメントは絶対に削らないこと。**

リポジトリ名 `ReiichiroGomi.github.io` は GitHub Pages の User site 規約名で、`main` への push が即 `https://reiichirogomi.github.io/` に反映される。

## ビルド・テスト・実行

ビルドシステムは無い。フレームワーク・パッケージマネージャ・バンドラのいずれも使っていない。素の HTML / CSS / JS のみで、各ページが 1 ファイル完結（CSS は `<style>`、JS は `<script>` でインライン）。

- **プレビュー**: HTML ファイルをブラウザで直接開くだけ（dev サーバ不要）。
- **公開**: `main` ブランチに push すれば GitHub Pages が自動で `https://reiichirogomi.github.io/` に配信。Jekyll 等のビルドは経由しない（素の静的ファイルがそのまま配信される）。
- **更新公開**: ファイルを編集 → コミット → `git push origin main`。反映には数十秒〜数分かかることがある。

## ファイル構成

| ファイル / ディレクトリ | 役割 |
| --- | --- |
| `index.html` | 本番ページ。HTML / CSS / JS 全部入りの単一ファイル（約 2100 行） |
| `404.html` | Not Found ページ。スタンドアロン |
| `favicon.svg` | ファビコン |
| `apple-touch-icon.png` | iOS「ホーム画面に追加」用アイコン（180×180）。iOS が `/apple-touch-icon.png` をルート直下に自動で探しに来る規約があるため、ルート直下が正しい配置 |
| `images/rei.jpg` | ポートレート写真（構造化データ・About セクション・ヒーロー背景で参照） |
| `images/ogp.jpg` | SNS シェア用 OGP 画像（1200×630）。HTML+CSS のテンプレートを Playwright でスクショして生成（テンプレートは共同編集者のローカルにあり） |
| `CLAUDE.md` | このファイル（リポジトリにコミット・共有） |
| `CLAUDE.local.md` | 個人環境固有の作業メモ（`.gitignore` 済み・各自のローカル専用） |

コンテンツ画像は `images/` 配下のみ。アイコン類（`favicon.svg` / `apple-touch-icon.png`）だけは Web の規約上ルート直下に置く。

## アーキテクチャ上の決まり

### グレースフル劣化（GSAP）
末尾で cdnjs 経由の GSAP + ScrollTrigger を読み込む。**「コンテンツはデフォルトで可視」**＋「GSAP が読み込めたときだけ `gsap.set(...)` で hidden state を当てる」 という順序になっているため、ネット不通や CDN 障害でも本文は読める。

- CSS 側（L126〜）で `.reveal` / `.h-anim` の初期状態を hidden にしない。
- GSAP ロード成功時のみ JS（L1885 付近）で `opacity: 0` 等の hidden state を当てる。
- `prefers-reduced-motion: reduce` の判定が JS 冒頭にあり、ループアニメは CSS 側（L1340 付近 `@media`）でも止めている。

### テーマカラー・フォントの単一管理
`<style>` 冒頭の `:root { … }` 内 CSS カスタムプロパティ（`--bg`, `--ink`, `--accent`, `--hero-1`, `--hero-2`, `--font-serif`, `--font-display` …）が配色・タイポの唯一の入口。色やフォントをいじるときはここを書き換える。ハードコードの `#xxxxxx` を散らさない。

### モバイル対応のクセ
- **ヘッダー**: スクロールで `header.scrolled` が付いて背景が出る。**900px 以下では `scrolled` 時の `backdrop-filter` を外している**（L1247 付近）。`backdrop-filter` は包含ブロックを作って `position: fixed` の子要素を壊すため、モバイルのメニューパネル全画面表示が破綻する。ここを安易に戻さないこと。
- **MV カルーセル**: 900px 以下では `.embed-row` を横スクロール＋`scroll-snap-type: x mandatory` に切り替える。`#mvDots` は JS が iframe の本数を見て自動生成し、スクロール位置に応じてアクティブドットを切り替える。**MV を増減しても JS 側を触る必要はない。**
- **楽曲提供セクション**: 901px 以上で `:has(.credit-embed)` を使ってクレジット左／動画右の 2 カラムに切り替える。`:has()` 非対応ブラウザでは縦並びにフォールバック。

### お問い合わせは `mailto:`
バックエンドは無い。`#mailBtn` は `<a href="mailto:gomi.wds@gmail.com?subject=...">` として HTML に直接書かれており（L1755 付近）、JS 不通でも動く。アドレスを変更したい場合は `href` 内の `mailto:` の直後と、必要なら `subject=` を書き換える。

### SEO / OGP
`<head>` 内に `og:*`、`twitter:card`、`application/ld+json`（Schema.org Person）が入っている。`og:image` は横長専用画像 `images/ogp.jpg`（1200×630）の絶対 URL、JSON-LD の `image` は `images/rei.jpg` の絶対 URL を指す。`<title>` には検索流入用キーワード（作曲・編曲・楽曲提供）を含めており、安易に削らない。

### コンテンツ編集パターン
- **楽曲提供アーティスト追加**: `.credit-list` 内に `<div class="credit-row reveal">…</div>` を丸ごとコピー。曲は `<li>曲名<span class="role">役割</span></li>` を `.credit-songs` に追加。
- **バンド作品（リリース）追加**: `.works-list` 内に `<li class="work-item">…</li>` を 1 行コピー。
- **試聴埋め込み**: YouTube / Spotify / Apple Music の公式埋め込み `<iframe>` を既存の `<iframe>` と差し替える（`style` / `allow` 属性は既存に合わせる）。

## 作業時の注意

- **日本語コメント（`▼ ここを編集`、`貼るだけ見本`、`【◯◯】` 見出し、料金表の編集要領 など）は剥がさない**。アーティストの編集 UI そのもの。HTML 構造を編集する前に、付近のコメントを必ず確認すること（コメントの存在自体が「ここは触り方に注意」のシグナル）。
- `class="reveal"` / `class="h-anim"` は GSAP 連携の目印。HTML 構造を編集するときに不用意に外さない。
- `desktop.ini` / `.DS_Store` / `Thumbs.db` は OS が生成するシステムファイル。`.gitignore` 済みなので触らない／追跡しない。
- `main` への push は即本番公開になる。コミット前に index.html をブラウザで開いて確認する。

## レビュー・公開フロー（PR ベース）

`main` へ直接 push せず、**ブランチ → Pull Request → 所有者レビュー → マージ**の流れを基本とする。

- **見た目が変わる変更**: PR 本文に BEFORE / AFTER のスクリーンショットを表で埋め込む。スクショ画像は本番に混ぜないよう、マージしない専用ブランチ（`compare/<トピック>`）にコミットし、`raw.githubusercontent.com` の URL で参照する。レビュー完了後に `compare/*` ブランチは削除してよい（削除すると過去 PR 内の画像は表示されなくなるが、レビューは済んでいるので問題ない）。
- **見た目・動作が変わらない内部改善**: 静止画比較ができないので、「何が変わる／効果／見た目はどう（変わらない）」を箇条書きで説明し、やる・やらないを所有者に判断してもらう。
- PR・Issue とも**非エンジニアが読む前提の平易な日本語**で書く。専門用語を使う場合はひと言補足する。
- 各項目に「👀 ここを見て（判定ポイント）」を付けると、レビューする側が迷わない。

## Issue / PR (GitHub) 運用

- `gh` CLI は @Ryota-Mochizuki-04 のアカウントで認証されている
- 所有者に確認してほしい Issue / PR には **@ReiichiroGomi メンション必須**（しないと通知が届かない、と本人が明言済み）
- カジュアル口調（「〜したよ」「〜お願いします！」など）でOK。過去コメントの口調に合わせる

### Issue 本文のチェックリスト記号の使い分け

- `- [x] N. 項目名` ＝ 対応完了
- `- [ ] N. 項目名` ＝ 未対応（今後やるかも・pending）
- `- [ ] ~~N. 項目名~~ — 理由` ＝ 検討の上やらないと決定（取り消し線で意図を明示）

3つ目の書き方は、所有者からの「やらないってこと分からなくない？」指摘で確立した運用。

## デプロイ完了の自動検知

`main` への push 後、GitHub Pages の反映を待つときは、`curl + grep` で「新しく追加した特徴的なテキスト」が本番URLに現れたか6秒ごとにポーリングする。`run_in_background: true` で Bash を実行すれば、完了通知が自動で来る（ポーリング中はこっちは別のことをしていてOK）。

```bash
for i in $(seq 1 100); do
  if curl -sf https://reiichirogomi.github.io/ 2>/dev/null | grep -q "新しい文字列"; then
    echo "Deployed: attempt $i at $(date -u +%H:%M:%S)Z"
    exit 0
  fi
  sleep 6
done
echo "Timeout: not deployed within 10 minutes" >&2
exit 1
```

判定文字列は「今回 push で追加した文字列」を選ぶ（旧バージョンに含まれていないことが確実なもの）。例: meta description の新フレーズ、新規 iframe の title、新規CSSクラス名など。反映時間目安は **12秒〜数分**（GitHub Pages のキャッシュ次第）。

## コミットメッセージ作法

- 個人名や属人的・主観的表現（「デザイナー視点」「いい感じ」など）は入れない
- 「検討の上 / 反映 / 決定」など客観的・受動的な表現を使う
- description（件名）は `<type>: <description>` 形式（feat / fix / docs / refactor など）、50文字以内、動詞で始める、句読点なし
- body には**「何を変えたか」と「なぜか」をペアで**書く。後から `git blame` で経緯がたどれるように
- 「Item N は検討の上やらないと決定」のような**やらなかった判断も body に記録**する（将来「あれ対応した？」と聞かれた時に履歴で答えられる）
