# Step 2 — アーキテクチャを設計する

## このステップの目的

Architect Agent を使って、`spec/enterprise-search.md` の仕様書からシステムアーキテクチャを設計します。
AI がどのように仕様から設計を行うか、そのプロセスを体験します。

---

## 所要時間

約 30〜40 分

---

## 前提条件

- [Step 1](step1-spec.md) が完了していること
- 仕様書（`spec/enterprise-search.md`）の内容を把握していること

---

## タスク 1: アーキテクチャ設計を依頼する

Architect Agent の役割で Claude Code にアーキテクチャ設計を依頼します。

### Claude Code プロンプト

```
You are the Architect Agent defined in AGENTS.md.

Read the following files:
- spec/enterprise-search.md
- CLAUDE.md
- agents/architect-agent.md

Design the system architecture for the Enterprise Knowledge Search API.

Please provide:
1. Component breakdown with responsibilities for each component
2. Directory structure for the src/ implementation
3. Data flow: from HTTP request to response
4. Key design decisions and their rationale

Constraints:
- Use FastAPI (as defined in CLAUDE.md)
- Follow the layered architecture in CLAUDE.md
- Do NOT generate implementation code yet
```

### 期待される出力例

```
## System Architecture: Enterprise Knowledge Search API

### Components

1. FastAPI Application (main.py)
   - Registers routers and middleware
   - Configures CORS, exception handlers

2. Auth Middleware (core/auth.py)
   - Validates Bearer API key
   - Returns 401 if invalid

3. Search Router (routers/search.py)
   - Handles GET /api/v1/search
   - Delegates to SearchService

4. Search Service (services/search_service.py)
   - Core search logic
   - Calls SummaryService

5. Summary Service (services/summary_service.py)
   - Calls Claude API to generate summary

### Directory Structure
src/
  main.py
  routers/
    search.py
    health.py
  services/
    search_service.py
    summary_service.py
  models/
    request.py
    response.py
  core/
    config.py
    auth.py
```

---

## タスク 2: 設計を深掘りする

生成された設計について、詳細を確認します。

### Claude Code プロンプト（フォローアップ）

```
Based on the architecture you designed, answer these questions:

1. How does the SearchService return mock data for development 
   (before connecting to a real search index)?

2. How should we handle the case where the Claude API (summary service) 
   is slow or unavailable?

3. What environment variables does this system need?
   Create a list in .env.example format.
```

---

## タスク 3: アーキテクチャを評価する

設計の品質をレビューします。

### 確認ポイント

以下の観点で生成されたアーキテクチャを評価してください:

| 観点 | チェック項目 |
|---|---|
| 分離 | 各コンポーネントの責務が明確に分かれているか |
| 拡張性 | 将来的に検索バックエンドを変更できるか |
| テスト容易性 | サービス層をモックしてルーターをテストできるか |
| セキュリティ | 認証レイヤーが独立しているか |

### Claude Code プロンプト（評価）

```
Review the architecture we designed against these quality criteria:
1. Separation of concerns — each component has a single responsibility
2. Testability — can we unit test each layer in isolation?
3. Extensibility — how easy would it be to swap the search backend?
4. Security — is authentication handled consistently?

Provide specific feedback and any suggested improvements.
```

---

## タスク 4: ディレクトリ構成を作成する（実習）

設計したディレクトリ構成を実際に作成します。

### Claude Code プロンプト

```
Based on the architecture we designed, create the directory structure for src/.

Create empty Python files with module-level docstrings only.
Do NOT implement any logic yet — just the file structure.

Also create:
- requirements.txt with the necessary dependencies
- .env.example with required environment variables
```

---

## 学習ポイント

このステップで学んだこと:

- ✅ Architect Agent の役割と AGENTS.md の使い方を理解した
- ✅ 仕様書からアーキテクチャを設計するプロセスを体験した
- ✅ コンポーネント分離の重要性を理解した
- ✅ AI 設計のフォローアップ質問の重要性を理解した

---

## 次のステップ

[exercises/step3-implementation.md](step3-implementation.md) に進んで、アーキテクチャを実装します。
