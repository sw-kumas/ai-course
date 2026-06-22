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

このプロジェクトは `sw-kumas.github.io/ai-course/` のサブディレクトリに配布される。資産パスは相対にしないとルーティングと衝突する：

```bash
# ✅ 正しい — 相対パスでビルド
pnpm exec slidev build slides.md --base ./

# ❌ 誤り — 絶対パスにするとルーターが /ai-course/ai-course/2 になる
pnpm exec slidev build slides.md --base /ai-course/
```

`package.json` の `build` スクリプトには `--base` が含まれていないため、GitHub Pages 用には必ず `pnpm exec slidev build slides.md --base ./` を手動実行すること。

## Deployment

GitHub Pages（ブランチ：`gh-pages`）。手順：

```bash
# 1. ビルド
pnpm exec slidev build slides.md --base ./

# 2. dist を gh-pages にコピーしてプッシュ
git checkout gh-pages
rm -rf * .gitignore
cp -r /path/to/main/dist/* .
git add -A
git commit -m "deploy: update static site"
git push origin gh-pages --force

# 3. main に戻る
git checkout main
```

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

1. **サブディレクトリ配布**：`--base ./` 必須。絶対パスにするとルーターが二重プレフィックスになる。
2. **gh-pages に dist/ をそのまま入れてはいけない**：`dist/` の**中身**をブランチルートに置くこと。
3. **`base` headmatter フィールドは無効**：Slidev の headmatter に `base: /ai-course/` と書いても効かない。CLI の `--base` のみ有効。
4. **開発サーバーのポート**：`pnpm run dev` のポートは `3030` がデフォルトだが、占有時は自動で次（3031, 3032...）になる。
5. **パッケージ追加時**：pnpm の lockfile のみをコミットし、`package-lock.json` は混在させないこと。
