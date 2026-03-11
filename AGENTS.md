# AGENTS.md

このファイルは Claude Code のマルチエージェント機能で使用する **エージェント定義ファイル** です。
各エージェントの役割・権限・使用スキルを定義します。

> **参考**: Claude Code の AGENTS.md については [公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code) を参照してください。

---

## エージェント一覧

| エージェント | 役割 | 使用スキル |
|---|---|---|
| Architect Agent | システム設計 | api-design |
| Implementation Agent | コード生成 | api-design |
| Review Agent | コードレビュー | code-review |

---

## Architect Agent

**ファイル**: `agents/architect-agent.md`

```
役割: システムアーキテクチャの設計
入力: 仕様書 (spec/)
出力: アーキテクチャ設計ドキュメント、コンポーネント図
```

### 責務

- `spec/` ディレクトリの仕様書を読み込み、システム全体のアーキテクチャを設計する
- コンポーネントの分割方法と依存関係を定義する
- API エンドポイントの設計を行う
- 技術選定の根拠を説明する

### 制約

- `spec/` ディレクトリは読み取り専用（変更不可）
- `CLAUDE.md` に定義された技術スタックに従う
- コードは生成しない（設計ドキュメントのみ）

---

## Implementation Agent

**ファイル**: `agents/implementation-agent.md`

```
役割: 仕様とアーキテクチャに基づくコード生成
入力: 仕様書 (spec/)、アーキテクチャ設計
出力: Python / FastAPI 実装コード
```

### 責務

- アーキテクチャ設計に従って FastAPI コードを生成する
- `CLAUDE.md` のコーディング規約に従う
- 型ヒント・ドキュメント文字列・テストコードを含める

### 制約

- `spec/` ディレクトリは読み取り専用
- `agents/` ディレクトリは読み取り専用
- `src/` ディレクトリにのみコードを生成する

---

## Review Agent

**ファイル**: `agents/review-agent.md`

```
役割: 生成されたコードのレビューと品質チェック
入力: 実装コード (src/)
出力: レビューコメント、改善提案
```

### 責務

- セキュリティ上の問題を検出する
- パフォーマンスのボトルネックを指摘する
- コーディング規約への準拠を確認する
- 可読性・保守性の改善提案を行う

### 制約

- コードを直接変更しない（コメント・提案のみ）
- `CLAUDE.md` の規約を基準にレビューする

---

## エージェント連携フロー

```
spec/enterprise-search.md
         │
         ▼
  Architect Agent
  (アーキテクチャ設計)
         │
         ▼
Implementation Agent
  (コード生成)
         │
         ▼
   Review Agent
  (品質レビュー)
         │
         ▼
     PR / Merge
```

---

## サブエージェント

### api-designer

```
スキル: skills/api-design.md
用途: API エンドポイントの詳細設計
使用タイミング: Implementation Agent が API 設計の詳細を必要とする場合
```

### security-reviewer

```
スキル: skills/code-review.md (セキュリティセクション)
用途: セキュリティ観点でのコードレビュー
使用タイミング: Review Agent がセキュリティ専門のレビューを必要とする場合
```

### test-generator

```
スキル: skills/api-design.md (テストセクション)
用途: テストコードの自動生成
使用タイミング: Implementation Agent がテストコードの生成を依頼する場合
```
