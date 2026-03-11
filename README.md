# Claude Code Spec-Driven Development Handson

Claude Code を使ったモダンな AI ネイティブ開発を体験するハンズオンリポジトリです。

## 学べること

- **Spec Driven Development** — 仕様書から AI がコードを生成するフローを体験
- **Context Files** — `CLAUDE.md` / `AGENTS.md` で AI の振る舞いを定義
- **Multi-Agent Development** — Architect・Implementer・Reviewer の役割分担を体験
- **Progressive Workflow** — Spec → Design → Implement → Review を段階的に進める
- **Subagents / Skills** — 再利用可能なスキルとサブエージェントを活用する

## ゴール

```
Spec → AI 実装 → AI レビュー → PR
```

という AI 開発フローを実際に体験する。

---

## 開発テーマ: Enterprise Knowledge Search API

本ハンズオンでは **Enterprise Knowledge Search API** を題材として開発します。

```
GET /search?q=azure
```

技術スタック:
- **FastAPI** (Python)
- 検索 API エンドポイント
- AI Summarization 機能

---

## ハンズオン手順

### Step 1 — 仕様を書く (Spec)

[exercises/step1-spec.md](exercises/step1-spec.md) を開き、`spec/enterprise-search.md` を参照しながら仕様を理解します。

Claude Code プロンプト例:
```
Read spec/enterprise-search.md and summarize the requirements.
```

### Step 2 — アーキテクチャを設計する

[exercises/step2-architecture.md](exercises/step2-architecture.md) を参照してください。

Claude Code プロンプト例:
```
Read spec/enterprise-search.md and design system architecture. Use FastAPI.
```

### Step 3 — コードを生成する

[exercises/step3-implementation.md](exercises/step3-implementation.md) を参照してください。

Claude Code プロンプト例:
```
Generate FastAPI implementation code from the architecture we designed.
```

### Step 4 — AI レビューを実施する

[exercises/step4-review.md](exercises/step4-review.md) を参照してください。

Claude Code プロンプト例:
```
Review this code for security, performance, and maintainability.
```

---

## リポジトリ構成

```
claude-code-spec-driven-dev-handson/
│
├── README.md               # このファイル
├── CLAUDE.md               # Claude Code 向けコンテキストファイル
├── AGENTS.md               # エージェント定義ファイル
│
├── docs/
│   ├── intro.md            # ハンズオン概要
│   ├── spec-driven-dev.md  # Spec Driven Development 解説
│   └── claude-code-basics.md  # Claude Code 基礎
│
├── spec/
│   └── enterprise-search.md   # Enterprise Search API 仕様書
│
├── agents/
│   ├── architect-agent.md     # アーキテクト・エージェント定義
│   ├── implementation-agent.md  # 実装エージェント定義
│   └── review-agent.md        # レビューエージェント定義
│
├── skills/
│   ├── api-design.md          # API 設計スキル
│   └── code-review.md         # コードレビュースキル
│
└── exercises/
    ├── step1-spec.md           # Step 1: 仕様を書く
    ├── step2-architecture.md   # Step 2: アーキテクチャ設計
    ├── step3-implementation.md # Step 3: 実装
    └── step4-review.md         # Step 4: AI レビュー
```

---

## 始め方

1. **Claude Code** をインストールする（[公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code) 参照）
2. このリポジトリをクローンする
   ```bash
   git clone https://github.com/ishidahra01/claude-code-spec-driven-dev-handson.git
   cd claude-code-spec-driven-dev-handson
   ```
3. [docs/intro.md](docs/intro.md) を読んで概要を把握する
4. [exercises/step1-spec.md](exercises/step1-spec.md) から順番に進める

---

## 関連ドキュメント

| ドキュメント | 内容 |
|---|---|
| [docs/intro.md](docs/intro.md) | ハンズオン全体の概要 |
| [docs/spec-driven-dev.md](docs/spec-driven-dev.md) | Spec Driven Development の解説 |
| [docs/claude-code-basics.md](docs/claude-code-basics.md) | Claude Code の基礎知識 |
| [CLAUDE.md](CLAUDE.md) | AI コンテキスト設定 |
| [AGENTS.md](AGENTS.md) | エージェント定義 |
