---
layout: default
title: ダッシュボード
nav_order: 2
---

# コンテンツダッシュボード

AI駆動開発プレイブックのコンテンツ構造を可視化します。

---

## カテゴリ別ページ数

<div style="max-width: 480px; margin: 0 auto;">
<canvas id="categoryChart"></canvas>
</div>

---

## コンテンツマップ

```mermaid
graph TD
    ROOT["🏠 AI駆動開発プレイブック"]

    ROOT --> COMMON["📘 共通ガイド<br/>4ページ"]
    ROOT --> IMPL["⚙️ 実装工程<br/>9ページ"]
    ROOT --> UT["🧪 単体テスト工程<br/>6ページ"]
    ROOT --> RESEARCH["🔬 リサーチ<br/>3ページ"]
    ROOT --> REVIEWS["📝 レビュー記録<br/>9ページ"]
    ROOT --> REF["📂 参考資料"]

    COMMON --> QS["クイックスタート"]
    COMMON --> MODES["Copilotモード解説"]
    COMMON --> CHK["レビューチェックリスト"]
    COMMON --> ANTI["アンチパターン集"]

    IMPL --> UC01["UC01: コード骨格生成"]
    IMPL --> UC02["UC02: 既存コード改修"]
    IMPL --> UC03["UC03: リファクタリング"]
    IMPL --> UC04["UC04: コードベース理解"]
    IMPL --> UC05["UC05: レビュー前チェック"]
    IMPL --> UC06["UC06: 修正箇所特定"]
    IMPL --> UC07["UC07: エラーログ解析"]
    IMPL --> UC08["UC08: 修正方針策定"]

    UT --> UT01["UC01: テスト自動生成"]
    UT --> UT02["UC02: テストデータ生成"]
    UT --> UT03["UC03: カバレッジ補完"]
    UT --> UT04["UC04: テストコード改善"]
    UT --> UT05["UC05: テスト仕様設計"]

    RESEARCH --> ADR["ADR: プロンプト構成設計"]
    RESEARCH --> POC["PoC ペットクリニック計画"]
    RESEARCH --> OSS["Spring Boot OSS候補"]

    REF --> AGENTS["エージェント 2件"]
    REF --> SKILLS["スキル 18件"]
    REF --> CLAUDE["Claude Code<br/>コマンド 5件"]

    SKILLS --> CD["実装系 11件"]
    SKILLS --> UTS["テスト系 5件"]
    SKILLS --> SETUP["セットアップ系 2件"]

    style ROOT fill:#1a73e8,color:#fff,stroke:none
    style COMMON fill:#34a853,color:#fff,stroke:none
    style IMPL fill:#ea4335,color:#fff,stroke:none
    style UT fill:#fbbc04,color:#333,stroke:none
    style RESEARCH fill:#9334e6,color:#fff,stroke:none
    style REVIEWS fill:#e67c73,color:#fff,stroke:none
    style REF fill:#607d8b,color:#fff,stroke:none
```

---

## スキル一覧と分類

<div style="max-width: 600px; margin: 0 auto;">
<canvas id="skillChart"></canvas>
</div>

### スキル詳細

<table>
  <thead>
    <tr>
      <th>カテゴリ</th>
      <th>サブカテゴリ</th>
      <th>スキル名</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
{% for skill in site.data.skills %}
    <tr>
      <td>{{ skill.category }}</td>
      <td>{{ skill.subcategory }}</td>
      <td><code>/{{ skill.name }}</code></td>
      <td>{{ skill.description }}</td>
    </tr>
{% endfor %}
  </tbody>
</table>

---

## レビュー履歴

