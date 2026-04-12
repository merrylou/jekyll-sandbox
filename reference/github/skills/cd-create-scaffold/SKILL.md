---
layout: default
title: "/cd-create-scaffold"
parent: "GitHub Copilot 設定"
nav_order: 21
name: cd-create-scaffold
description: "設計書からController/Service/Repository/DTOの骨格コードを一括生成する。既存パターンに合わせたSpring Bootのコーディング規約に従い、ビジネスロジック部分はTODOコメントで明示する。新規機能の骨格作成時に使用する。"
---


> コーディング規約は `copilot-instructions.md` に従う。規約と既存コードのパターンが矛盾する場合は既存コードを優先すること。

プロジェクトの既存パターンを確認し、以下の設計書に基づいてコード骨格を生成してください。

## 生成対象

- Controller（リクエスト/レスポンスのマッピング）
- Service（インターフェースと実装クラス）
- Repository（Spring Data JPAのインターフェース）
- DTO（リクエスト/レスポンス/エンティティ）

## Spring Boot コーディング規約

生成するコードは以下の規約に従うこと。

### パッケージ構成
- **機能/ドメイン単位**でパッケージを切る（例: `com.example.app.order`）
- レイヤー単位（`controller` / `service` / `repository` を横断するパッケージ）は避ける

### 依存性注入
- **コンストラクタインジェクション**を使用する（`@Autowired` フィールドインジェクション禁止）
- 依存フィールドは `private final` で宣言する

### Web層 (Controller)
- **DTOを使用**してリクエスト/レスポンスをマッピングする。JPAエンティティをAPIに直接公開しない
- リクエストDTOには `@Valid` を付与し、フィールドに `@NotNull` / `@Size` 等のBean Validationアノテーションを付与する
- 例外ハンドリングは `@ControllerAdvice` + `@ExceptionHandler` で一元管理する（Controllerに個別実装しない）

### サービス層 (Service)
- `@Transactional` はServiceメソッド単位で付与する（Controller・Repositoryには付与しない）
- Serviceクラスはインターフェース + 実装クラスの構成とする

### データ層 (Repository)
- Spring Data JPA の `JpaRepository` または `CrudRepository` を継承する
- 複雑なクエリは `@Query` またはJPA Criteria APIを使用する
- APIレスポンス用にカラムを絞る場合はDTOプロジェクションを使用する

### 設定・ログ
- シークレット・認証情報はコードに直書きしない（`application.yml` の環境変数参照 `${ENV_VAR}` を使用）
- ロガーは SLF4J を使用し、メッセージは `logger.info("Processing {}...", id)` のパラメータ化形式で記述する

## 指示

1. 指定する既存の類似機能パッケージを読み取り、スタイル・構成を把握する
2. 上記「Spring Boot コーディング規約」に従いコード骨格を生成する
3. ビジネスロジック部分は `// TODO: [設計書参照] xxxを実装` コメントで明示し、中身は実装しない
4. バリデーションアノテーションは設計書の入力チェック仕様に基づく
5. プロジェクトに i18n（国際化）が導入されている場合は、テンプレートやビューで使用する文字列をメッセージプロパティ（`messages.properties` 等）に定義し、テンプレートではメッセージキー参照（例: `#{key}`）を使用すること。全ロケールファイルにキーを追加すること
6. 生成後、`./mvnw compile`（または対応するビルドコマンド）でコンパイルエラーがないことを確認する

## 必要な情報（未指定なら質問すること。対話できない場合は妥当な仮定を置き、仮定した内容を出力に明記すること）

- 参考にする既存機能のパッケージパス
- 設計書のファイルパスまたは内容
- 対象機能名
- 生成先のベースパッケージ
- プロジェクト固有の例外基底クラス名
