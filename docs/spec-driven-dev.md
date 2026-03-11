# Spec Driven Development

## Spec Driven Development とは

**Spec Driven Development（仕様駆動開発）** とは、コードを書く前に **仕様書（Specification）** を明確に定義し、その仕様書を唯一の真実（Single Source of Truth）として開発を進めるアプローチです。

AI 時代においては、仕様書が AI へのインプットになるため、従来以上に重要性が増しています。

---

## なぜ Spec Driven Development が重要か

### 従来の開発（AI なし）

```
曖昧な要件
  ↓
開発者が解釈してコードを書く
  ↓
レビューで認識の齟齬が発覚
  ↓
手戻り
```

### AI を活用した開発

```
明確な仕様書
  ↓
AI が仕様を解釈してコードを生成
  ↓
AI がコードをレビュー
  ↓
高品質な成果物
```

**仕様が曖昧だと AI の出力も曖昧になります。** 仕様の品質がそのまま成果物の品質に直結します。

---

## 良い仕様書の条件

### 1. 目的・背景が明確

```markdown
## 背景
社内に散在する文書を横断検索できるシステムが必要。
現状は各チームがバラバラのツールを使っており、情報の発見に時間がかかっている。
```

### 2. 機能要件が具体的

```markdown
## 機能要件
- キーワード検索でドキュメントを取得できる
- 検索結果を AI が要約して返す
- 認証済みユーザーのみアクセス可能
- 検索結果は最大 20 件を返す
```

### 3. API 設計が詳細

```markdown
## API 仕様

GET /api/v1/search

クエリパラメータ:
  q (string, 必須): 検索クエリ
  limit (int, オプション, デフォルト: 10): 最大件数
  offset (int, オプション, デフォルト: 0): オフセット

レスポンス:
  {
    "results": [
      {
        "id": "doc-001",
        "title": "Azure 導入ガイド",
        "snippet": "...",
        "score": 0.95
      }
    ],
    "summary": "Azure 関連のドキュメントが 5 件見つかりました...",
    "total": 5
  }
```

### 4. 非機能要件も含む

```markdown
## 非機能要件
- レスポンスタイム: 95 パーセンタイルで 2 秒以内
- 同時接続: 100 リクエスト/秒
- 可用性: 99.9% 以上
```

### 5. 制約・前提条件が明記されている

```markdown
## 制約
- 社内ネットワーク上で動作する
- 既存の認証基盤（OAuth 2.0）と連携する
- 検索対象は PDF / Word / Markdown ファイル
```

---

## Claude Code での実践方法

### Step 1: 仕様書を用意する

`spec/` ディレクトリに仕様書を Markdown で作成します。

```
spec/
  enterprise-search.md   ← 仕様書
```

### Step 2: Architect Agent に設計を依頼する

```
Read spec/enterprise-search.md

Design a system architecture for this specification.
Consider:
- Component breakdown
- API design
- Data flow
- Technology choices (use FastAPI as defined in CLAUDE.md)
```

### Step 3: Implementation Agent にコード生成を依頼する

```
Based on the architecture we designed, generate FastAPI implementation code.
Follow the coding standards defined in CLAUDE.md.
Include:
- All API endpoints
- Request/response models with Pydantic
- Service layer
- Unit tests
```

### Step 4: Review Agent にレビューを依頼する

```
Review the generated code for:
- Security vulnerabilities
- Performance issues
- Code quality and maintainability
- Adherence to CLAUDE.md standards
```

---

## Spec の品質を上げるテクニック

### ユーザーストーリーを起点にする

```
As a [ユーザー]
I want to [行動]
So that [目的]
```

例:
```
As an engineer
I want to search internal documents by keyword
So that I can quickly find relevant information
```

### 受け入れ条件（Acceptance Criteria）を定義する

```
Given: 認証済みユーザーが "azure" で検索したとき
When:  GET /api/v1/search?q=azure を呼び出す
Then:  Azure 関連のドキュメント一覧と AI サマリが返る
       ステータスコードは 200
       レスポンスタイムは 2 秒以内
```

### エラーケースも仕様に含める

```
- 認証されていない場合: 401 Unauthorized
- クエリが空の場合: 400 Bad Request
- 検索結果が 0 件の場合: 200 OK, results: []
```

---

## 実際の仕様書

本ハンズオンで使用する仕様書は [spec/enterprise-search.md](../spec/enterprise-search.md) を参照してください。
