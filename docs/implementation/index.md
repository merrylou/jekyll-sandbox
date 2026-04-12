---
layout: default
title: "概要・UC一覧"
parent: "実装工程"
nav_order: 1
---

# 実装工程プレイブック

## 工程概要

詳細設計書に基づきコードを作成する工程。既存改修案件（Java / Spring Boot）が主な対象。

---

## AI活用の原則

1. **開発の主体は人間** — AIは下書き・たたき台の作成役。最終判断と責任は開発者にある
2. **差分確認は必須** — AI生成コードは必ず差分レビューしてからコミットする
3. **セキュリティはAI任せにしない** — 認証・認可・入力検証などはAI出力をそのまま使わず、人間が設計・確認する
4. **既存パターン準拠を指示する** — 「既存の `XxxService` と同じ構成で」のようにプロンプトで明示する

---

## ユースケース一覧

| UC | ユースケース | 主な利用モード | 概要 |
|----|-------------|---------------|------|
| [UC01](uc01-code-scaffold.md) | 設計書からのコード骨格生成 | Agent Mode | 設計書を入力にクラス・メソッドの骨格を生成（プロンプト: `/cd-create-scaffold`） |
| [UC02](uc02-existing-code-mod.md) | 既存コードの改修 | Agent Mode + Inline | 既存コードに対する機能追加・変更（プロンプト: `/cd-change-existing`） |
| [UC03](uc03-refactoring.md) | リファクタリング | Agent Mode | 可読性・保守性の向上を目的としたコード改善（プロンプト: `/cd-change-refactor`） |
| [UC04](uc04-codebase-analysis.md) | 既存コードベースの理解 | Chat + Agent | 既存コードの処理フロー・設計意図の把握（プロンプト: `/cd-analyze-codebase`, `/cd-analyze-spec`） |
| [UC05](uc05-code-review-prep.md) | レビュー前の品質セルフチェック | Chat | レビュー提出前に品質観点でセルフチェック（プロンプト: `/cd-check-review`） |
| [UC06](uc06-changepoint-analysis.md) | 要件起点の修正箇所特定 | Agent Mode | 変更要件から「どのクラスを直すか」を逆引き（プロンプト: `/cd-analyze-changepoint`） |
| [UC07](uc07-errorlog-analysis.md) | エラーログ解析 | Agent Mode | スタックトレースからコードと照合して原因を特定（プロンプト: `/cd-analyze-errorlog`） |
| [UC08](uc08-change-plan.md) | 修正方針の策定 | Agent Mode | 複数アプローチを比較し計画確定後に実装へ移行（プロンプト: `/cd-change-plan`） |

---

## どのUCから始めるか

- **AI活用が初めての方** — まず **UC04（既存コードベースの理解）** から。コードを変更しないため安全に試せる
- **慣れてきたら** — **UC01（コード骨格生成）** や **UC02（既存コードの改修）** で生産性向上を実感する
