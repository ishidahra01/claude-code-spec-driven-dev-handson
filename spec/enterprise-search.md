# Enterprise Knowledge Search API — 仕様書

## 背景・目的

社内に散在する技術ドキュメント・設計書・議事録などを横断的に検索できる API を構築する。
エンジニアが必要な情報を素早く発見できるようにし、情報探索にかかる時間を削減することを目的とする。

---

## ユーザーストーリー

```
As an engineer,
I want to search internal documents using keywords,
So that I can quickly find relevant technical information.
```

```
As an engineer,
I want to receive an AI-generated summary of search results,
So that I can understand the key points without reading all documents.
```

---

## 機能要件

### 1. キーワード検索

- キーワードを入力すると、関連するドキュメントの一覧を返す
- スコアの高い順に並び替えて返す
- 検索結果には タイトル・スニペット・スコア・ドキュメント ID を含む

### 2. AI サマリ生成

- 検索結果に基づき、AI が日本語でサマリを生成して返す
- サマリは検索結果の主要なポイントをまとめたものとする
- サマリは 200 文字以内とする（Unicode 文字数。日本語・英字ともに 1 文字としてカウント）

### 3. 認証

- API キーによる認証をサポートする
- 認証されていないリクエストは拒否する

### 4. ページネーション

- `limit` パラメータで返却件数を制御できる（デフォルト: 10、最大: 50）
- `offset` パラメータでオフセットを指定できる

---

## 非機能要件

| 項目 | 要件 |
|---|---|
| レスポンスタイム | 95 パーセンタイルで 3 秒以内 |
| スループット | 50 リクエスト/秒 |
| 可用性 | 99.5% 以上 |
| セキュリティ | HTTPS 必須、API キー認証 |

---

## API 設計

### ベース URL

```
https://api.example.com/api/v1
```

---

### エンドポイント一覧

| メソッド | パス | 説明 |
|---|---|---|
| GET | /search | ドキュメントを検索する |
| GET | /health | ヘルスチェック |

---

### GET /search

ドキュメントをキーワード検索し、結果と AI サマリを返す。

#### リクエスト

**ヘッダー**

```
Authorization: Bearer <api_key>
```

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| q | string | ✅ | — | 検索クエリ |
| limit | integer | ❌ | 10 | 返却件数（最大 50） |
| offset | integer | ❌ | 0 | オフセット |

**リクエスト例**

```
GET /api/v1/search?q=azure&limit=5
Authorization: Bearer sk-xxxxxxxxxxxx
```

#### レスポンス

**200 OK**

```json
{
  "query": "azure",
  "total": 42,
  "results": [
    {
      "id": "doc-001",
      "title": "Azure 導入ガイド 2024",
      "snippet": "Azure のサービス概要と導入手順について説明します。まず Azure Portal へのアクセス方法から...",
      "score": 0.95,
      "url": "https://docs.internal/azure-guide"
    },
    {
      "id": "doc-002",
      "title": "Azure Kubernetes Service (AKS) 設計書",
      "snippet": "AKS クラスターの設計方針と運用手順について記載します...",
      "score": 0.87,
      "url": "https://docs.internal/aks-design"
    }
  ],
  "summary": "Azure に関するドキュメントが 42 件見つかりました。Azure 導入ガイドでは基本的なセットアップ手順が、AKS 設計書ではコンテナ環境の構築方法が解説されています。"
}
```

**400 Bad Request** — クエリが空、またはパラメータが不正

```json
{
  "detail": "Query parameter 'q' is required and must not be empty."
}
```

**401 Unauthorized** — 認証されていない

```json
{
  "detail": "Invalid or missing API key."
}
```

**500 Internal Server Error** — サーバーエラー

```json
{
  "detail": "Internal server error. Please try again later."
}
```

---

### GET /health

サービスの稼働状況を確認する。

#### レスポンス

**200 OK**

```json
{
  "status": "healthy",
  "version": "1.0.0"
}
```

---

## データモデル

### SearchResult

```python
class SearchResult(BaseModel):
    id: str           # ドキュメント ID
    title: str        # ドキュメントタイトル
    snippet: str      # スニペット（検索結果の抜粋）
    score: float      # 関連度スコア (0.0 〜 1.0)
    url: str          # ドキュメント URL
```

### SearchResponse

```python
class SearchResponse(BaseModel):
    query: str                  # 検索クエリ
    total: int                  # 総件数
    results: list[SearchResult] # 検索結果一覧
    summary: str                # AI 生成サマリ
```

---

## アーキテクチャ方針

### コンポーネント

```
Client
  │
  ▼
FastAPI Application
  ├── Auth Middleware (API キー認証)
  ├── Search Router (/api/v1/search)
  │     ├── Search Service (検索ロジック)
  │     └── Summary Service (AI サマリ生成)
  └── Health Router (/health)
```

### 技術選定

| コンポーネント | 技術 | 理由 |
|---|---|---|
| API フレームワーク | FastAPI | 高速・型安全・自動ドキュメント生成 |
| データバリデーション | Pydantic v2 | FastAPI との統合・高速バリデーション |
| AI サマリ | Anthropic Claude API | 高品質な日本語サマリ生成 |
| テスト | pytest + httpx | FastAPI の推奨テストツール |

---

## セキュリティ要件

1. **認証**: API キーを `Authorization: Bearer` ヘッダーで検証する
2. **入力バリデーション**: Pydantic でクエリパラメータをバリデーションする
3. **レート制限**: IP あたり 100 リクエスト/分
4. **HTTPS**: 本番環境では HTTPS 必須
5. **エラーメッセージ**: 内部情報（スタックトレース等）を外部に漏らさない

---

## 受け入れ条件

### 正常系

```
Given: 有効な API キーを持つユーザーが "azure" で検索する
When:  GET /api/v1/search?q=azure を呼び出す
Then:  ステータスコード 200 が返る
       results に Azure 関連のドキュメントが含まれる
       summary に日本語のサマリが含まれる
       レスポンスタイムが 3 秒以内
```

### 異常系

```
Given: 認証されていないユーザーが検索する
When:  Authorization ヘッダーなしで GET /api/v1/search?q=azure を呼び出す
Then:  ステータスコード 401 が返る

Given: クエリが空のリクエスト
When:  GET /api/v1/search?q= を呼び出す
Then:  ステータスコード 400 が返る

Given: limit が 50 を超えるリクエスト
When:  GET /api/v1/search?q=azure&limit=100 を呼び出す
Then:  ステータスコード 400 が返る
```
