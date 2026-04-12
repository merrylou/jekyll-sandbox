---
layout: default
title: "ADR: プロンプト構成設計"
parent: "リサーチ"
nav_order: 1
---

# ADR: プロンプトのディレクトリ構成設計

| 項目 | 内容 |
|------|------|
| ステータス | 採択（実装前・検証待ち） |
| 決定日 | 2026-04-10 |
| 関連Issue | [microsoft/vscode#268780](https://github.com/microsoft/vscode/issues/268780) — `.github/prompts/` のサブディレクトリ非対応 |

---

## 1. 背景と課題

本リポジトリでは、実装工程（`cd-*`）とテスト工程（`ut-*`）のプロンプトを `.github/prompts/` にフラット配置している（現在18ファイル）。

今後、以下の工程が追加されるとプロンプト数が大幅に増加し、フラット構造では管理が困難になる。

- 詳細設計工程
- 結合テスト工程
- 性能テスト・セキュリティテスト
- デプロイ・運用

**解決したい問題**: プロンプトを階層構造で整理しつつ、GitHub Copilotから確実に利用できる構成を確立する。

---

## 2. 制約条件（調査で判明した事実）

### `.github/prompts/` のサブディレクトリ非対応

- `.github/prompts/` **直下**の `.prompt.md` のみがスラッシュコマンド（`/name`）で認識される
- サブディレクトリ内の `.prompt.md` はスラッシュコマンドに表示されない
- VS Code Issue [#268780](https://github.com/microsoft/vscode/issues/268780) として報告済み（2025年9月）、2026年4月時点で未修正

### Markdownリンクの自動ロード

- `.agent.md` / `.prompt.md` 内のMarkdownリンク `[text](path)` で参照されたファイルは自動的にコンテキストにロードされる
- サブディレクトリ内のファイルもMarkdownリンクで参照可能
- ただし、エージェントファイル内の全リンクが起動時に一括ロードされる（選択的ロードではない）
- `chat.includeReferencedInstructions` 設定に依存する場合がある

### `.github/instructions/` はサブディレクトリ対応

- `.instructions.md` + `applyTo`（glob構文）で、対象ファイルパターンに応じた自動適用が可能
- サブディレクトリもサポートされている

---

## 3. 検討した選択肢

### 方式A: フラット + プレフィックス命名（現状維持）

```
.github/prompts/
├── cd-analyze-codebase.prompt.md
├── cd-analyze-spec.prompt.md
├── ut-create-test.prompt.md
└── ...（全ファイルがフラット）
```

| 評価軸 | 結果 |
|--------|------|
| スラッシュコマンド | 全プロンプトで利用可能 |
| コンテキスト効率 | 呼び出したプロンプトのみロード（最良） |
| ディレクトリの見通し | プレフィックスで分類。30個超で一覧性が低下 |
| 運用コスト | なし |

### 方式B: サブディレクトリ + Markdownリンク全ロード

```
.github/prompts/
├── cd-menu.prompt.md              ← フラット（スラッシュコマンド可）
├── implementation/
│   ├── analyze-codebase.prompt.md ← サブディレクトリ
│   └── ...
└── testing/
    └── ...

.github/agents/
└── implementer.agent.md           ← Markdownリンクでサブディレクトリ内を参照
```

エージェントファイルやメニュープロンプトからMarkdownリンクで参照すれば、サブディレクトリ内のファイルもコンテキストに含まれる。

| 評価軸 | 結果 |
|--------|------|
| スラッシュコマンド | メニューのみ。個別プロンプトは不可 |
| コンテキスト効率 | リンクされた全ファイルが一括ロード（現在約63KB ≒ 15,000〜20,000トークン） |
| ディレクトリの見通し | 階層で整理される |
| 運用コスト | 低い |

**不採択の理由**: Markdownリンクは「全ロードか何もロードしないか」の二択であり、タスクに応じた選択的ロードができない。現在の63KBは許容範囲だが、工程追加で倍増以上になった場合のスケーラビリティに懸念がある。

### 方式C: サブディレクトリ + パス文字列指示（エージェントのreadFile）

方式Bと同じディレクトリ構造だが、Markdownリンクの代わりにバッククォートのパス文字列を記載し、エージェントにファイル読み取りツール（readFile）での動的読み込みを指示する。

```markdown
## タスク別の参照プロンプト
**選択されたタスクに対応するプロンプトファイルを読み取り、指示に従うこと。**

| タスク | プロンプトファイル |
|---|---|
| リポジトリ概要把握 | `.github/prompts/implementation/analyze-codebase.prompt.md` |
```

| 評価軸 | 結果 |
|--------|------|
| スラッシュコマンド | メニューのみ。個別プロンプトは不可 |
| コンテキスト効率 | 必要なプロンプトのみロード（理論上） |
| ディレクトリの見通し | 階層で整理される |
| 確実性 | **AIの判断に依存。ファイルを読まない可能性がゼロではない** |
| 運用コスト | 低い |

**不採択の理由**: 「必ずファイルを読め」という指示を書いても、AIが確実に従う保証がない。hookのような機械的な仕組みではなく、指示文の強度に依存する点が許容できない。

### 方式D: Agent Skills（段階的ロード）【採択】

```
.github/
├── prompts/
│   ├── cd-menu.prompt.md          ← メニューはプロンプトとして残す
│   └── ut-menu.prompt.md
├── skills/
│   ├── cd-analyze-codebase/
│   │   └── SKILL.md               ← name/description + 指示本文
│   ├── cd-analyze-spec/
│   │   └── SKILL.md
│   ├── ut-create-test/
│   │   ├── SKILL.md
│   │   └── templates/             ← テンプレート等もバンドル可能
│   └── ...
└── agents/
    ├── implementer.agent.md
    └── tester.agent.md
```

Agent Skills（2025年12月GA）の **progressive loading**（段階的ロード）を活用する。

**段階的ロードの仕組み（VS Code公式ドキュメント）**:

1. **Discovery** — Copilotが全スキルの `name` と `description` のみを読み取る（軽量）
2. **Instructions Loading** — ユーザーのタスクと関連性ありと判断した場合に `SKILL.md` 本文をコンテキストに注入
3. **Resource Access** — スキルディレクトリ内の追加ファイル（テンプレート等）は参照時のみアクセス

| 評価軸 | 結果 |
|--------|------|
| スラッシュコマンド | メニューのみ。スキルはCopilotが自動選択 |
| コンテキスト効率 | **関連スキルのみ選択的にロード（最良）** |
| ディレクトリの見通し | スキル単位でディレクトリが分かれ、階層的に整理される |
| 確実性 | **Copilotのメタデータベース選択ロジック（方式Cより高い）** |
| 運用コスト | 低い（ファイル配置のみ、追加インフラ不要） |

### 方式E: MCP Server経由の動的注入

MCPサーバーを構築し、タスク種別に応じてプロンプト内容を動的に返す。

| 評価軸 | 結果 |
|--------|------|
| スラッシュコマンド | 不要（MCPツール経由） |
| コンテキスト効率 | 必要なプロンプトのみロード |
| 確実性 | 高い（サーバーが機械的に返す） |
| 運用コスト | **高い（サーバー構築・運用・Enterprise環境でのポリシー設定が必要）** |

**不採択の理由**: SIer組織での横展開を考えると、MCPサーバーの構築・運用はハードルが高すぎる。プロンプトの整理という課題に対してオーバーエンジニアリング。

---

## 4. 決定事項

**方式D: Agent Skills（段階的ロード）を採用する。**

### 採択理由

1. **コンテキスト効率と確実性のバランスが最良**: 方式B（全ロード）の無駄と方式C（AI判断依存）の不確実性を同時に解消する
2. **追加インフラ不要**: `.github/skills/` にファイルを配置するだけ。SIer環境への展開障壁が低い
3. **自然な階層管理**: スキルごとにディレクトリが分かれるため、工程が増えても見通しが維持される
4. **テンプレート等のバンドル**: スキルディレクトリ内にテンプレートやサンプルコードを同梱でき、プロンプトとリソースを一体管理できる
5. **エージェントとの親和性**: `@implementer` がタスクに応じて必要なスキルを自動選択する動作は、現在のメニュー → プロンプト選択の流れと自然に置き換わる

### メニューのエージェント統合

当初はメニュープロンプト（`cd-menu.prompt.md` / `ut-menu.prompt.md`）を `.github/prompts/` に残す方針だったが、追加検討の結果、メニュー内容をエージェントファイル（`implementer.agent.md` / `tester.agent.md`）に統合した。

**統合の理由**:
- `@implementer` 起動 → タスク選択 → スキル自動ロード、の1本の導線に集約できる
- `@implementer` → `/cd-menu` の2段階呼び出しが不要になる
- `.github/prompts/` にメニューだけ残る中途半端な構成が解消される
- 失うもの（`/cd-menu` スラッシュコマンド）は `@implementer` で完全に代替可能

この結果、`.github/prompts/` ディレクトリは不要となり削除した。

---

## 5. 今後の検証事項

Skills は2025年12月にGAとなった比較的新しい機能のため、移行前に以下の検証が必要。

| 検証項目 | 確認内容 |
|----------|----------|
| 基本動作 | VS Code + Copilot Business/Enterprise 環境でスキルが正しく認識されるか |
| 段階的ロード | description に基づく関連性判断が期待通りに動作するか（不要なスキルがロードされないか） |
| エージェント連携 | `@implementer` / `@tester` のタスクメニューからスキルが自動的に利用されるか |
| Resource Access | SKILL.md以外のリソース（テンプレート等）が参照時のみロードされるか |
| 移行互換性 | 既存の `.prompt.md` から `SKILL.md` への変換で指示内容が劣化しないか |

---

## 6. 参考情報

### 公式ドキュメント

- [About agent skills - GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Creating agent skills - GitHub Docs](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-skills)
- [Agent Skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [GitHub Changelog: Agent Skills発表（2025-12-18）](https://github.blog/changelog/2025-12-18-github-copilot-now-supports-agent-skills/)

### 関連する制約・Issue

- [microsoft/vscode#268780](https://github.com/microsoft/vscode/issues/268780) — `.github/prompts/` のサブディレクトリ非対応
- [VS Code docs: Prompt files](https://code.visualstudio.com/docs/copilot/customization/prompt-files) — Markdownリンク参照の仕様
- [VS Code docs: Custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions) — `.instructions.md` + `applyTo`

### GitHub Copilot カスタマイズ階層の全体像

| 仕組み | 配置先 | ロードタイミング | サブディレクトリ |
|--------|--------|-----------------|----------------|
| copilot-instructions.md | `.github/` | 常時自動 | N/A |
| .instructions.md + applyTo | `.github/instructions/` | 対象ファイル操作時に自動 | 対応 |
| Prompt Files | `.github/prompts/` | スラッシュコマンドで手動 | **非対応** |
| Agent Skills | `.github/skills/` | Copilotが関連性判断で選択的ロード | 対応（スキル単位でディレクトリ） |
| Custom Agents | `.github/agents/` | `@name` で手動 | N/A |
| MCP Server | `.vscode/mcp.json` | エージェントモードでツール呼び出し時 | N/A |
