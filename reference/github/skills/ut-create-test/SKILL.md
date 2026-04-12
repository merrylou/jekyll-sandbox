---
layout: default
title: "/ut-create-test"
parent: "GitHub Copilot 設定"
nav_order: 26
name: ut-create-test
description: "実装済みクラスに対しJUnit5+Mockito+AssertJの単体テストを生成する。クラス種別に応じたアノテーション選択・AAAパターン・正常系/異常系/境界値の網羅を行う。テストコード作成時に使用する。"
---


> コーディング規約は `copilot-instructions.md` に従う。規約と既存コードのパターンが矛盾する場合は既存コードを優先すること。

指定されたファイルを読み取り、単体テストを生成してください。

## 必要な情報（未指定なら質問すること。対話できない場合は妥当な仮定を置き、仮定した内容を出力に明記すること）

- テスト対象のファイルパス
- スコープ（クラス全体 or 特定メソッド）
- 参照すべき既存テストファイル（あれば）
- テストメソッドの命名言語（日本語 / 英語。デフォルト: 日本語）

## テスト種別の選択

テスト対象のクラス種別に応じて以下のアノテーション・構成を使い分けること。

| 対象クラス | 推奨アノテーション | 用途 |
|---|---|---|
| Service / Component | `@ExtendWith(MockitoExtension.class)` | 純粋な単体テスト。依存はすべてMockにする |
| Controller | `@WebMvcTest(XxxController.class)` + `MockMvc` | Webレイヤのみロード。Serviceは`@MockBean`でMockする |
| Repository | `@DataJpaTest` | JPAレイヤのみロード。デフォルトはH2インメモリDB |
| Repository（実DB検証） | `@DataJpaTest` + Testcontainers | 本番DBと同じエンジン（PostgreSQL等）で検証が必要な場合 |
| Validator / Formatter / Utility 等 | アノテーションなし（plain JUnit5） | Springコンテキスト不要のクラス。`new` でインスタンス生成してテストする |
| 統合テスト | `@SpringBootTest` | 全コンテキストをロードして検証。範囲が広いため必要な場合のみ使用 |

- `@WebMvcTest` と `@DataJpaTest` は必要なレイヤのみロードするため、`@SpringBootTest` よりも起動が速い
- Testcontainersを使用する場合は `testcontainers` および対象DBドライバの依存関係が `pom.xml` / `build.gradle` に含まれているか確認すること

## テスト構成

- フレームワーク: JUnit5 + Mockito (`@ExtendWith(MockitoExtension.class)`)
- アサーション: AssertJ (`assertThat`)。関連する複数アサーションは `assertAll` でグループ化する
- Mock設定: `@Mock` + `@InjectMocks` を基本とする。`new` で生成できる依存はMockしない
- 各テストは独立して実行可能にする（`@BeforeEach` でセットアップを共通化）

## テスト構造

- **AAA パターン**（Arrange-Act-Assert）に従ってテストメソッドを記述する
  - `// Arrange`: テスト対象の準備・Mockの設定
  - `// Act`: テスト対象メソッドの呼び出し
  - `// Assert`: 結果の検証
- **テストメソッド名**: 命名言語の指定に従う
  - 英語の場合: `methodName_should_expectedBehavior_when_scenario` 形式
  - 日本語の場合: `@DisplayName("〇〇の場合、〇〇になること")` を付与してメソッド名は英語スネークケースにする

## テスト方針

- 正常系・異常系・境界値を網羅する
- 境界値・同値クラスが複数あるケースは `@ParameterizedTest` を活用する
  - 単純な値の列挙: `@CsvSource` または `@ValueSource`
  - 複雑なオブジェクトや期待値の組み合わせ: `@MethodSource`
- 例外スローはテスト対象メソッドの直接呼び出しで `assertThrows` を使う
- 既存テストファイルがある場合はスタイル・命名規則を参照する

## 手順

1. テスト対象クラスを読み取り、全メソッドの分岐・条件・例外パスを把握する
2. 既存テストファイルがあればスタイルを参照する
3. テストクラスを生成する（出力先: `src/test/java/` 配下の対応パッケージ）
4. 生成したテストを実行し、全件パスすることを確認する
5. 生成したテストケース一覧（メソッド名・観点・正常/異常の区分）を表形式で出力する
