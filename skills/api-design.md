# API Design Skill

## 概要

このスキルは FastAPI を使った REST API 設計のベストプラクティスと実装パターンを定義します。
**Architect Agent** と **Implementation Agent** が API 設計を行う際に参照します。

---

## REST API 設計原則

### URL 設計

```
# リソース名は複数形・名詞を使う
✅ /api/v1/documents
✅ /api/v1/search
❌ /api/v1/getDocuments
❌ /api/v1/doSearch

# バージョニング
/api/v1/search      ← 推奨
/api/v2/search      ← 将来のバージョン

# ネスト（2 階層まで）
/api/v1/documents/{id}/sections
```

### HTTP メソッドの使い方

| メソッド | 用途 | べき等 |
|---|---|---|
| GET | リソースの取得 | ✅ |
| POST | リソースの作成 | ❌ |
| PUT | リソースの全更新 | ✅ |
| PATCH | リソースの部分更新 | ❌ |
| DELETE | リソースの削除 | ✅ |

### HTTP ステータスコード

| コード | 意味 | 使用場面 |
|---|---|---|
| 200 | OK | 正常な GET / PATCH / DELETE |
| 201 | Created | 正常な POST（新規作成） |
| 400 | Bad Request | バリデーションエラー |
| 401 | Unauthorized | 認証なし・認証失敗 |
| 403 | Forbidden | 権限なし |
| 404 | Not Found | リソースが存在しない |
| 422 | Unprocessable Entity | Pydantic バリデーションエラー（FastAPI 自動） |
| 500 | Internal Server Error | サーバー内部エラー |

---

## FastAPI 実装パターン

### ルーター定義

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from src.models.response import SearchResponse
from src.core.auth import verify_api_key

router = APIRouter(prefix="/api/v1", tags=["search"])


@router.get("/search", response_model=SearchResponse)
async def search_documents(
    q: str = Query(..., min_length=1, description="Search query"),
    limit: int = Query(default=10, ge=1, le=50, description="Max results"),
    offset: int = Query(default=0, ge=0, description="Offset"),
    api_key: str = Depends(verify_api_key),
) -> SearchResponse:
    """Search documents by keyword.

    Args:
        q: Search query string.
        limit: Maximum number of results to return (1-50).
        offset: Number of results to skip.
        api_key: Validated API key from Authorization header.

    Returns:
        SearchResponse with results and AI-generated summary.

    Raises:
        HTTPException: 400 if query is invalid, 500 if search fails.
    """
    ...
```

### Pydantic v2 モデル定義

```python
from pydantic import BaseModel, Field, field_validator


class SearchResult(BaseModel):
    """Individual search result item."""

    id: str = Field(description="Document ID")
    title: str = Field(description="Document title")
    snippet: str = Field(description="Text snippet from the document")
    score: float = Field(ge=0.0, le=1.0, description="Relevance score (0-1)")
    url: str = Field(description="Document URL")


class SearchResponse(BaseModel):
    """Response model for the search endpoint."""

    query: str = Field(description="Original search query")
    total: int = Field(ge=0, description="Total number of matching documents")
    results: list[SearchResult] = Field(description="List of search results")
    summary: str = Field(description="AI-generated summary of results")
```

### 認証ミドルウェア

```python
from fastapi import Header, HTTPException
from src.core.config import settings


async def verify_api_key(authorization: str = Header(...)) -> str:
    """Verify Bearer API key from Authorization header.

    Args:
        authorization: Authorization header value (Bearer <key>).

    Returns:
        Validated API key string.

    Raises:
        HTTPException: 401 if key is missing or invalid.
    """
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Invalid or missing API key.")
    
    api_key = authorization.removeprefix("Bearer ")
    if api_key != settings.api_key:
        raise HTTPException(status_code=401, detail="Invalid or missing API key.")
    
    return api_key
```

### サービス層パターン

```python
from src.models.response import SearchResponse, SearchResult


class SearchService:
    """Service class for document search logic."""

    async def search(
        self, query: str, limit: int = 10, offset: int = 0
    ) -> SearchResponse:
        """Search documents and generate AI summary.

        Args:
            query: Search query string.
            limit: Maximum number of results.
            offset: Number of results to skip.

        Returns:
            SearchResponse with results and summary.
        """
        # 1. 検索実行
        results = await self._fetch_results(query, limit, offset)
        
        # 2. AI サマリ生成
        summary = await self._generate_summary(query, results)
        
        return SearchResponse(
            query=query,
            total=len(results),
            results=results,
            summary=summary,
        )
```

### 環境設定

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    """Application settings loaded from environment variables."""

    api_key: str
    anthropic_api_key: str
    search_index_url: str = "http://localhost:9200"
    max_results: int = 50

    class Config:
        env_file = ".env"


settings = Settings()
```

---

## テストパターン

### API エンドポイントのテスト

```python
import pytest
from httpx import AsyncClient
from src.main import app


@pytest.mark.asyncio
async def test_search_success():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get(
            "/api/v1/search?q=azure",
            headers={"Authorization": "Bearer test-key"},
        )
    assert response.status_code == 200
    data = response.json()
    assert "results" in data
    assert "summary" in data


@pytest.mark.asyncio
async def test_search_unauthorized():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/api/v1/search?q=azure")
    assert response.status_code == 401


@pytest.mark.asyncio
async def test_search_empty_query():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get(
            "/api/v1/search?q=",
            headers={"Authorization": "Bearer test-key"},
        )
    assert response.status_code == 422  # Pydantic validation error
```

---

## API ドキュメント

FastAPI は `/docs`（Swagger UI）と `/redoc` で自動的に API ドキュメントを生成します。

ドキュメントの品質を上げるために:

1. **タグ**を設定してエンドポイントをグループ化する
2. **description** を `Field()` に設定する
3. **response_model** を必ず指定する
4. エンドポイントに **docstring** を書く
