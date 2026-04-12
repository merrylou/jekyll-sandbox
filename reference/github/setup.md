---
layout: default
title: "セットアップ手順"
parent: "GitHub Copilot 設定"
nav_order: 1
---

# AI駆動開発ガイドライン

この `.github/` 配下のファイル群は **マスターテンプレート** です。  
プロジェクトにコピーした後、**使い始める前にカスタマイズが必要です。**

> **重要**: 以下のセットアップを完了するまで、`skills/` 配下のスキルや `agents/` 配下のエージェントを使用しないでください。カスタマイズなしで使用すると、プロジェクトの実態と異なるコード（不要なService層の生成、DTO生成の強制、テストアノテーションの不一致等）が生成されます。

## セットアップ手順（初回のみ・所要15分）

### 1. コピー

```bash
cp -r <playbook-repo>/.github/ <your-project>/.github/
```

### 2. カスタマイズ

[カスタマイズガイド](CUSTOMIZATION.md) に従い、プロジェクトの実態に合わせて設定ファイルを調整してください。

## カスタマイズせずに使うとどうなるか

テンプレートはJava / Spring Bootの標準的な構成（Controller → Service → Repository、DTO使用）を前提としています。プロジェクトの実態と異なる場合、以下の問題が発生します。

- Service層がないプロジェクトでService層の生成が指示される
- DTOを使わない設計なのにDTO生成が求められる
- テストアノテーションや命名規則が既存テストと不一致になる

各スキルには「既存コードのパターンを優先する」ルールが含まれていますが、カスタマイズによりこのギャップを事前に解消しておくことで、AI出力の精度が大幅に向上します。

## ディレクトリ構成

```
.github/
├── README.md                  ← 本ファイル（セットアップ手順）
├── CUSTOMIZATION.md           ← プロジェクトへのカスタマイズガイド
├── copilot-instructions.md    ← プロジェクト共通ルール（AIが毎回参照）
├── agents/
│   ├── implementer.agent.md   ← 実装工程エージェント（タスクメニュー内蔵）
│   └── tester.agent.md        ← テスト工程エージェント（タスクメニュー内蔵）
└── skills/                    ← タスク別スキル（段階的ロード）
    ├── cd-analyze-codebase/   ← リポジトリ概要把握
    │   └── SKILL.md
    ├── cd-analyze-spec/       ← 仕様復元
    │   └── SKILL.md
    ├── ...                    ← 実装系スキル（11種）
    ├── ut-create-test/        ← テスト生成
    │   └── SKILL.md
    └── ...                    ← テスト系スキル（5種）
```

## 最初に試すなら

**セットアップ完了後**、コード変更なしで安全に試せる **リポジトリ概要把握** がおすすめです。

```
@implementer リポジトリの概要を把握して
```

---

## 他のAIツールでの利用

本テンプレートはGitHub Copilot向けに設計されていますが、他のAIコーディングツールでも活用できます。

| ツール | 利用方法 |
|--------|---------|
| Claude Code | `copilot-instructions.md` の内容を `CLAUDE.md` に転記する。スキルファイル（SKILL.md）はそのまま利用可能 |
| Cursor | `copilot-instructions.md` の内容を `.cursorrules` に転記する |
| その他 | `copilot-instructions.md` をプロジェクトルールとして設定し、スキルファイルの内容を直接指示として使用する |

スキルファイル（`.github/skills/*/SKILL.md`）の内容はツール非依存であり、そのまま指示として使用できます。
