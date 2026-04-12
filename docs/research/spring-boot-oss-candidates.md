---
layout: default
title: "Spring Boot OSS 候補"
parent: "リサーチ"
nav_order: 3
---

# ユニットテスト練習用 Spring Boot OSSプロジェクト候補

## 目的
AI駆動開発標準を活用して「ユニットテスト作成 → フィードバック → 改修」サイクルを回す練習に適したOSSプロジェクトの選定。

## 選定基準
- Java + Spring Boot
- GitHubリポジトリが公開されている（Stars 100以上）
- ローカル環境で起動可能
- 実際の業務ロジックがある（チュートリアル骨格ではない）

---

## 候補一覧

### 1. spring-petclinic（動物病院管理システム）

| 項目 | 内容 |
|------|------|
| GitHub | https://github.com/spring-projects/spring-petclinic |
| Stars | ~7,600 |
| 技術スタック | Spring Boot + Spring Data JPA + Thymeleaf + H2/MySQL/PostgreSQL |
| ローカル起動 | H2組み込みDBで即起動可能。docker-composeあり |

**概要**: Spring公式リファレンスアプリ。飼い主・ペット・診察・獣医のCRUD管理。Controller/Service/Repositoryの典型的レイヤード構成。

**テスト練習への適性**:
- 既存テストが含まれており「読む→理解→追加」の流れが作りやすい
- コードベースが小さく全体像を把握しやすい
- Spring公式のため模範的なアーキテクチャ

**推奨度**: ★★★★★（初回の題材として最適）

---

### 2. spring-boot-realworld-example-app（Medium風ブログAPI）

| 項目 | 内容 |
|------|------|
| GitHub | https://github.com/gothinkster/spring-boot-realworld-example-app |
| Stars | ~1,400 |
| 技術スタック | Spring Boot + MyBatis + Spring Security + JWT + H2 |
| ローカル起動 | `./gradlew bootRun` で即起動 |

**概要**: RealWorld API仕様に準拠したMedium風ブログバックエンド。ユーザー認証（JWT）、記事CRUD、コメント、お気に入り、フォロー、タグ管理。

**テスト練習への適性**:
- DDD/CQRSパターンでRead/Writeモデルが明確に分離
- 既存テストが充実しており参考にしやすい
- API仕様が標準化されているためテスト観点が立てやすい

**推奨度**: ★★★★★（設計パターン学習と両立したい場合に最適）

---

### 3. piggymetrics（個人家計簿マイクロサービス）

| 項目 | 内容 |
|------|------|
| GitHub | https://github.com/sqshq/piggymetrics |
| Stars | ~13,800 |
| 技術スタック | Spring Boot + Spring Cloud + MongoDB + Docker |
| ローカル起動 | docker-composeで全サービス一括起動 |

**概要**: 個人の収支を追跡するマイクロサービスアプリ。Account Service、Statistics Service、Notification Serviceの3サービス構成。

**テスト練習への適性**:
- 各サービスが独立した業務ロジック（金融計算、統計集約、通知ルール）を持つ
- マイクロサービス間のテスト戦略を学べる
- サービス単位が小さいため取り組みやすい

**推奨度**: ★★★★☆（マイクロサービスのテスト戦略を学びたい場合）

---

### 4. shopizer（ECプラットフォーム）

| 項目 | 内容 |
|------|------|
| GitHub | https://github.com/shopizer-ecommerce/shopizer |
| Stars | ~3,000 |
| 技術スタック | Spring Boot + Hibernate/JPA + Spring Security + MySQL/H2 |
| ローカル起動 | Docker対応あり |

**概要**: 本格的なオープンソースECプラットフォーム。商品カタログ、ショッピングカート、チェックアウト、決済連携、税計算、配送ルール、マルチストア対応。

**テスト練習への適性**:
- 価格ルール・税計算・在庫管理・注文処理など、SIer案件に近い複雑な業務ロジック
- サービス層が充実しておりテスト対象が豊富
- 実運用レベルのコマース基盤

**推奨度**: ★★★★☆（SIer案件に近い複雑さで練習したい場合）

---

### 5. mall（大規模ECシステム）

| 項目 | 内容 |
|------|------|
| GitHub | https://github.com/macrozheng/mall |
| Stars | ~79,200 |
| 技術スタック | Spring Boot + MyBatis + MySQL + Redis + Elasticsearch + RabbitMQ + MongoDB |
| ローカル起動 | Docker完全対応 |

**概要**: フロントエンド店舗 + バックエンド管理を含む完全なECシステム。商品管理、注文管理、会員管理、プロモーション、ショッピングカート、コンテンツ管理、経理レポート、権限管理。

**テスト練習への適性**:
- 膨大な業務ロジック（商品カタログ、注文、決済、プロモーション）
- 複数ドメインにまたがるサービス層が充実
- ただしコードベースが非常に大きいため、特定モジュールに絞って取り組む必要あり

**推奨度**: ★★★☆☆（上級者向け。特定モジュールに絞れば有効）

---

## 推奨選定フロー

```
初回・チーム研修 → spring-petclinic
  ↓ 慣れてきたら
設計パターン学習 → realworld-example-app
  ↓ さらに発展
SIer案件に近い練習 → shopizer
マイクロサービス練習 → piggymetrics
```

## 調査日
2026-04-03
