# AGENTS.md

## Project Overview

Slidev プレゼンテーション — LLM（大規模言語モデル）の基礎知識を解説する 8 枚構成のスライドデッキ。

- **自作フレーム**：Slidev（Vite + Vue 3 + Markdown）
- **パッケージマネージャ**：pnpm
- **テーマ**：`@slidev/theme-default`
- **言語**：日本語（スライド本文）＋ 中国語（コードコメント・コミット）
- **配布**：GitHub Pages（`sw-kumas.github.io/ai-course/`）

## Setup Commands

```bash
pnpm install        # 依存をインストール
```

## Development Workflow

```bash
pnpm run dev        # 開発サーバー起動 → http://localhost:3030
pnpm run build      # 本番ビルド → dist/
pnpm run export     # PDF/PPTX エクスポート（要 playwright-chromium）
```

ホットリロード対応。スライド編集時は `pnpm run dev` でブラウザを開き、保存ごとに自動反映。

## Building for GitHub Pages

このプロジェクトは `sw-kumas.github.io/ai-course/` のサブディレクトリに配布される。

```bash
# ビルド（--base で資産パスとルーター base を /ai-course/ に統一）
pnpm exec slidev build slides.md --base /ai-course/
```

`package.json` の `build` スクリプトには `--base` が含まれていないため、
GitHub Pages 用には必ず `pnpm exec slidev build slides.md --base /ai-course/` を手動実行すること。

## Deployment

GitHub Pages（ブランチ：`gh-pages`）。手順：

```bash
# 1. main ブランチでビルド
git checkout main
pnpm exec slidev build slides.md --base /ai-course/

# 2. gh-pages に切り替えて中身を差し替え
git checkout gh-pages
rm -rf * .gitignore
cp dist/index.html .
cp dist/404.html .
cp dist/_redirects .
cp -r dist/assets .

# 3. コミット＆プッシュ
git add -A
git commit -m "deploy: update static site"
git push origin gh-pages --force

# 4. main に戻る
git checkout main
```

⚠️ **絶対に `cp -r dist /tmp/...` してから `cp -r /tmp/.../* .` してはいけない。**
`dist/` がサブディレクトリとしてコピーされるバグがある。必ず `dist/` の**中身のファイル**を 1 つずつルートにコピーすること。

ビルド状況の確認：

```bash
gh api repos/sw-kumas/ai-course/pages --jq '{url: .html_url, status: .status}'
```

## Slide Structure

全 8 スライド。ファイルは **単一** `slides.md`：

| # | タイトル | 特徴 |
|---|----------|------|
| 1 | カバー | `layout: cover`, カスタム CSS |
| 2 | LLMとは | Vue アニメーション（トークン逐次生成） |
| 3 | Tokenとは | 形態素分割の図 |
| 4 | 文字分割の粒度 | バブルアニメーション（魑魅魍魎） |
| 5 | Promptのベストプラクティス | 対比バブル＋食べ物ローテーション |
| 6 | 世界理解＋幻覚 | 3D 回転コイン＋確率??% |
| 7 | コンテキストウィンドウ | 文字流スクロール（破棄 vs 圧縮） |
| 8 | まとめ | 6 カードグリッド |

## Conventions

### Slide Content

- 日本語は子エージェントで自然さを自己審査する（`派个子代理自我审查一下`）
- アニメーションは Vue `<script setup>` + `onSlideEnter`/`onSlideLeave` で制御
- CSS は各スライドの `<style>` にスコープ（Slidev が自動スコープ）
- 配色：`#2563eb`（青）、`#0f766e`（緑）、`#dc2626`（赤/警告）

### Git Commits

Conventional Commits 形式：`<type>: <description>`

| type | 用途 |
|------|------|
| `feat` | 新スライド追加 |
| `fix` | 不具合修正 |
| `chore` | メンテナンス（設定、クリーンアップ） |
| `deploy` | gh-pages 更新 |

### Headmatter

```yaml
theme: default
title: LLMの基礎知識
transition: fade
canvasWidth: 980
aspectRatio: 16/9
```

## Common Gotchas

### Slidev v52.16.0 のバグに注意（最重要）

`@slidev/cli@52.16.0` には [issue #2635](https://github.com/slidevjs/slidev/issues/2635) の regression がある。
`getSlidePath` が base 付きパスを返すのに router history も base を持つため、
ナビゲーション時に base が二重に付与されて `/ai-course/ai-course/2` → 404 になる。
history モードも hash モードも両方死ぬ。

**対策**：`@slidev/cli` を `52.15.2` に正確固定（`package.json` に caret なしで `"52.15.2"`）。
このバージョンは `getSlidePath` が base なしパスを返すので正常に動く。

### GitHub Pages サブディレクトリ配布の正しい手順

1. `@slidev/cli@52.15.2` を使う（↑）
2. ビルド時 `--base /ai-course/`（資産パスもルーター base も正しく /ai-course/ になる）
3. `404.html` は Slidev が `index.html` と同一内容で生成 → そのまま gh-pages ルートに置く
4. 直接 `/ai-course/N` にアクセスすると GitHub Pages は 404 ステータス + `404.html`(=SPA) を返す。
   ブラウザは SPA を読み込み、router が URL を解決して該当スライドを表示。

### gh-pages に dist/ をそのまま入れてはいけない

`dist/` の**中身**（`index.html`, `404.html`, `assets/`）を gh-pages ブランチのルートに置くこと。
`cp -r dist /tmp/foo` → `cp -r /tmp/foo/* .` は `dist/` サブディレクトリごとコピーされるバグがある。
代わりにファイルを 1 つずつコピーする：

```bash
cp dist/index.html .
cp dist/404.html .
cp dist/_redirects .
cp -r dist/assets .
```

### `base` headmatter フィールドは無効

Slidev の headmatter に `base: /ai-course/` と書いても**一切効かない**。
base は CLI の `--base` フラグでのみ設定可能。

### 開発サーバーのポート

`pnpm run dev` のポートは `3030` がデフォルトだが、占有時は自動で次（3031, 3032...）になる。

### パッケージ追加時

pnpm の lockfile（`pnpm-lock.yaml`）のみをコミットし、`package-lock.json` を混在させないこと。
