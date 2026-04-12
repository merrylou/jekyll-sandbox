---
layout: default
title: "/ut-change-test"
parent: "GitHub Copilot 設定"
nav_order: 25
name: ut-change-test
description: "レガシーなテストコードをJUnit5+Mockito+AssertJのモダンなスタイルに移行する。JUnit4からの移行・Mock置換・アサーション統一を、テスト意図を維持しつつ実施する。テストのモダナイズ時に使用する。"
---


> コーディング規約は `copilot-instructions.md` に従う。規約と既存コードのパターンが矛盾する場合は既存コードを優先すること。

指定されたテストファイルを読み取り、以下の方針でモダンなテストコードに移行してください。

## 必要な情報（未指定なら質問すること。対話できない場合は妥当な仮定を置き、仮定した内容を出力に明記すること）

- 改善対象のテストファイルパス
- チームのテスト規約ファイル（あれば）
- 命名規則の指定（日本語/英語。未指定の場合は既存のスタイルを維持）

## 移行前の現状確認

以下を確認し、該当しないセクションはスキップすること:

- [ ] JUnit4を使用しているか（`org.junit.Test` が存在するか）→ 該当しなければ「JUnit4 → JUnit5」セクションをスキップ
- [ ] `@MockBean` を使用しているか → 該当しなければ「Mock アノテーション」セクションをスキップ
- [ ] Hamcrest / JUnit4 Assert を使用しているか → 該当しなければ「アサーション」セクションをスキップ
- [ ] PowerMock を使用しているか → 該当しなければ「Mock」セクションのstaticメソッド関連をスキップ

## 移行方針

### JUnit4 → JUnit5
- `@Test`: `org.junit.Test` → `org.junit.jupiter.api.Test`
- `@Before` / `@After` → `@BeforeEach` / `@AfterEach`
- `@BeforeClass` / `@AfterClass` → `@BeforeAll` / `@AfterAll`（`static` メソッドに変更）
- `@RunWith(MockitoJUnitRunner.class)` → `@ExtendWith(MockitoExtension.class)`
- `@Rule ExpectedException` → `assertThrows` に変更
- `@Ignore` → `@Disabled`
- `@Test(expected = ...)` → `assertThrows` に変更

### Mock
- 手動Mockインスタンス生成 / PowerMock → Mockito + `@ExtendWith(MockitoExtension.class)`
- **staticメソッドのMock（PowerMock使用箇所）:** `Mockito.mockStatic(TargetClass.class)` に移行（Mockito 3.4+）。移行が困難な場合はコメントで警告を付与する
- **finalクラス・メソッドのMock:** `mockito-inline` 依存追加が必要な旨をコメントで付与する

### アサーション
- JUnit4 `Assert.*` / Hamcrest `assertThat` → AssertJ `assertThat` に統一
- JUnit5 `Assertions.assertTrue(x)` / `assertFalse(x)` → AssertJ `assertThat(x).isTrue()` / `.isFalse()` に統一
- JUnit5 `Assertions.assertEquals(expected, actual)` → AssertJ `assertThat(actual).isEqualTo(expected)` に統一
- JUnit5 `Assertions.assertNotNull(x)` → AssertJ `assertThat(x).isNotNull()` に統一
- Spring MVC Test の `MockMvcResultMatchers` はそのまま維持する（MockMvc固有のマッチャーAPIのため）

### Spring テスト
- `@RunWith(SpringRunner.class)` → `@ExtendWith(SpringExtension.class)`（または `@SpringBootTest` で暗黙的に適用）
- `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest` 等の Spring アノテーションはそのまま維持する

## 制約

- テストの意図・カバレッジは維持すること（テストケースの追加・削除はしない）
- テスト条件や期待値を変更しない（フレームワーク移行のみ）
- 不要になったimport文は削除する
- PowerMockからの移行で対応困難な箇所は、コード内に `// TODO: PowerMock移行 - [理由]` のコメントを残す

## 出力

移行後に、以下を実施すること：
1. 移行したテストを実行し、全件パスすることを確認する
2. 以下を出力する：
   - 変更箇所の一覧（変更前 → 変更後）
   - 手動対応が必要な箇所（TODOコメント一覧）
