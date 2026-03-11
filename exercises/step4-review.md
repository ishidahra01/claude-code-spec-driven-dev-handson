# Step 4 — AI コードレビューを実施する

## このステップの目的

Review Agent を使って、生成した実装コードをレビューします。
AI がどのような観点でコードをレビューするか、そのプロセスを体験します。

---

## 所要時間

約 30〜40 分

---

## 前提条件

- [Step 3](step3-implementation.md) が完了していること
- `src/` ディレクトリに実装コードが存在すること
- テストが通過していること（`pytest tests/ -v` が全て PASSED）

---

## タスク 1: セキュリティレビューを依頼する

まずセキュリティの観点でレビューを行います。

### Claude Code プロンプト

```
You are the Review Agent defined in AGENTS.md.

Read the following files:
- CLAUDE.md
- AGENTS.md
- agents/review-agent.md
- skills/code-review.md
- src/ (all files)

Perform a SECURITY review of the implementation.

Check for:
1. Authentication bypass risks
2. Input validation gaps (are all user inputs validated?)
3. Sensitive data exposure (API keys in logs/responses?)
4. Error messages leaking internal details
5. Missing rate limiting

For each issue:
- File and line number
- Severity: HIGH / MEDIUM / LOW
- Description
- Suggested fix

Do NOT modify any files. Output review comments only.
```

### 期待される出力

Review Agent がコードを分析し、セキュリティ上の問題点を列挙します。
例えば:
- ✅ API キーが環境変数から読み込まれている（問題なし）
- ⚠️ エラーレスポンスに Python の例外メッセージが含まれている
- ⚠️ レート制限が未実装

---

## タスク 2: コード品質レビューを依頼する

次にコード品質の観点でレビューを行います。

### Claude Code プロンプト

```
You are the Review Agent.

Review src/ for code quality against the standards in CLAUDE.md.

Check:
1. Type hints — all function parameters and return values annotated?
2. Docstrings — all public functions have Google-style docstrings?
3. Error handling — using HTTPException correctly?
4. No hardcoded values — all config from environment?
5. Layering — router only calls service, service doesn't use FastAPI types?
6. Naming — following snake_case for variables/functions, PascalCase for classes?

Output a structured report with:
- Summary (total issues by severity)
- Detailed issue list
- Positive findings (things done well)
- Verdict: Approved / Approved with suggestions / Changes required
```

---

## タスク 3: パフォーマンスレビューを依頼する

### Claude Code プロンプト

```
You are the Review Agent.

Review src/ for performance issues.

Check:
1. All I/O operations use async/await?
2. No blocking calls inside async functions (requests, time.sleep, etc.)?
3. HTTP clients properly managed (not creating new client per request)?
4. Pagination implemented correctly?

Note: The current implementation uses mock data, so focus on the patterns
that would cause issues when real I/O is added.
```

---

## タスク 4: レビュー結果を修正する

レビューで指摘された問題を修正します。

### Claude Code プロンプト

```
Based on the code review, fix the following issues:

[Review Agent が指摘した HIGH/MEDIUM の問題を列挙する]

For each fix:
1. Show the before/after code
2. Explain why the fix addresses the issue

Follow CLAUDE.md coding standards when making fixes.
```

---

## タスク 5: 修正後の再レビュー

修正が正しく行われたか確認します。

### Claude Code プロンプト

```
You are the Review Agent.

The following issues were fixed in the previous review:
[修正した問題の一覧]

Verify that:
1. Each fix correctly addresses the reported issue
2. The fix didn't introduce new problems
3. Code quality standards from CLAUDE.md are still maintained

Output: Verification result for each fix (Fixed / Partially Fixed / Not Fixed)
```

---

## タスク 6: テストレビューを依頼する（オプション）

### Claude Code プロンプト

```
You are the Review Agent.

Review tests/ for test quality.

Check:
1. Coverage of normal cases
2. Coverage of error cases (400, 401, 422, 500)
3. Edge cases covered (empty query, max limit)
4. Tests are independent (no shared state)
5. External dependencies properly mocked
6. Test names are descriptive

Suggest any missing test cases.
```

---

## 完成した AI 開発フローの振り返り

このハンズオンで体験した完全なフロー:

```
spec/enterprise-search.md
        │
        ▼ Step 1
   仕様を理解する
        │
        ▼ Step 2
  アーキテクチャ設計
  (Architect Agent)
        │
        ▼ Step 3
    コード生成
(Implementation Agent)
        │
        ▼ Step 4
    AI レビュー
  (Review Agent)
        │
        ▼
      PR / Merge
```

---

## 学習ポイント

このステップで学んだこと:

- ✅ Review Agent の役割と使い方を理解した
- ✅ AI によるセキュリティ・品質・パフォーマンスレビューを体験した
- ✅ レビュー結果に基づいてコードを改善するサイクルを体験した
- ✅ `CLAUDE.md` が一貫したレビュー基準として機能することを確認した

---

## さらに学ぶには

- [docs/spec-driven-dev.md](../docs/spec-driven-dev.md) — Spec Driven Development の詳細
- [docs/claude-code-basics.md](../docs/claude-code-basics.md) — Claude Code の応用機能
- [skills/api-design.md](../skills/api-design.md) — API 設計の追加パターン
- [skills/code-review.md](../skills/code-review.md) — レビュー観点の詳細

---

## ハンズオン完了！ 🎉

おめでとうございます！このハンズオンを完了し、以下を体験しました:

1. **Spec Driven Development** — 仕様書を起点にした AI 開発フロー
2. **Context Files** — `CLAUDE.md` と `AGENTS.md` による AI 制御
3. **Multi-Agent** — Architect・Implementer・Reviewer の役割分担
4. **Progressive Workflow** — 段階的な開発フロー
5. **Subagents / Skills** — 再利用可能なスキルの活用
