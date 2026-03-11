# Step 3 — コードを生成する

## このステップの目的

Implementation Agent を使って、アーキテクチャ設計から FastAPI の実装コードを生成します。
AI がどのように設計をコードに変換するか、そのプロセスを体験します。

---

## 所要時間

約 40〜60 分

---

## 前提条件

- [Step 2](step2-architecture.md) が完了していること
- `src/` ディレクトリが作成されていること
- アーキテクチャ設計が完了していること

---

## タスク 1: データモデルを生成する

最初にリクエスト・レスポンスのデータモデルを生成します。

### Claude Code プロンプト

```
You are the Implementation Agent defined in AGENTS.md.

Read the following files:
- spec/enterprise-search.md
- CLAUDE.md
- skills/api-design.md

Generate Pydantic v2 models for src/models/response.py:
1. SearchResult — individual search result
2. SearchResponse — full API response

Requirements:
- Use Pydantic v2 BaseModel
- Add Field() with descriptions for all fields
- Add field validators where appropriate (e.g., score must be 0.0-1.0)
- Follow Google docstring style
- Add type hints to all fields
```

### 確認ポイント

生成されたモデルを確認:
- `score` フィールドに `ge=0.0, le=1.0` の制約があるか
- `limit` フィールドに `le=50` の制約があるか
- すべてのフィールドに `description` があるか

---

## タスク 2: 認証ミドルウェアを生成する

### Claude Code プロンプト

```
You are the Implementation Agent.

Read CLAUDE.md and skills/api-design.md.

Generate src/core/auth.py with:
- verify_api_key() dependency function
- Validates Authorization: Bearer <key> header
- Returns 401 if missing or invalid
- API key is loaded from settings (not hardcoded)

Also generate src/core/config.py with:
- Settings class using pydantic-settings
- api_key field (required)
- Load from .env file
```

---

## タスク 3: サービス層を生成する

### Claude Code プロンプト

```
You are the Implementation Agent.

Read:
- spec/enterprise-search.md
- CLAUDE.md

Generate src/services/search_service.py with:
- SearchService class
- search() async method
- For now, return MOCK DATA (we'll integrate real search later)
- Mock data should include 3 realistic-looking documents about various tech topics

Generate src/services/summary_service.py with:
- SummaryService class  
- generate_summary() async method
- For now, return a simple mock summary string
- Add a TODO comment where the real Claude API call will go
```

---

## タスク 4: ルーターを生成する

### Claude Code プロンプト

```
You are the Implementation Agent.

Read:
- spec/enterprise-search.md
- CLAUDE.md
- skills/api-design.md
- The models in src/models/response.py
- The services in src/services/

Generate src/routers/search.py with:
- GET /api/v1/search endpoint
- Query parameters: q (required), limit (default 10, max 50), offset (default 0)
- Uses verify_api_key dependency
- Calls SearchService and SummaryService
- Proper error handling (400 for validation, 401 for auth, 500 for server errors)
- Response model: SearchResponse
- Docstring

Also generate src/routers/health.py with:
- GET /health endpoint (no auth required)
- Returns {"status": "healthy", "version": "1.0.0"}
```

---

## タスク 5: アプリケーションエントリーポイントを生成する

### Claude Code プロンプト

```
You are the Implementation Agent.

Generate src/main.py:
- FastAPI app instance
- Include search and health routers
- Add app metadata (title, description, version)
- Add exception handler for unhandled exceptions (return 500 without leaking internals)
```

---

## タスク 6: テストコードを生成する

### Claude Code プロンプト

```
You are the Implementation Agent acting as test-generator subagent.

Generate tests/test_search.py with pytest tests for src/routers/search.py.

Test cases:
1. test_search_success — valid query, returns 200 with results and summary
2. test_search_empty_query — empty q parameter, expect 422
3. test_search_no_auth — missing Authorization header, expect 401
4. test_search_invalid_auth — wrong API key, expect 401
5. test_search_limit_too_large — limit=100, expect 422
6. test_health — GET /health, returns 200

Use:
- httpx.AsyncClient with app
- pytest-asyncio
- Mock the API key via environment variable or settings override
```

---

## 動作確認

コードが生成されたら、実際に動かして確認します。

### 依存関係のインストール

```bash
pip install fastapi uvicorn pydantic pydantic-settings httpx pytest pytest-asyncio
```

### 環境変数の設定

```bash
cp .env.example .env
# .env を編集して api_key を設定
echo "API_KEY=test-key-12345" > .env
```

### 開発サーバーの起動

```bash
uvicorn src.main:app --reload
```

### API の動作確認

```bash
# 検索 API を呼び出す
curl -H "Authorization: Bearer test-key-12345" \
     "http://localhost:8000/api/v1/search?q=azure"

# ヘルスチェック
curl http://localhost:8000/health

# Swagger UI でも確認
open http://localhost:8000/docs
```

### テストの実行

```bash
pytest tests/ -v
```

---

## 学習ポイント

このステップで学んだこと:

- ✅ Implementation Agent の役割と使い方を理解した
- ✅ 段階的なコード生成（モデル → サービス → ルーター → エントリーポイント）を体験した
- ✅ AI が `CLAUDE.md` のコーディング規約に従ってコードを生成することを確認した
- ✅ モックデータを使った段階的な実装アプローチを理解した

---

## 次のステップ

[exercises/step4-review.md](step4-review.md) に進んで、AI によるコードレビューを行います。
