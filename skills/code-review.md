# Code Review Skill

## 概要

このスキルはコードレビューの観点・チェックリスト・レポートフォーマットを定義します。
**Review Agent** がコードレビューを行う際に参照します。

---

## レビュー観点

コードレビューは以下の 4 つの観点で行います:

```
1. Security    — セキュリティ
2. Performance — パフォーマンス
3. Quality     — コード品質
4. Tests       — テスト
```

---

## 1. セキュリティチェックリスト

### 認証・認可

- [ ] すべてのエンドポイントで認証が行われているか
- [ ] 公開エンドポイント（`/health` 等）は意図的に認証を外しているか（コメントで明示されているか）
- [ ] API キーがログに出力されていないか
- [ ] API キーがコードにハードコードされていないか（環境変数を使っているか）

### 入力バリデーション

- [ ] ユーザー入力はすべて Pydantic でバリデーションされているか
- [ ] クエリパラメータに最大値・最小値の制約があるか（例: `limit` は最大 50）
- [ ] 文字列の最大長が制限されているか
- [ ] ファイルアップロードがある場合、ファイルタイプ・サイズが制限されているか

### エラーハンドリング

- [ ] エラーレスポンスに内部情報（スタックトレース・ファイルパス・DB クエリ）が含まれていないか
- [ ] 例外が適切にキャッチされているか（`except Exception` で握りつぶしていないか）
- [ ] 認証エラーのメッセージが情報を漏洩していないか（`"Invalid API key"` ではなく `"Invalid or missing API key"` のように）

### 依存関係

- [ ] 外部ライブラリに既知の脆弱性がないか（`pip audit` で確認）
- [ ] 固定バージョン（pinned）で依存関係が指定されているか

---

## 2. パフォーマンスチェックリスト

### 非同期処理

- [ ] I/O を伴う処理は `async def` で実装されているか
- [ ] `async def` 内で同期的なブロッキング処理（`time.sleep`, `requests.get` 等）を呼んでいないか
- [ ] 外部 API 呼び出しは `httpx.AsyncClient` を使っているか

### データ処理

- [ ] 不必要にすべてのデータをメモリに読み込んでいないか
- [ ] ページネーションが実装されているか
- [ ] 同じデータを繰り返し取得していないか（N+1 問題）

### リソース管理

- [ ] DB 接続・HTTP クライアントは適切にクローズされているか（コンテキストマネージャを使っているか）
- [ ] 無限ループや終了条件のないループがないか

---

## 3. コード品質チェックリスト

### 型安全性

- [ ] すべての関数の引数と戻り値に型ヒントがあるか
- [ ] `Any` 型を使っている場合、コメントで理由が説明されているか
- [ ] `mypy` で型エラーがないか

### ドキュメント

- [ ] 公開関数・クラスに docstring があるか（Google スタイル）
- [ ] 複雑なロジックにコメントがあるか
- [ ] TODO コメントは Issue 番号付きか

### 設計・構造

- [ ] CLAUDE.md のレイヤー設計（Router → Service → Repository）に従っているか
- [ ] 関数が単一責任の原則に従っているか（1 関数 1 責務）
- [ ] マジックナンバー・マジック文字列が定数として定義されているか
- [ ] 重複コードがないか（DRY 原則）

### 命名

- [ ] 変数・関数名が意味を明確に表しているか
- [ ] CLAUDE.md の命名規則（snake_case 等）に従っているか

### Lint

- [ ] `ruff check` でエラーがないか
- [ ] インポートが整理されているか（`isort`）

---

## 4. テストチェックリスト

### カバレッジ

- [ ] 正常系のテストがあるか
- [ ] 主要な異常系のテストがあるか（400, 401, 404, 500）
- [ ] エッジケース（空入力、最大値、最小値）のテストがあるか
- [ ] カバレッジが 80% 以上か

### テスト品質

- [ ] テスト名が何をテストしているか明確か（`test_search_returns_results_for_valid_query`）
- [ ] テストが独立しているか（他のテストの状態に依存していないか）
- [ ] 外部依存（API 呼び出し、DB）は適切にモックされているか
- [ ] `pytest.mark.asyncio` が非同期テストに付いているか

---

## Severity 分類

| Severity | 基準 | 対応方針 |
|---|---|---|
| **HIGH** | セキュリティ脆弱性・本番で重大な障害を引き起こすバグ | マージ前に必ず修正 |
| **MEDIUM** | パフォーマンス問題・保守性に影響する設計問題 | 早期に修正することを推奨 |
| **LOW** | コーディング規約違反・軽微な改善提案 | 次のイテレーションで修正可 |

---

## レビューレポートフォーマット

```markdown
# Code Review Report

**Reviewer**: Review Agent  
**Date**: YYYY-MM-DD  
**Target**: src/

## Summary

| Severity | Count |
|---|---|
| HIGH | 0 |
| MEDIUM | 2 |
| LOW | 3 |

## Issues

### [HIGH] <issue title>
**File**: path/to/file.py:line_number  
**Description**: <説明>  
**Suggested Fix**:
```python
# 修正後のコード例
```

---

### [MEDIUM] <issue title>
...

---

## Positive Findings

- <良かった点>
- <良かった点>

## Verdict

[ ] ✅ Approved — No issues or only LOW severity issues
[ ] ⚠️ Approved with suggestions — MEDIUM issues found
[ ] ❌ Changes required — HIGH issues found
```

---

## よくある問題と修正パターン

### ハードコードされた API キー

```python
# ❌ 悪い例
API_KEY = "sk-1234567890abcdef"

# ✅ 良い例
from src.core.config import settings
API_KEY = settings.api_key
```

### 同期処理を非同期関数内で呼ぶ

```python
# ❌ 悪い例（ブロッキング）
async def fetch_data():
    import requests
    response = requests.get("https://api.example.com/data")
    return response.json()

# ✅ 良い例（非ブロッキング）
async def fetch_data():
    import httpx
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
    return response.json()
```

### 内部情報を含むエラーメッセージ

```python
# ❌ 悪い例
except Exception as e:
    raise HTTPException(status_code=500, detail=str(e))

# ✅ 良い例
except Exception:
    logger.exception("Unexpected error during search")
    raise HTTPException(
        status_code=500,
        detail="Internal server error. Please try again later."
    )
```
