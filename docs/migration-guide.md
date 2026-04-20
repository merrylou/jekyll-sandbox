# Jekyll デザインシステム移行手順書

**移行元**: `merrylou/jekyll-sandbox`
**移行先**: `aidd-guide/CreFutureHub`
**作成日**: 2026-04-14

---

## 前提

- 移行元（`merrylou/jekyll-sandbox`）は **個人アカウント** のリポジトリ
- 移行先（`aidd-guide/CreFutureHub`）は **会社組織** のリポジトリ — アカウントが異なる
- 移行先にはコンテンツ（`.md` ファイル）が既に存在する
- 移行するのは **Jekyll インフラ（レイアウト・CSS・ナビゲーション・ダッシュボード・ワークフロー）** のみ
- 移行先で GitHub Pages を有効にする（将来 Crepe 静的サイトホスティングへの移行可能性あり）
- 移行方式: **移行元をクローン → 不要ファイル除去 → リモートを移行先に差替え → プッシュ**

---

## 1. 移行対象ファイル一覧

| パス | 役割 | 備考 |
|------|------|------|
| `_config.yml` | Jekyll 設定 | **baseurl / url を変更する（後述）** |
| `_layouts/default.html` | 全ページ共通レイアウト | ヘッダー・サイドバー・フッター |
| `_includes/nav-items.html` | サイドバーナビゲーション | Liquid でページ階層を自動生成 |
| `_includes/sidebar.html` | サイドバーラッパー | nav-items.html を include |
| `_includes/mobile-header.html` | モバイルメニュー | 旧デザインの残骸。現行では未使用だが念のため移行 |
| `assets/css/main.css` | グローバルCSS | デザイントークン + 全コンポーネント |
| `_data/skills.yml` | スキルデータ | ダッシュボードの表・バーチャートを Liquid で動的生成 |
| `_data/reviews.yml` | レビュー履歴データ | ダッシュボードのタイムライン・コンテンツマップを Liquid で動的生成 |
| `dashboard.html` | ダッシュボードページ | Vercel/Linear 風 UI、インラインCSS含む |
| `index.md` | トップページ | 移行先に既存の場合は内容を確認して判断 |
| `.github/workflows/jekyll.yml` | GitHub Actions | ビルド＆デプロイパイプライン |

### 除外ファイル

| パス | 理由 |
|------|------|
| `mock-coverage.html` | デザイン検証用モック（本番不要） |
| `docs/**`, `reference/**` | コンテンツ MD（移行先に既存） |
| `.claude/` | Claude Code 設定（個人環境依存） |

---

## 2. 移行手順

### Step 1: 移行元リポジトリをクローン

```bash
git clone https://github.com/merrylou/jekyll-sandbox.git
cd jekyll-sandbox
```

### Step 2: 不要ファイルを除去

検証用ファイルとコンテンツ MD を削除する（移行先に既存のため）。

```bash
# 検証用モック
rm -f mock-coverage.html

# コンテンツ MD（移行先に既存）
rm -rf docs/ reference/

# インデックス MD（移行先で既存の場合のみ削除。なければ残す）
# rm -f index.md docs-index.md reference-index.md

# Claude Code 設定（個人環境依存）
rm -rf .claude/
```

> **判断ポイント**: `index.md` と各 `-index.md` は移行先に同等のファイルがあるか確認してから削除すること。
> なければ残してそのまま移行する。

### Step 3: リモートを移行先に差し替え

```bash
# 現在のリモートを確認
git remote -v

# 移行元を削除し、移行先を設定
git remote remove origin
git remote add origin https://github.com/aidd-guide/CreFutureHub.git

# 確認
git remote -v
# origin  https://github.com/aidd-guide/CreFutureHub.git (fetch)
# origin  https://github.com/aidd-guide/CreFutureHub.git (push)
```

> **認証**: 組織リポジトリへの push には組織アカウントの認証情報が必要。
> PAT（Personal Access Token）または SSH キーが組織に紐づいていることを確認する。

### Step 4: 移行先の既存コンテンツを取得・統合

```bash
# 移行先の main ブランチを取得
git fetch origin

# 作業ブランチを作成（移行先の main をベースに）
git checkout -b feature/jekyll-design-system origin/main

# 移行元のファイルを作業ブランチにコピー
# （Step 2 で不要ファイルを除去済みの状態から持ってくる）
git checkout main -- \
  _config.yml \
  _layouts/default.html \
  _includes/nav-items.html \
  _includes/sidebar.html \
  _includes/mobile-header.html \
  assets/css/main.css \
  _data/skills.yml \
  _data/reviews.yml \
  dashboard.html \
  .github/workflows/jekyll.yml
```

