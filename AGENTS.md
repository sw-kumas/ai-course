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

**2 段階ビルド**が必要：
1. `--base /` でルーターをクリーンに（hash URL が `#/1`, `#/2` になる）
2. 資産パス `/assets/` を `./assets/` に後処理（CDN URL は触らない）

```bash
# ビルド
pnpm exec slidev build slides.md --base /

# 資産パスを相対に（CDN の favicon 等は置換しないよう注意）
sed -i '' 's|\(["'\''=]\)/assets/|\1./assets/|g' dist/index.html dist/404.html
```

`package.json` の `build` スクリプトには `--base` が含まれていないため、
GitHub Pages 用には必ず上記を手動実行すること。

## Deployment

GitHub Pages（ブランチ：`gh-pages`）。手順：

```bash
# 1. main ブランチで 2 段階ビルド
git checkout main
pnpm exec slidev build slides.md --base /
sed -i '' 's|\(["'\''=]\)/assets/|\1./assets/|g' dist/index.html dist/404.html

# 2. gh-pages に切り替えて中身を差し替え
git checkout gh-pages
rm -rf * .gitignore
cp /path/to/main/dist/index.html .
cp /path/to/main/dist/404.html .
cp /path/to/main/dist/_redirects .
cp -r /path/to/main/dist/assets .

# 3. コミット＆プッシュ
git add -A
git commit -m "deploy: update static site"
git push origin gh-pages --force

# 4. main に戻る
git checkout main
```

⚠️ **絶対に `cp -r dist /tmp/...` してから `cp -r /tmp/.../* .` してはいけない。**
`dist/` がサブディレクトリとしてコピーされる。必ず `dist/` の**中身のファイル**を 1 つずつルートにコピーすること。

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
routerMode: hash           # GitHub Pages サブディレクトリ用。history だと SPA ルーターが壊れる
```

## Common Gotchas

### GitHub Pages サブディレクトリとルーター（最重要）

GitHub Pages のプロジェクトサイト（`user.github.io/repo-name/`）で SPA の history ルーターは死亡する。
hash ルーターでも `--base /repo-name/` を使うと hash パスに base が注入されて `#/repo-name/2` になり、
ルーターがルート解決に失敗する。

**唯一動く組み合わせ**：
1. headmatter に `routerMode: hash`
2. ビルド時 `--base /`（ルーターをクリーンに）
3. ビルド後 `sed` で資産パス `/assets/` → `./assets/`（CDN URL は触らない）

この 3 点を守らないと以下のいずれかが起きる：
- 直接 URL アクセスで 404
- ナビゲーションで `#/ai-course/2` のような壊れた hash URL
- 資産（CSS/JS）が読み込めず白画面

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

### `base` headmatter フィールドと `--base` CLI フラグ

- headmatter の `base:` は**無効**。CLI の `--base` のみ有効。
- `--base` は Vite の base を設定し、資産パスとルーター base の**両方**に影響する。
  この副作用を避けるため、資産パスはビルド後の `sed` で個別に調整する。

### 開発サーバーのポート

`pnpm run dev` のポートは `3030` がデフォルトだが、占有時は自動で次（3031, 3032...）になる。

### パッケージ追加時

pnpm の lockfile（`pnpm-lock.yaml`）のみをコミットし、`package-lock.json` を混在させないこと。