<style>
.timeline {
  position: relative;
  padding: 20px 0;
}
.timeline::before {
  content: '';
  position: absolute;
  left: 20px;
  top: 0;
  bottom: 0;
  width: 3px;
  background: #1a73e8;
}
.timeline-item {
  position: relative;
  margin-left: 50px;
  margin-bottom: 30px;
  padding: 16px 20px;
  background: #f8f9fa;
  border-radius: 8px;
  border-left: 4px solid #1a73e8;
}
.timeline-item::before {
  content: '';
  position: absolute;
  left: -38px;
  top: 20px;
  width: 12px;
  height: 12px;
  background: #1a73e8;
  border-radius: 50%;
  border: 3px solid #fff;
}
.timeline-date {
  font-weight: bold;
  font-size: 1.1em;
  color: #1a73e8;
  margin-bottom: 8px;
}
.timeline-scores {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  margin: 8px 0;
}
.timeline-score {
  background: #fff;
  padding: 4px 12px;
  border-radius: 4px;
  font-size: 0.9em;
  border: 1px solid #e0e0e0;
}
.timeline-desc {
  color: #555;
  font-size: 0.95em;
  margin-top: 8px;
}
</style>

<div class="timeline">
  <div class="timeline-item">
    <div class="timeline-date">2026-04-02 #01</div>
    <div>初版ドラフトの全体レビュー</div>
    <div class="timeline-scores">
      <span class="timeline-score">PM: 4/5</span>
      <span class="timeline-score">リーダー: 4/5</span>
      <span class="timeline-score">開発者: 4/5</span>
    </div>
    <div class="timeline-desc">
      「読まれる設計」「段階的導入パス（UC04から）」「人間が確認すべきこと + よくある落とし穴の二段構え」が全ロールから高評価
    </div>
  </div>
  <div class="timeline-item">
    <div class="timeline-date">2026-04-03 #01</div>
    <div>Copilotネイティブ設定導入後のレビュー</div>
    <div class="timeline-scores">
      <span class="timeline-score">PM: 4/5</span>
      <span class="timeline-score">リーダー: 4/5</span>
      <span class="timeline-score">開発者: 4/5</span>
    </div>
    <div class="timeline-desc">
      全ロール維持（質的向上）。Copilotネイティブ設定への移行が完了し、実用性がさらに向上
    </div>
  </div>
</div>

---

## サマリー

| 指標 | 値 |
|------|-----|
| 総ページ数 | 57 |
| ドキュメントページ | 32 |
| 参考資料ページ | 25 |
| ユースケース数 | 13（実装8 + テスト5） |
| スキル数 | 18（セットアップ2 + 実装11 + テスト5） |
| エージェント数 | 2（@implementer, @tester） |
| レビューセッション | 2回 |

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
  // Category donut chart
  new Chart(document.getElementById('categoryChart'), {
    type: 'doughnut',
    data: {
      labels: ['共通ガイド', '実装工程', '単体テスト工程', 'リサーチ', 'レビュー記録', '要件・背景', '参考資料（GitHub）', '参考資料（Claude）'],
      datasets: [{
        data: [4, 9, 6, 3, 9, 1, 21, 5],
        backgroundColor: [
          '#34a853', '#ea4335', '#fbbc04', '#9334e6',
          '#e67c73', '#78909c', '#4285f4', '#ff6d01'
        ],
        borderWidth: 2,
        borderColor: '#fff'
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'bottom', labels: { font: { size: 12 } } },
        title: { display: true, text: 'カテゴリ別ページ構成', font: { size: 16 } }
      }
    }
  });

  // Skill bar chart
  new Chart(document.getElementById('skillChart'), {
    type: 'bar',
    data: {
      labels: ['セットアップ', '実装 — 分析', '実装 — 変更', '実装 — 生成', '実装 — 確認', '単体テスト — 設計', '単体テスト — 生成', '単体テスト — 分析', '単体テスト — 変更'],
      datasets: [{
        label: 'スキル数',
        data: [2, 6, 3, 1, 1, 1, 2, 1, 1],
        backgroundColor: [
          '#607d8b',
          '#ef5350', '#ef5350', '#ef5350', '#ef5350',
          '#fdd835', '#fdd835', '#fdd835', '#fdd835'
        ],
        borderRadius: 4
      }]
    },
    options: {
      indexAxis: 'y',
      responsive: true,
      plugins: {
        legend: { display: false },
        title: { display: true, text: 'スキル分類別の内訳', font: { size: 16 } }
      },
      scales: {
        x: { beginAtZero: true, ticks: { stepSize: 1 } }
      }
    }
  });
});
</script>