> `index.md` や `-index.md` も必要な場合は上記に追加する。

### Step 5: `_config.yml` を編集

`baseurl` と `url` を移行先に合わせて変更する。

```yaml
# 変更前（merrylou/jekyll-sandbox）
url: "https://merrylou.github.io"
baseurl: "/jekyll-sandbox"

# 変更後（aidd-guide/CreFutureHub）
url: "https://aidd-guide.github.io"
baseurl: "/CreFutureHub"
```

それ以外の設定（title, description, defaults, plugins）は変更不要。

> **重要**: `defaults` セクションの `path` 値（`docs/common`, `docs/implementation` 等）は、
> 移行先リポジトリの MD ファイルの配置パスと一致させる必要がある。
> パスが異なる場合は `defaults` の `path` を合わせること。

### Step 6: コンテンツ MD の front matter を確認

移行先の各 MD ファイルに以下の front matter が必要。なければ追加する。

```yaml
---
layout: default
title: "ページタイトル"
parent: "親カテゴリ名"    # ナビゲーション階層に使用
nav_order: 1              # 表示順
---
```

**`parent` の値は `nav-items.html` のカテゴリ名と完全一致が必要。** 対応表:

| parent 値 | 対象ディレクトリ |
|-----------|-----------------|
| `共通ガイド` | `docs/common/` |
| `実装工程` | `docs/implementation/` |
| `単体テスト工程` | `docs/unit-testing/` |
| `リサーチ` | `docs/research/` |
| `レビュー記録` | `docs/reviews/` |
| `GitHub Copilot 設定` | `reference/github/` |
| `Claude Code コマンド` | `reference/claude/` |

### Step 7: GitHub Pages を有効化

1. GitHub リポジトリの **Settings > Pages** を開く
2. **Source**: `GitHub Actions` を選択
3. カスタムドメインが必要な場合はここで設定

### Step 8: コミット＆プッシュ

```bash
git add -A
git commit -m "feat: Jekyll デザインシステム統合（Storybook Docs風 + Vercel/Linear ダッシュボード）"
git push -u origin feature/jekyll-design-system
```

PR を作成してマージすると、GitHub Actions が自動でビルド＆デプロイする。

---

## 3. デプロイ後の検証チェックリスト

- [ ] トップページ（`/CreFutureHub/`）が表示される
- [ ] ヘッダーロゴが「A」アイコンで表示される
- [ ] サイドバーのナビゲーションに全カテゴリが展開される
- [ ] 各カテゴリ配下のページリンクが正しく遷移する
- [ ] ダッシュボード（`/CreFutureHub/dashboard.html`）が表示される
  - [ ] KPI カード 5枚が横並びで表示
  - [ ] ドーナツチャート + バーチャートが 2カラムで表示
  - [ ] コンテンツマップの全カテゴリカードが表示
  - [ ] スキル詳細テーブルに18件表示（`skills.yml` から動的生成）
  - [ ] レビュー履歴タイムラインが表示
- [ ] モバイル表示（768px以下）でハンバーガーメニューが動作する
- [ ] 404 が出るリンクがないこと

---

## 4. 技術上の注意事項

### Chart.js / JavaScript は使えない

Jekyll のビルドプロセスが `<canvas>` や `<script>` を除去するため、チャートは **純CSS**（`conic-gradient`, `width%`）で実装している。JavaScript に置き換えないこと。

### ダッシュボードは全面 Liquid 自動生成

ダッシュボードの全セクションが `site.pages` と `_data/*.yml` から Liquid で動的に計算される。**コンテンツを追加・削除しても `dashboard.html` の手動更新は不要**。

| セクション | データソース | 自動化の仕組み |
|-----------|-------------|---------------|
| KPI カード | `site.pages`, `site.data.skills`, `site.data.reviews` | ページ数・スキル数・エージェント数・レビュー回をフィルタ集計 |
| ドーナツチャート | `site.pages` の `parent` 値 | カテゴリ別ページ数から角度を算出（`count × 360 ÷ total`）し `conic-gradient` を生成 |
| バーチャート | `_data/skills.yml` | `category` × `subcategory` でグルーピングし、最大値を基準に width% を算出 |
| コンテンツマップ | `site.pages` の `parent` 値 | カテゴリごとにページを Liquid ループでピル表示 |
| スキル詳細テーブル | `_data/skills.yml` | 全レコードをループ展開 |
| レビュー履歴タイムライン | `_data/reviews.yml` | 全レコードをループ展開（スコア・説明含む） |

