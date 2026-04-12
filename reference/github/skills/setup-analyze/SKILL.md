---
layout: default
title: "/setup-analyze"
parent: "GitHub Copilot 設定"
nav_order: 22
name: setup-analyze
description: "Javaプロジェクトの技術スタック・アーキテクチャ・テスト構成・コーディング規約を自動調査し、project-profile.md を生成する。プレイブックのカスタマイズ前準備として使用する。"
---


## 手順

### 1. 規模判定

まずプロジェクトの規模を判定する。

```bash
# ビルドファイルの確認
ls pom.xml build.gradle build.gradle.kts settings.gradle 2>/dev/null

# マルチモジュール構成の確認（Maven）
grep -A10 "<modules>" pom.xml 2>/dev/null | head -15

# ソースファイル数の概算
find . -name "*.java" -not -path "*/test/*" -not -path "*/target/*" | wc -l
```

判定結果を報告する:
- **標準規模**（〜300ファイル、単一モジュール）→ 手順2をそのまま実行
- **大規模**（300ファイル超 or マルチモジュール）→ 以下の手順で進める:
  1. モジュール一覧を `module-overview.md` として出力する（モジュール名・役割・依存関係）
  2. モジュールを役割別に分類する（API / batch / common / infrastructure 等）
  3. 各分類から代表モジュールを1つ選定し、手順2をフルで実施する
  4. 残りのモジュールに対して差異スキャン（手順2-A）を実施する
  5. 差異が検出されたモジュールは手順2のフル調査に昇格する


### 3. 調査の制約

- 不整合がある場合は統一されていると断定せず、混在状況をそのまま報告すること
- マルチモジュール構成の場合、全モジュールのビルドファイルを確認し、差異を明記すること
- 「確認できなかった」項目は空欄ではなく「確認不可（理由）」と記載すること


## よくある落とし穴（SIer現場）

調査中に以下を発見した場合は project-profile.md の「注意点」に明記すること。

1. **DDL自動更新が有効**
   `hbm2ddl.auto=update/create` が開発設定のまま残っていると、本番環境への接続でテーブル構造が自動変更される。

2. **テストDBと本番DBの方言差異**
   H2 と MySQL/PostgreSQL では型・関数・NULL扱いが異なる。H2 統合テストのみでは品質保証が不十分な場合がある。

3. **起動時のデフォルト設定の未確認**
   デフォルトで読み込まれる設定ファイル・DB接続先を必ず確認する。意図しない接続先への接続やデータ操作が発生する場合がある。

4. **DIパターンの混在**
   コンストラクタ注入とフィールド注入が混在していると、テスト時のモック差し替え方法が統一できない。プロンプトに明示する必要がある。

5. **アサーションライブラリの混在**
   AssertJ と Hamcrest が混在していると、生成AIが提案するテストコードが統一されない。プロンプトに明示する必要がある。

6. **`@Transactional` の乱用・未使用**
   Service 層への付け忘れ、または不要なメソッドへの付与が混在している場合は明記する。


## 完了時の案内

調査完了後、以下を案内すること:
1. 生成した project-profile.md のパスを表示する
2. 「内容を確認し、AIが検出できなかったチーム規約があれば手動で追記してください」と案内する
3. 「確認が完了したら `/setup-customize` を実行してカスタマイズを開始してください」と案内する
---


# プロジェクト調査（Java）

このJavaプロジェクトの技術的な特徴を調査し、カスタマイズの入力となる project-profile.md を生成してください。

`$ARGUMENTS` に調査対象のパスや追加指示が含まれる場合はそれに従ってください。
指定がない場合は現在の作業ディレクトリを対象とします。


## 手順

### 1. 規模判定

まずプロジェクトの規模を判定する。

```bash
# ビルドファイルの確認
ls pom.xml build.gradle build.gradle.kts settings.gradle 2>/dev/null

# マルチモジュール構成の確認（Maven）
grep -A10 "<modules>" pom.xml 2>/dev/null | head -15

# ソースファイル数の概算
find . -name "*.java" -not -path "*/test/*" -not -path "*/target/*" | wc -l
```

