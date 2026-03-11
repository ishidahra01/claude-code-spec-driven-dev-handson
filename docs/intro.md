# ハンズオン概要

## Claude Code Spec-Driven Development Handson とは

このハンズオンは **Claude Code** を使ったモダンな AI ネイティブ開発スタイルを体験するための実践的なワークショップです。

単なる「AI にコードを書かせる」体験ではなく、**仕様駆動でシステム全体を AI と協調して開発する** 現代的なワークフローを習得します。

---

## 対象者

- Claude Code を使い始めたばかりのエンジニア
- AI を活用した開発フローを体系的に学びたいエンジニア
- チームでの AI 活用方法を検討しているエンジニア・テックリード

## 前提知識

- Python の基礎知識
- REST API の概念
- Git の基本操作

---

## ハンズオンで学ぶ 5 つのトピック

### 1. Spec Driven Development

仕様書（`spec/enterprise-search.md`）を起点に、AI がコードを生成します。
「何を作るか」を明確に定義することで、AI の出力品質が大幅に向上します。

```
仕様書 → AI がアーキテクチャ設計 → AI がコード生成 → AI がレビュー
```

詳細: [spec-driven-dev.md](spec-driven-dev.md)

---

### 2. Context Files (CLAUDE.md / AGENTS.md)

AI の振る舞いをファイルで定義します。

| ファイル | 目的 |
|---|---|
| `CLAUDE.md` | 技術スタック・コーディング規約・アーキテクチャ方針を定義 |
| `AGENTS.md` | エージェントの役割・責務・制約を定義 |

これらのファイルを用意することで、AI が毎回同じ基準でコードを生成・レビューするようになります。

---

### 3. Multi-Agent Development

複数のエージェントが異なる役割を担って協調します。

```
Architect Agent → Implementation Agent → Review Agent
```

各エージェントは専門領域に集中することで、より高品質な成果物を生み出します。

---

### 4. Progressive Workflow

段階的なワークフローで複雑なシステムを構築します。

```
Step 1: 仕様を書く・読む
Step 2: アーキテクチャを設計する
Step 3: コードを生成する
Step 4: AI でレビューする
```

---

### 5. Subagents / Skills

再利用可能なスキルとサブエージェントを活用します。

```
skills/api-design.md    → API 設計のベストプラクティス
skills/code-review.md   → レビュー観点のチェックリスト
```

---

## ハンズオンの進め方

```
docs/intro.md (このファイル)
  ↓
docs/claude-code-basics.md   ← Claude Code の基礎を確認
  ↓
docs/spec-driven-dev.md      ← Spec Driven Dev を理解
  ↓
exercises/step1-spec.md      ← 実際に手を動かす
  ↓
exercises/step2-architecture.md
  ↓
exercises/step3-implementation.md
  ↓
exercises/step4-review.md
```

所要時間の目安: **約 2〜3 時間**

---

## 準備するもの

1. **Claude Code** のインストールと認証設定
   ```bash
   # Claude Code CLI のインストール
   npm install -g @anthropic-ai/claude-code
   
   # 認証
   claude auth login
   ```

2. **Python 3.11+** のインストール

3. このリポジトリのクローン
   ```bash
   git clone https://github.com/ishidahra01/claude-code-spec-driven-dev-handson.git
   cd claude-code-spec-driven-dev-handson
   ```

準備ができたら [exercises/step1-spec.md](../exercises/step1-spec.md) から始めましょう！