#### コンテンツ追加時の運用

| 追加内容 | 必要な作業 |
|---------|-----------|
| ユースケース MD の追加 | 正しい `parent` を front matter に設定するだけ |
| 新規スキル | `_data/skills.yml` にレコードを追加 |
| 新規レビュー回 | `_data/reviews.yml` にレコードを追加（date / subtitle / scores / description） |
| 新規カテゴリ追加 | `dashboard.html` の Liquid 変数定義に行を追加（KPI・ドーナツ・マップカードそれぞれ1行） |

#### ダッシュボード内の Liquid ロジック構成

`dashboard.html` 冒頭の Liquid ブロックで全ての変数が定義されている:

```liquid
{% raw %}{%- comment -%} Category page counts {%- endcomment -%}
{% assign cat_common = site.pages | where: "parent", "共通ガイド" | size %}
{% assign cat_impl   = site.pages | where: "parent", "実装工程"   | size %}
...
{% assign total_pages = cat_common | plus: cat_impl | plus: ... %}

{%- comment -%} Donut cumulative angles {%- endcomment -%}
{% assign d_common = cat_common | times: 360.0 | divided_by: total_pages %}
{% assign a1 = d_common %}
{% assign a2 = a1 | plus: d_impl %}
...{% endraw %}
```

新カテゴリ追加時は、この冒頭ブロック → 対応する HTML セクションの順に修正すればよい。

### CSS クラス命名規則

| プレフィックス | スコープ |
|---------------|---------|
| `sb-` | サイト全体（ヘッダー、サイドバー、メニュー、フッター） |
| `ds-` | ダッシュボード固有 |

### デザイントークンの変更

ブランドカラーやフォントを変更する場合は `assets/css/main.css` の `:root` セクションのみ編集する。

```css
/* 主要トークン */
--blue-500: #029CFD;     /* アクセントカラー */
--sb-pink: #FF4785;      /* ロゴ背景色 */
--font-sans: 'Nunito Sans', ...;  /* 本文フォント */
```

### Crepe 移行時の考慮事項

Crepe（静的サイトホスティング）に移行する場合:

- Jekyll ビルド済みの `_site/` ディレクトリをそのままデプロイ可能
- `baseurl` を Crepe のパス構成に合わせて変更する
- GitHub Actions ワークフローの deploy ステップを Crepe 用に差し替える

---

## 5. ファイル構成図（移行後の想定）

```
CreFutureHub/
├── .github/workflows/
│   └── jekyll.yml              # ビルド＆デプロイ
├── _config.yml                 # Jekyll 設定
├── _data/
│   └── skills.yml              # スキルデータ
├── _includes/
│   ├── nav-items.html          # ナビゲーション
│   ├── sidebar.html            # サイドバーラッパー
│   └── mobile-header.html      # モバイルメニュー
├── _layouts/
│   └── default.html            # 共通レイアウト
├── assets/css/
│   └── main.css                # グローバルCSS
├── dashboard.html              # ダッシュボード
├── index.md                    # トップページ
├── docs/                       # コンテンツ（既存）
│   ├── common/
│   ├── implementation/
│   ├── unit-testing/
│   ├── research/
│   └── reviews/
└── reference/                  # 参考資料（既存）
    ├── github/
    └── claude/
```

---

## 6. トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| ページが 404 | `baseurl` の不一致 | `_config.yml` の `baseurl` を確認 |
| サイドバーにページが出ない | front matter の `parent` 不一致 | MD の `parent` 値と `nav-items.html` のカテゴリ名を照合 |
| CSS が当たらない | CSS パスの不一致 | `_layouts/default.html` の `<link>` が `relative_url` を使っているか確認 |
| ダッシュボードの表が空 | `skills.yml` がない or パス違い | `_data/skills.yml` の存在を確認 |
| ビルドが失敗 | Jekyll プラグイン不足 | `_config.yml` の `plugins` に `jekyll-seo-tag` があるか確認（GitHub Pages 標準サポート） |
| フォントが表示されない | Google Fonts の読み込みブロック | 社内プロキシが Fonts API をブロックしていないか確認 |