判定結果を報告する:
- **標準規模**（〜300ファイル、単一モジュール）→ 手順2をそのまま実行
- **大規模**（300ファイル超 or マルチモジュール）→ 以下の手順で進める:
  1. モジュール一覧を `module-overview.md` として出力する（モジュール名・役割・依存関係）
  2. モジュールを役割別に分類する（API / batch / common / infrastructure 等）
  3. 各分類から代表モジュールを1つ選定し、手順2をフルで実施する
  4. 残りのモジュールに対して差異スキャン（手順2-A）を実施する
  5. 差異が検出されたモジュールは手順2のフル調査に昇格する


### 2. 技術プロファイル調査

以下の項目を調査する。各項目について、実際のソースファイル・テストファイル・設定ファイルを確認して回答すること（推測や一般論で埋めない）。

**サンプリング基準**:
- 標準規模: ソース・テスト・設定それぞれ最低3件、異なるパッケージから偏りなく選ぶ
- 大規模（代表モジュール）: モジュール内でソース・テスト・設定それぞれ最低3件、異なるパッケージから選ぶ

#### 2-1. 技術スタック

```bash
# 言語バージョン
grep -E "<java.version>|<source>|<target>|sourceCompatibility|JavaVersion" \
  pom.xml build.gradle 2>/dev/null | head -5

# フレームワーク（親POM）
grep -A5 "<parent>" pom.xml 2>/dev/null | head -10

# バージョン定義一覧
grep -E "<[a-z.-]+\.version>" pom.xml 2>/dev/null | sort -u

# 主要依存ライブラリ（フレームワーク・ORM・DB・ユーティリティ）
grep -E "hibernate|mybatis|liquibase|flyway|jwt|lombok|guice|quarkus|micronaut|jakarta" pom.xml 2>/dev/null
# Gradle の場合
grep -E "implementation|api|compileOnly" build.gradle build.gradle.kts 2>/dev/null | head -30
```

確認事項:
- 言語バージョン
- フレームワーク・バージョン（Spring / Quarkus / Micronaut / Jakarta EE / なし 等）
- DB・ORマッパー（JPA/Hibernate/MyBatis/JDBC直接 等）
- ビルドツール（Maven/Gradle）
- テストフレームワーク・ライブラリ

#### 2-2. アーキテクチャ

```bash
# ディレクトリ構造（深さ4まで、不要ディレクトリ除外）
find . -mindepth 1 -maxdepth 4 -type d \
  | grep -vE "target|\.git|\.idea" | sort

# 主要クラスの存在確認
find . \( -name "*Controller*" -o -name "*Service*" -o -name "*Repository*" \
  -o -name "*UseCase*" -o -name "*Facade*" \) \
  | grep -v "test\|target" | sort | head -40

# DIの有無と方式の確認（@Autowired=Spring, @Inject=CDI/Guice, コンストラクタ注入）
find . -name "*.java" -not -path "*/test/*" -not -path "*/target/*" \
  | xargs grep -l "@Autowired\|@Inject\|@Resource" 2>/dev/null | head -5
```

確認事項:
- レイヤー構成（実際に存在するレイヤーのみ記載。存在しないレイヤーは「なし」と明記）
- パッケージ構成パターン（機能単位 / レイヤー単位 / その他）
- DIの有無とパターン（DIフレームワーク使用 / コンストラクタ注入 / フィールド注入 / DIなし）
- DTOの使用有無と用途（API応答用 / レイヤー間値渡し / 未使用）
- 例外ハンドリングパターン

#### 2-3. テスト構成

```bash
# テストファイル数
find . -path "*/test*" -name "*.java" | wc -l

# テスト種別の内訳（モックを使わない統合テストの有無）
find . -path "*/test*" -name "*.java" \
  | xargs grep -l "@SpringBootTest\|@QuarkusTest\|@MicronautTest\|@ContextConfiguration" 2>/dev/null | wc -l
find . -path "*/test*" -name "*.java" | xargs grep -l "@Mock\|@MockBean\|Mockito" 2>/dev/null | wc -l

# テストフレームワーク・ライブラリ
grep -E "junit|mockito|assertj|hamcrest|testcontainers|rest-assured" pom.xml 2>/dev/null | head -15

# テスト命名規則（サンプル）
find . -path "*/test*" -name "*.java" | head -10
```

