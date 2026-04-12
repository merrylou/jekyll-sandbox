---
layout: default
title: "カスタマイズガイド"
parent: "GitHub Copilot 設定"
nav_order: 2
---

# プロジェクトへのカスタマイズガイド

本プレイブックの `.github/` 配下（copilot-instructions.md / README.md / agents / skills）は **マスターテンプレート** です。プロジェクトの技術スタック・アーキテクチャ・チーム規約に合わせてカスタマイズすることで、AIの出力精度が大幅に向上します。

**カスタマイズ自体もGitHub Copilotで実施できます。** 本ガイドでは、2つのスキル（`/setup-analyze` + `/setup-customize`）を使い、Copilotにプロジェクト調査からカスタマイズまでを自動実行させる手順を説明します。

> **実施者**: 本カスタマイズは **開発リーダー** が実施し、カスタマイズ済みの `.github/` をリポジトリにコミットして開発メンバーに展開します。開発メンバーが個別にカスタマイズを行う必要はありません。

> **適用範囲**: 本ガイドのマスターテンプレートは **Java / Spring Boot** を前提に記述されています。Spring Boot 以外の Java フレームワーク（Quarkus / Micronaut / Jakarta EE 等）を使用するプロジェクトでも、スキルが自動的にフレームワークの差異を検出・対応します。詳細は「[付録B: フレームワーク差異への対応](#付録b-フレームワーク差異への対応)」を参照してください。Java 以外の言語の場合は、copilot-instructions.md・エージェント・スキルの技術固有部分を全面的に書き換える必要があります。

---

## 全体の流れ

```
Step 1  .github/ 配下をプロジェクトにコピー（開発リーダー）
  ↓
Step 2  /setup-analyze → 調査結果を確認（開発リーダー）
  ↓
Step 3  /setup-customize → 差分を確認（開発リーダー）
  ↓
Step 4  コミット・展開してチームで試して育てる
```

> スキルが利用できない環境では「[付録A: 手動実行ガイド](#付録a-手動実行ガイド)」のプロンプトを使用してください。

---

## Step 1: `.github/` 配下をコピー

プレイブックリポジトリの `.github/` を対象プロジェクトにコピーします。

```bash
cp -r <playbook-repo>/.github/ <your-project>/.github/
```

既に `.github/copilot-instructions.md` が存在する場合は、既存の内容を残しつつマスターの内容をマージしてください。

> **注**: `README.md`（セットアップ手順）と `CUSTOMIZATION.md`（本ガイド）はテンプレートリポジトリの説明用ドキュメントです。プロジェクトへのコピーは不要です。

---

## Step 2: プロジェクトの実態を調査する

Copilot Chat（Agent Mode）で `/setup-analyze` を実行してください。

スキルが自動的に以下を実行します:
- プロジェクトの規模判定（標準規模 / 大規模・マルチモジュール）
- 規模に応じた調査範囲の自動調整
- 技術スタック・アーキテクチャ・テスト構成・コーディング規約の調査
- `docs/analysis/YYYY-MM-DD_NN/project-profile.md` の生成

### 調査結果の確認

生成された `project-profile.md` を確認し、以下の観点でチェックしてください。

- 各項目が実際のコードと一致しているか（最低1項目は実ファイルを開いて確認）
- 「存在しないレイヤー」が正しく報告されているか
- テスト構成の不整合が漏れなく報告されているか
- AIがコードから読み取れなかった情報（チーム規約、非コード上のルール等）があれば手動で追記する

確認の具体例:
- 「Service層: あり」→ `src/main/java` 配下にServiceクラスが存在するか確認
- 「テストフレームワーク: JUnit5」→ テストファイルで `import org.junit.jupiter` が使われているか確認
- 「アサーションライブラリ: AssertJ」→ テスト内で `assertThat` が使われているか確認

---

## Step 3: カスタマイズを実行する

調査結果の確認後、**新しいチャットセッション** で `/setup-customize` を実行してください。

スキルが自動的に以下を順序通り実行します:
1. `copilot-instructions.md` のカスタマイズ（技術スタック・規約の書き換え）
2. エージェントのカスタマイズ（技術スタック・テスト構成の調整）
3. 全スキルのカスタマイズ（レイヤー構成・テスト構成・ビルドコマンドの調整）
4. カスタマイズレポートの生成

### カスタマイズ結果の確認

ファイル種別ごとに差分を確認し、**段階的にコミット** してください。

```bash
# 1. copilot-instructions.md（全体の基準になるため最初に確認）
git diff .github/copilot-instructions.md

# 2. エージェント
git diff .github/agents/

# 3. スキル（カテゴリ単位で確認）
git diff .github/skills/cd-create-scaffold/ .github/skills/cd-change-existing/ .github/skills/cd-change-plan/ .github/skills/cd-change-refactor/
git diff .github/skills/cd-analyze-*/
git diff .github/skills/cd-check-review/
git diff .github/skills/ut-*/
```

> **段階コミットの推奨**: 上記の順序で確認し、問題がなければファイル種別ごとにコミットしてください。問題が見つかった場合に切り戻しが容易になります。

以下の観点でレビューしてください。

| 確認観点 | 確認内容 |
|----------|----------|
| 技術スタック | `copilot-instructions.md` の技術スタック表がプロジェクトの実態と一致しているか |
| レイヤー構成 | プロジェクトに存在しないレイヤーへの言及が残っていないか |
| テスト構成 | テストフレームワーク・アサーションライブラリ・アノテーションの指定が正しいか |
| 規約の矛盾 | カスタマイズで変更されたルールがチームの既存規約と矛盾していないか |
| 過剰な削除 | セキュリティポリシー・データ取扱等、維持すべきセクションが削除されていないか |
| スキル構造 | スキルの手順・出力形式・YAMLフロントマターが意図せず変更されていないか |

### スモークテスト

`/cd-analyze-codebase` を実行し、出力がプロジェクトの実態と合っていることを確認してください。

---

## Step 4: コミット・展開してチームで試して育てる

### 初回導入の進め方

1. **開発リーダーがカスタマイズ済み `.github/` をコミット** — Step 1〜3 完了後、リポジトリにプッシュ
2. **開発メンバーに展開** — メンバーは `git pull` するだけで、カスタマイズ済みのプロンプト・エージェントが使える状態になる
3. **1週間運用して問題を収集** — 「AIの出力が既存コードと合わなかった」事例をメンバーから収集
4. **チームふりかえりで調整** — 収集した事例をもとに開発リーダーが copilot-instructions.md を更新

### カスタマイズ後のメンテナンス

プロジェクトの構成が変わったときに `/setup-analyze` を再実行して `project-profile.md` を更新し、`/setup-customize` を再実行してください。

主なトリガー:
- フレームワーク・ライブラリのバージョンアップ
- アーキテクチャの変更（レイヤー追加・統合等）
- チームのコーディング規約の変更
- テスト方針の変更
- プレイブックのマスターテンプレートの更新（新スキル追加・構成変更等）

### project-profile.md の位置づけ

`project-profile.md` はカスタマイズの入力情報であり、Copilotが直接参照するファイルではありません。`docs/analysis/` 配下にカスタマイズレポートとともに残しておくことで、再カスタマイズ時の比較やメンバー入れ替え時の引き継ぎに役立ちます。

---

## カスタマイズ後の整合性チェックリスト

カスタマイズ完了後、以下を確認してください:

- [ ] `copilot-instructions.md` の記載がプロジェクトの実態と一致している
- [ ] `implementer.agent.md` / `tester.agent.md` の記述が `copilot-instructions.md` と矛盾していない
- [ ] 各スキルファイル内のレイヤー構成の記述がプロジェクトに合っている
- [ ] テスト関連スキルのフレームワーク・アサーションライブラリの記述がプロジェクトに合っている
- [ ] エージェントのタスク一覧が個別スキルと矛盾していない
- [ ] （Spring Boot 以外の場合）マスターテンプレートの Spring Boot 固有記述が残っていないこと

---
---

## 付録A: 手動実行ガイド

スキルが利用できない環境や、カスタマイズプロセスを細かく制御したい場合は、以下のプロンプトを Copilot Chat（Agent Mode）に貼り付けて実行してください。

> **リポジトリ規模による分岐**:
> - **標準規模**（〜300クラス、単一モジュール）: A-1 のプロンプトをそのまま実行
> - **大規模**（300クラス超 or マルチモジュール）: A-2 の手順に従う

### A-1. プロジェクト調査プロンプト（Step 2 の代替）

```
このプロジェクトの技術的な特徴を調査し、
以下の項目をレポートしてください。

## 調査項目

### 1. 技術スタック
- 言語・バージョン（pom.xml / build.gradle / package.json 等から読み取る）
- フレームワーク・バージョン
- DB・ORマッパー
- ビルドツール
- テストフレームワーク・ライブラリ
- 設定ファイル（application.properties / application.yml）の主要設定

### 2. アーキテクチャ
- レイヤー構成（Controller / Service / Repository 等、実際にどの層が存在するか）
- パッケージ構成パターン（機能単位 / レイヤー単位 / その他）
- DIパターン（コンストラクタインジェクション / フィールドインジェクション）
- DTOの使用有無と用途（API応答用 / フォームバッキング / 未使用）
- 例外ハンドリングパターン（@ControllerAdvice / 個別Controller / 未定義）

### 3. テスト構成
- テストフレームワーク（JUnit4 / JUnit5 / その他）
- Mockライブラリ（Mockito / 手動Mock / その他）
- アサーションライブラリ（AssertJ / Hamcrest / JUnit Assert / 混在）
- テストアノテーション（@MockBean / @MockitoBean / @WebMvcTest 等）
- テストの命名規則
- **不整合の有無**: 混在しているかを明記する

### 4. コーディング規約
- 命名規則（クラス・メソッド・変数）
- コメントの言語（日本語 / 英語 / 混在）
- ログ出力パターン（SLF4J / Log4j / System.out / なし）
- トランザクション管理（@Transactional の付与先）

### 5. ビルド・実行
- ビルドコマンド（mvn / gradle / npm 等）
- テスト実行コマンド
- 起動コマンド

## 調査の制約
- 実際のソースファイル・テストファイル・設定ファイルを確認して回答すること
  - 目安: それぞれ最低3件、異なるパッケージから偏りなく選ぶ
  - 100クラスを超える場合は各カテゴリ5〜10件に増やす
- マルチモジュール構成の場合、全モジュールのビルドファイルを確認し、技術差異を明記すること
- 不整合がある場合は混在状況をそのまま報告すること

## 出力
docs/analysis/YYYY-MM-DD_NN/project-profile.md に保存してください。
- YYYY-MM-DD は実行日の日付、NN は同一日内の2桁連番（初回は 01）

### 出力テンプレート

| 項目 | 詳細 |
|------|------|
| 言語 | |
| フレームワーク | |
| ... | ... |

（各セクションの詳細は 2. アーキテクチャ 〜 5. ビルド・実行 の項目に従う）
```

### A-2. 大規模リポジトリでの実施手順

300クラスを超えるプロジェクトやマルチモジュール構成の場合、以下の3フェーズに分けて**それぞれ新規チャットセッション**で実行してください。

**Phase 1: ビルドファイル・モジュール構成の把握**

```
このプロジェクトのビルドファイル（pom.xml / build.gradle / settings.gradle 等）を
すべて確認し、以下をレポートしてください。

- モジュール一覧（親子関係を含む）
- 各モジュールの技術スタック（言語・フレームワーク・主要依存）
- モジュール間で共通する技術と、モジュール固有の技術
- ビルドコマンド・テスト実行コマンド

出力先: docs/analysis/YYYY-MM-DD_NN/module-overview.md
```

**Phase 2: モジュール単位の詳細調査**（モジュールごとに新規チャットで実行）

```
このプロジェクトの [モジュール名] モジュールの技術的な特徴を調査してください。
対象ディレクトリ: [モジュールのパス]

（A-1 のプロンプトの「調査項目」セクション全体を貼り付ける）

出力先: docs/analysis/YYYY-MM-DD_NN/profile-[モジュール名].md
```

- 技術スタックが同じモジュールはまとめて1回で調査してよい
- 各プロンプトではモジュールのパスを明示し、調査範囲を限定すること

**Phase 3: 統合 project-profile.md の作成**

```
docs/analysis/YYYY-MM-DD_NN/ 配下の module-overview.md と
各 profile-*.md を読み取り、プロジェクト全体の project-profile.md を作成してください。

以下の点に注意:
- モジュール間で統一されている事項と、モジュール固有の事項を区別して記載する
- 不整合は明示する
- A-1 のプロンプトの出力テンプレートに従って記載する

出力先: docs/analysis/YYYY-MM-DD_NN/project-profile.md
```

### A-3. カスタマイズプロンプト（Step 3 の代替）

> **実行ルール**:
> - 以下の各プロンプトは**必ず別のチャットセッション**で実行してください（前のステップの修正結果をファイルシステムから確実に読み取らせるため）
> - スキルの修正はカテゴリ単位で分割実行を推奨します

#### copilot-instructions.md のカスタマイズ（必須）

```
docs/analysis/ 配下の最新の project-profile.md を読み取り、
.github/copilot-instructions.md をこのプロジェクトの実態に合わせて修正してください。

## 修正方針
- 「対象技術スタック」表を、調査結果の技術スタックで書き換える
- 「コーディング規約」内の各ルールについて、調査結果と矛盾するものを修正する
- プロジェクトに存在しないレイヤーや構成要素に関するルールは削除する
- プロジェクト固有のルールがあれば追記する

## 制約
- マスターの構成（セクション構成・セキュリティポリシー・データ取扱）は維持する
- 「既存コードのスタイル・命名規則・パッケージ構成に準拠すること」のルールは必ず残す
- セキュリティポリシーとデータ取扱は基本的に変更しない
```

#### スキルのカスタマイズ（推奨）

> カテゴリ単位で分割実行を推奨:
>
> | カテゴリ | 対象スキル |
> |---------|-----------|
> | コード生成系 | cd-create-scaffold, cd-change-existing, cd-change-plan, cd-change-refactor |
> | 分析系 | cd-analyze-codebase, cd-analyze-spec, cd-analyze-errorlog, cd-analyze-bug, cd-analyze-impact, cd-analyze-changepoint |
> | レビュー系 | cd-check-review |
> | テスト系 | ut-create-test, ut-change-test, ut-analyze-coverage, ut-design-testspec, ut-create-testdata |

```
docs/analysis/ 配下の最新の project-profile.md を読み取り、
.github/skills/ 配下の全スキルファイル（SKILL.md）を確認してください。

## 修正方針

### コード生成系スキル
- コーディング規約セクションがある場合、copilot-instructions.md と一貫性を保つ
- 生成対象リストがプロジェクトのレイヤー構成に合っていなければ修正する
- 手順の末尾にビルド・テスト実行ステップを追加する

### 分析系スキル
- 技術スタック固有の調査手順・観点をプロジェクトの実態に合わせる

### テスト系スキル
- テストフレームワーク・アサーションライブラリ・アノテーションの指定を合わせる

### 全スキル共通
- 各スキルの先頭に優先ルールを追加する:
  「既存コードのパターンと本スキルの規約が矛盾する場合は、既存コードのパターンを優先すること。」

## 制約
- スキルの構造（手順・出力形式・ファイル出力ルール等）は変更しない
- YAMLフロントマター（name / description）は変更しない
- 変更したファイル名と変更概要を一覧で出力する
```

#### エージェントのカスタマイズ（推奨）

```
docs/analysis/ 配下の最新の project-profile.md を読み取り、
.github/agents/ 配下のエージェントファイルを修正してください。

## 修正方針
- テスト構成の指定を project-profile.md の実態に合わせる
- テスト種別の選択表がプロジェクトのテスト構成と矛盾していれば修正する
- 行動原則は基本的に変更しない

## 制約
- YAMLフロントマター（name / description）は変更しない
- エージェントの役割・責務は変更しない
```

#### レビュー観点の追加（任意）

```
docs/analysis/ 配下の最新の project-profile.md と .github/copilot-instructions.md を読み取り、
.github/skills/cd-check-review/SKILL.md のチェック観点に
このプロジェクト固有の観点を追加してください。

既存の観点は変更しないでください。追加のみ行ってください。
```

---

## 付録B: フレームワーク差異への対応

`project-profile.md` のフレームワークが Spring Boot 以外の場合、スキル（`/setup-customize`）は自動的にフレームワーク差異を検出・対応します。手動実行の場合は、以下の対応表を参考にしてください。

| マスターの Spring Boot 記述 | 置き換え例（Quarkus） | 置き換え例（Jakarta EE） |
|---|---|---|
| `@Autowired` / コンストラクタインジェクション | `@Inject`（CDI） | `@Inject`（CDI） |
| `@Transactional`（Spring） | `@Transactional`（Jakarta） | `@Transactional`（Jakarta） |
| `@WebMvcTest` | `@QuarkusTest` + REST Assured | Arquillian |
| `@DataJpaTest` | `@QuarkusTest` + `@TestTransaction` | Arquillian + JPA |
| `@MockBean` | `@InjectMock`（QuarkusMock） | CDI Alternatives / Mockito |
| `@SpringBootTest` | `@QuarkusTest` | Arquillian |
| Controller / Service / Repository | Resource / Service / Repository | Resource(JAX-RS) / EJB / DAO |
| Spring Data JPA | Panache / Hibernate | JPA（EntityManager） |

**対応の原則**:
- マスターテンプレートのフレームワーク固有記述は、対象フレームワークの同等概念にすべて置き換える
- 対象フレームワークに該当する概念がない記述は削除する
- エージェントファイル冒頭の「Java / Spring Boot」も実際のスタックに置き換える
- `copilot-instructions.md` の「Spring Boot 固有」セクションを対象フレームワーク用に書き換える（「共通」セクション・セキュリティポリシー・データ取扱は維持する）

手動実行の場合、各プロンプトの末尾に以下を追加してください:

```
project-profile.md のフレームワークが Spring Boot でない場合:
- マスターテンプレートの Spring Boot 固有の記述は、
  対象フレームワークの同等概念にすべて置き換えること
- 該当する概念がない記述は削除すること
```
