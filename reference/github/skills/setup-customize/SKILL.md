---
layout: default
title: "/setup-customize"
parent: "GitHub Copilot 設定"
nav_order: 23
name: setup-customize
description: "project-profile.md の調査結果に基づき、copilot-instructions.md・エージェント・スキルをプロジェクトの実態に合わせて一括カスタマイズする。/setup-analyze 実行後に使用する。"
---


# プロジェクトカスタマイズ

docs/analysis/ 配下の最新の project-profile.md を読み取り、.github/ 配下の Copilot 設定をこのプロジェクトの実態に合わせてカスタマイズしてください。

## 前提確認

1. docs/analysis/ 配下に project-profile.md が存在することを確認する
2. 存在しない場合は「先に `/setup-analyze` を実行してください」と案内して終了する

## カスタマイズ手順

以下の順序で修正する。前のステップの修正結果を考慮しながら進めること。

### Step 1: copilot-instructions.md

.github/copilot-instructions.md を修正する。

**修正方針**:
- 「対象技術スタック」表を project-profile.md の技術スタックで書き換える
- 「コーディング規約」内のルールで、プロジェクトの実態と矛盾するものを修正する
- プロジェクトに存在しないレイヤーや構成要素のルールは削除する
- プロジェクト固有のルールがあれば追記する
- フレームワークが Spring Boot でない場合、「Spring Boot 固有」セクションを対象フレームワーク用に書き換える

**制約**:
- セクション構成（共通 / フレームワーク固有 / セキュリティポリシー / データ取扱）は維持する
- 「既存コードのスタイル・命名規則・パッケージ構成に準拠すること」のルールは必ず残す
- セキュリティポリシーとデータ取扱は基本的に変更しない

### Step 2: エージェント

.github/agents/ 配下のエージェントファイルを修正する。

**修正方針**:
- 冒頭の技術スタック記述（「Java / Spring Boot」等）をプロジェクトの実態に合わせる
- テスト構成（フレームワーク・アサーション・アノテーション）を project-profile.md に合わせる
- テスト種別の選択表がプロジェクトのテスト構成と矛盾していれば修正する

**制約**:
- YAMLフロントマター（name / description）は変更しない
- エージェントの役割・責務・行動原則は変更しない

### Step 3: スキル

.github/skills/ 配下の全スキルファイル（SKILL.md）を修正する。ただし `setup-analyze` と `setup-customize` は修正対象外とする。

**コード生成系**（cd-create-scaffold, cd-change-existing, cd-change-plan, cd-change-refactor）:
- コーディング規約セクションを Step 1 の修正結果と一貫させる
- 生成対象リストがプロジェクトのレイヤー構成と合わなければ修正する
- 手順の末尾にビルド・テスト実行ステップを追加する（project-profile.md のコマンドを使用）

**分析系**（cd-analyze-codebase, cd-analyze-spec, cd-analyze-errorlog, cd-analyze-bug, cd-analyze-impact, cd-analyze-changepoint）:
- 技術スタック固有の調査手順・観点をプロジェクトの実態に合わせる

**テスト系**（ut-create-test, ut-change-test, ut-analyze-coverage, ut-design-testspec, ut-create-testdata）:
- テストフレームワーク・アサーションライブラリ・アノテーションを project-profile.md に合わせる

**レビュー系**（cd-check-review）:
- プロジェクト固有の品質観点があれば、チェック観点に追加する（既存観点は変更しない）

**全スキル共通**:
- 各スキルの先頭に「既存コードのパターンと本スキルの規約が矛盾する場合は、既存コードのパターンを優先すること。」が未設定なら追加する
- フレームワークが Spring Boot でない場合、Spring Boot 固有の記述を対象フレームワークの同等概念に置き換える。該当する概念がない記述は削除する

**制約**:
- スキルの構造（手順・出力形式・ファイル出力ルール等）は変更しない
- YAMLフロントマター（name / description）は変更しない

## レポート出力

カスタマイズ完了後、以下のレポートを出力する。

**出力先**: project-profile.md と同じディレクトリの `customization-report.md`

```markdown
# カスタマイズレポート

## 実施日
YYYY-MM-DD

## 変更ファイルと変更概要
| ファイル | 変更概要 |
|---------|---------|
| .github/copilot-instructions.md | （変更内容） |
| ... | ... |

## フレームワーク置き換え（該当する場合のみ）
（Spring Boot からの置き換え内容）

## 未変更ファイル
（変更不要だったファイルがあれば記載）
```

## 完了時の案内

1. 変更ファイル数と主な変更内容のサマリを報告する
2. 以下の確認手順を案内する:
   - `git diff .github/copilot-instructions.md` で基盤ルールの差分を確認
   - `git diff .github/agents/` でエージェントの差分を確認
   - `git diff .github/skills/` でスキルの差分を確認
3. 整合性チェックリストを提示する:
   - [ ] copilot-instructions.md の記載がプロジェクトの実態と一致しているか
   - [ ] エージェントの記述が copilot-instructions.md と矛盾していないか
   - [ ] 各スキルのレイヤー構成・テスト構成がプロジェクトに合っているか
   - [ ] （Spring Boot 以外の場合）Spring Boot 固有記述が残っていないか