確認事項:
- テストフレームワーク（JUnit4 / JUnit5 / TestNG 等）
- Mockライブラリ（Mockito / EasyMock / なし 等）
- アサーションライブラリ（AssertJ / Hamcrest / JUnit標準 / 混在）
- テストアノテーション（@Test, @ExtendWith, @RunWith 等）
- 統合テストの有無・種別（コンテキストロード型 / DBアクセス型 等）
- テストの命名規則（日本語メソッド名 / 英語 等）
- **不整合の有無**: 混在している項目は明記する（例:「AssertJとHamcrestが混在」）

#### 2-4. コーディング規約

```bash
# コメント言語・Javadocの確認（ランダムサンプル）
find . -name "*.java" -not -path "*/test/*" -not -path "*/target/*" | shuf | head -5

# ログ出力パターン（SLF4J/Logback/Log4j の使い方）
find . -name "*.java" -not -path "*/test/*" -not -path "*/target/*" \
  | xargs grep -E "log\.(info|debug|warn|error)|logger\." 2>/dev/null | head -10

# トランザクション管理
find . -name "*.java" -not -path "*/test/*" \
  | xargs grep -l "@Transactional" 2>/dev/null | head -10
```

確認事項:
- 命名規則（クラス・メソッド・変数）
- コメント・Javadocの言語（日本語 / 英語 / 混在）
- ログ出力パターン
- トランザクション管理

#### 2-5. ビルド・実行・環境

```bash
# 設定ファイルの一覧（プロファイル別・環境別含む）
find . \( -name "*.properties" -o -name "*.yml" -o -name "*.yaml" -o -name "*.xml" \) \
  -not -path "*/test/*" -not -path "*/target/*" | sort

# 環境変数の使用状況
find . \( -name "*.properties" -o -name "*.yml" -o -name "*.yaml" \) -not -path "*/test/*" \
  | xargs grep -oE "\$\{[A-Z_][A-Z_0-9]*\}" 2>/dev/null | sort -u | head -20

# CI/CDパイプライン
ls .github/workflows/ 2>/dev/null
ls .circleci/ Jenkinsfile .gitlab-ci.yml 2>/dev/null

# 静的解析ツール
grep -E "checkstyle|spotbugs|pmd|sonar" pom.xml 2>/dev/null | head -5
```

確認事項:
- ビルド / テスト実行 / 起動コマンド
- 設定ファイルの構成（環境別・プロファイル別の有無）
- 必須環境変数
- CI/CDツールとワークフロー概要

#### 2-A. 差異スキャン（大規模・代表以外のモジュール）

代表モジュールの調査結果をベースラインとし、残りのモジュールに対して以下を確認する:

- ビルドファイルの依存ライブラリ差異
- DIアノテーションの方式（コンストラクタ / フィールド）
- テストクラスの命名規則・使用アサーションライブラリ
- 例外ハンドリングパターン

各項目についてソース・テストを2〜3件確認し、代表モジュールと比較する。差異が見つかった場合はそのモジュールを手順2のフル調査に昇格し、差異内容を project-profile.md に明記すること。


### 3. 調査の制約

- 不整合がある場合は統一されていると断定せず、混在状況をそのまま報告すること
- マルチモジュール構成の場合、全モジュールのビルドファイルを確認し、差異を明記すること
- 「確認できなかった」項目は空欄ではなく「確認不可（理由）」と記載すること


## 確認漏れ防止チェックリスト

調査完了前に以下を確認すること。

