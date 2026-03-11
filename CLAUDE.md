# CLAUDE.md

このファイルは Claude Code がプロジェクトを理解し、適切なコードを生成するための **コンテキストファイル** です。

---

## プロジェクト概要

**Enterprise Knowledge Search API** を FastAPI で実装するハンズオンプロジェクトです。

---

## 技術スタック

| カテゴリ | 技術 |
|---|---|
| 言語 | Python 3.11+ |
| フレームワーク | FastAPI |
| バリデーション | Pydantic v2 |
| 非同期 HTTP | httpx |
| テスト | pytest, pytest-asyncio, httpx |
| Lint / Format | ruff, mypy |
| パッケージ管理 | uv (推奨) または pip |

---

## ディレクトリ構成（実装後）

```
src/
  main.py          # FastAPI アプリエントリポイント
  routers/
    search.py      # /search エンドポイント
  services/
    search_service.py    # 検索ロジック
    summary_service.py   # AI サマリ生成ロジック
  models/
    request.py     # リクエストモデル
    response.py    # レスポンスモデル
  core/
    config.py      # 環境設定
    auth.py        # 認証ミドルウェア
tests/
  test_search.py
  test_summary.py
```

---

## コーディング規約

1. **型ヒントを必ず付ける** — すべての関数・メソッドに型アノテーションを記述する
2. **Pydantic モデルを使う** — リクエスト・レスポンスは Pydantic BaseModel で定義する
3. **非同期関数を優先する** — I/O 処理は `async def` で実装する
4. **エラーハンドリング** — `HTTPException` を適切に使い、分かりやすいエラーメッセージを返す
5. **環境変数** — シークレットや設定値はハードコードせず、`core/config.py` で管理する
6. **ドキュメント文字列** — 公開関数には docstring を書く（Google スタイル）
7. **テスト** — 新機能には必ず対応するテストを追加する

### 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| 変数 / 関数 | snake_case | `search_results` |
| クラス | PascalCase | `SearchRequest` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RESULTS` |
| ファイル | snake_case | `search_service.py` |

---

## アーキテクチャガイドライン

### レイヤー設計

```
Router (HTTP 層)
  └─ Service (ビジネスロジック層)
       └─ Repository / Client (データアクセス層)
```

- **Router**: HTTP リクエストの受け取りとレスポンス返却のみ担当
- **Service**: 検索・サマリ生成などのビジネスロジックを担当
- **Repository / Client**: 外部 API 呼び出しや DB アクセスを担当

### API 設計原則

- RESTful 設計に従う
- バージョニング: `/api/v1/` プレフィックスを使用
- レスポンスは常に JSON
- HTTP ステータスコードを適切に使用（200, 400, 401, 404, 500）
- ページネーションには `limit` / `offset` パラメータを使用

### セキュリティ

- API キーは `Authorization: Bearer <token>` ヘッダーで受け取る
- 入力値は Pydantic で自動バリデーション
- SQL インジェクション・XSS を防ぐためユーザー入力をエスケープする
- レート制限を実装する

---

## テスト方針

```bash
# テスト実行
pytest tests/ -v

# カバレッジ付き
pytest tests/ -v --cov=src --cov-report=term-missing
```

- Unit テスト: サービス層のロジックをモックを使ってテスト
- Integration テスト: `httpx.AsyncClient` を使って API エンドポイントをテスト
- カバレッジ目標: 80% 以上

---

## 開発コマンド

```bash
# 依存関係インストール
uv pip install -r requirements.txt

# 開発サーバー起動
uvicorn src.main:app --reload

# Lint
ruff check src/

# 型チェック
mypy src/
```
