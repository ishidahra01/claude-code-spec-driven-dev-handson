# Claude Code 基礎

## Claude Code とは

**Claude Code** は Anthropic が提供する AI コーディングエージェントです。
ターミナルから直接利用でき、コードの生成・編集・レビュー・テスト実行などを AI と対話しながら行えます。

> 公式ドキュメント: https://docs.anthropic.com/en/docs/claude-code

---

## インストールと初期設定

### インストール

```bash
npm install -g @anthropic-ai/claude-code
```

### 認証

```bash
claude auth login
```

ブラウザが開き、Anthropic アカウントでの認証が求められます。

### 動作確認

```bash
claude --version
```

---

## 基本的な使い方

### プロジェクトで起動する

```bash
cd your-project/
claude
```

プロジェクトルートで起動すると、`CLAUDE.md` が自動的に読み込まれます。

### 単発でコマンドを実行する

```bash
claude "Explain what this project does"
claude "Generate a FastAPI endpoint for /search"
```

### ファイルを指定して処理する

```bash
claude "Review this file for security issues" src/main.py
```

---

## CLAUDE.md によるコンテキスト設定

`CLAUDE.md` はプロジェクトルートに置くことで、Claude Code が自動的に読み込みます。

```markdown
# CLAUDE.md

## Tech Stack
- Python 3.11
- FastAPI
- Pydantic v2

## Coding Standards
- Use type hints everywhere
- Follow Google docstring style
```

**効果**: 毎回同じ技術スタック・規約でコードが生成されるようになります。

---

## AGENTS.md によるエージェント定義

`AGENTS.md` ではマルチエージェント機能で使用するエージェントを定義します。

```markdown
# AGENTS.md

## Architect Agent
Role: Design system architecture
Input: spec/
Output: Architecture documents

## Review Agent
Role: Code quality review
Constraints: Read-only, no code changes
```

---

## よく使うプロンプトパターン

### コード生成

```
Generate [component] for [purpose].
Use [technology].
Follow the standards in CLAUDE.md.
```

例:
```
Generate a FastAPI router for the search endpoint.
Use Pydantic v2 for request/response models.
Follow the standards in CLAUDE.md.
```

### コードレビュー

```
Review [file/code] for:
- Security vulnerabilities
- Performance issues
- Code quality
```

### リファクタリング

```
Refactor [file] to:
- Improve readability
- Add type hints
- Follow the pattern in [reference file]
```

### テスト生成

```
Generate unit tests for [file].
Use pytest and pytest-asyncio.
Cover edge cases and error scenarios.
```

### ドキュメント生成

```
Generate API documentation for [router file].
Include request/response examples.
```

---

## マルチエージェント機能

Claude Code では複数のエージェントを連携させることができます。

### エージェントの呼び出し

```
Use the Architect Agent to design the system.
Then pass the design to the Implementation Agent.
```

### サブエージェントの活用

```
Use the api-designer subagent from AGENTS.md to design the search endpoint.
```

---

## ファイル操作の権限制御

`AGENTS.md` でエージェントのファイルアクセス権限を制御できます。

```markdown
## Review Agent
Constraints:
- Read: src/, tests/, spec/
- Write: (none - review only)
```

---

## ベストプラクティス

### 1. 具体的なプロンプトを書く

❌ 悪い例:
```
Make a search API
```

✅ 良い例:
```
Create a FastAPI GET endpoint at /api/v1/search that:
- Accepts query parameter 'q' (required, string)
- Accepts query parameter 'limit' (optional, int, default 10)
- Returns SearchResponse model with results and summary
- Raises 400 if q is empty
- Follow the standards in CLAUDE.md
```

### 2. コンテキストを提供する

```
Read spec/enterprise-search.md first, then generate the implementation.
```

### 3. 段階的に進める

一度にすべてを生成させず、段階的に進めます:

```
Step 1: Generate data models only
Step 2: Generate service layer
Step 3: Generate router
Step 4: Generate tests
```

### 4. レビューを必ず実施する

```
Review the generated code for security and quality before using it.
```

### 5. CLAUDE.md を常に最新に保つ

プロジェクトの技術スタックや規約が変わったら、`CLAUDE.md` も更新します。

---

## トラブルシューティング

### AI が規約を守らない場合

`CLAUDE.md` の記述を具体化します:

```markdown
# NG
Use type hints

# OK
Add type annotations to ALL function parameters and return values.
Example:
  async def search(query: str, limit: int = 10) -> SearchResponse:
```

### 生成されたコードが期待と違う場合

仕様書（`spec/`）を詳細化し、再生成を依頼します:

```
The generated code doesn't match the spec. 
Re-read spec/enterprise-search.md section "API Design" and regenerate the search endpoint.
```