- [ ] 言語バージョン（Java 8/11/17/21 等）
- [ ] フレームワーク・バージョン
- [ ] DB・ORマッパー・マイグレーションツール
- [ ] DIパターン（混在の有無）
- [ ] テストフレームワーク・ライブラリ（混在の有無）
- [ ] テスト種別（ユニット/統合）と本数
- [ ] テストDBの種別（H2/実DBの違い）
- [ ] 命名規則・コメント言語
- [ ] 環境プロファイル一覧
- [ ] 必須環境変数
- [ ] ビルド・テスト・起動コマンド
- [ ] CI/CDツール


## よくある落とし穴（SIer現場）

調査中に以下を発見した場合は project-profile.md の「注意点」に明記すること。

1. **DDL自動更新が有効**
   `hbm2ddl.auto=update/create` が開発設定のまま残っていると、本番環境への接続でテーブル構造が自動変更される。

2. **テストDBと本番DBの方言差異**
   H2 と MySQL/PostgreSQL では型・関数・NULL扱いが異なる。H2 統合テストのみでは品質保証が不十分な場合がある。

3. **起動時のデフォルト設定の未確認**
   デフォルトで読み込まれる設定ファイル・DB接続先を必ず確認する。意図しない接続先への接続やデータ操作が発生する場合がある。

4. **DIパターンの混在**
   コンストラクタ注入とフィールド注入が混在していると、テスト時のモック差し替え方法が統一できない。プロンプトに明示する必要がある。

5. **アサーションライブラリの混在**
   AssertJ と Hamcrest が混在していると、生成AIが提案するテストコードが統一されない。プロンプトに明示する必要がある。

6. **`@Transactional` の乱用・未使用**
   Service 層への付け忘れ、または不要なメソッドへの付与が混在している場合は明記する。


## 出力

### 出力先

```
docs/analysis/YYYY-MM-DD_NN/project-profile.md
```

- `YYYY-MM-DD` は実行日の日付
- `NN` は同一日内の2桁連番（既存ディレクトリを確認し、最大の連番 +1 を採番。初回は `01`）
- ディレクトリが存在しない場合は作成すること
- 大規模プロジェクトの場合、`module-overview.md`（モジュール一覧・依存関係）も同ディレクトリに出力すること

### 出力テンプレート

```markdown
# PROJECT_PROFILE

## 1. 技術スタック
| 項目 | 詳細 |
|------|------|
| 言語 | |
| フレームワーク | |
| DB・ORマッパー | |
| マイグレーションツール | |
| ビルドツール | |
| テストフレームワーク | |
| 主要ライブラリ | |

## 2. アーキテクチャ
### レイヤー構成
（実際に存在するレイヤーのみ記載。存在しないレイヤーは「なし」と明記）

### パッケージ構成パターン

### DIパターン

### DTOの使用

### 例外ハンドリングパターン

## 3. テスト構成
| 項目 | 詳細 |
|------|------|
| フレームワーク | |
| Mockライブラリ | |
| アサーションライブラリ | |
| テスト種別・本数 | |
| テストDB | |
| 命名規則 | |

### 不整合（該当する場合のみ）
（混在しているライブラリ・アノテーション・命名規則を列挙）

## 4. コーディング規約
| 項目 | 詳細 |
|------|------|
| 命名規則（クラス） | |
| 命名規則（メソッド・変数） | |
| コメント言語 | |
| ログ出力パターン | |
| トランザクション管理 | |

## 5. ビルド・実行
| 操作 | コマンド |
|------|----------|
| ビルド | |
| テスト実行 | |
| 起動 | |

### 設定ファイル構成
| ファイル名 | 用途・環境 |
|------|------|

### 必須環境変数
（設定ファイルで参照されている環境変数を列挙）

## 6. CI/CD・静的解析
| 項目 | 詳細 |
|------|------|
| CI/CDツール | |
| ワークフロー概要 | |
| 静的解析ツール | |

## 7. 注意点
（よくある落とし穴に該当した場合のみ記載）
```


## 完了時の案内

調査完了後、以下を案内すること:
1. 生成した project-profile.md のパスを表示する
2. 「内容を確認し、AIが検出できなかったチーム規約があれば手動で追記してください」と案内する
3. 「確認が完了したら `/setup-customize` を実行してカスタマイズを開始してください」と案内する
