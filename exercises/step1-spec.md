# Step 1 — 仕様を書く・読む

## このステップの目的

`spec/enterprise-search.md` を読み込み、今回作成する **Enterprise Knowledge Search API** の要件を把握します。
また、仕様書を起点にした **Spec Driven Development** の考え方を理解します。

---

## 所要時間

約 20〜30 分

---

## 前提条件

- Claude Code がインストール・認証済みであること
- このリポジトリのルートディレクトリで作業していること

---

## タスク 1: 仕様書を読む

まず、Claude Code に仕様書を読ませて内容を要約してもらいましょう。

### Claude Code プロンプト

```
Read spec/enterprise-search.md

Please summarize:
1. What problem does this API solve?
2. What are the main functional requirements?
3. What does the API response look like?
4. What are the non-functional requirements (performance, security)?
5. What are the acceptance criteria?
```

### 期待される出力

Claude Code が仕様書の内容を日本語で分かりやすく要約してくれるはずです。
要約が仕様書の内容を正確に反映しているか確認してください。

---

## タスク 2: 仕様の不明点を明確化する

実装を始める前に、仕様の曖昧な点を洗い出します。

### Claude Code プロンプト

```
Read spec/enterprise-search.md

Identify any ambiguities or missing details in the specification.
For each ambiguity, suggest how it could be clarified.

Focus on:
- Edge cases not covered
- Error scenarios that need more detail
- Performance requirements that need clarification
```

### 気づくべきポイント

良い仕様書でも、実装時には曖昧な点が出てきます。例えば:

- 検索対象のデータはどこに保存されているか？
- AI サマリは何語で生成するか？（仕様書には「日本語」と書いてある）
- 認証トークンの有効期限は？
- 検索キャッシュは必要か？

---

## タスク 3: 仕様書を拡張してみる（オプション）

自分でシナリオを追加してみましょう。

### やること

`spec/enterprise-search.md` を参考に、以下の機能の仕様を **追記** してみてください:

```
新機能案:
- ドキュメントのカテゴリでフィルタリングする機能
  例: GET /api/v1/search?q=azure&category=design
```

考えるべき点:
1. カテゴリのデータ型（文字列？列挙型？）
2. 存在しないカテゴリを指定した場合の挙動
3. レスポンスにカテゴリ情報を含めるか

---

## 学習ポイント

このステップで学んだこと:

- ✅ 仕様書がいかに重要かを理解した
- ✅ 仕様書から AI が情報を読み取る方法を体験した
- ✅ 良い仕様書の条件（具体的・明確・エラーケースを含む）を理解した

詳細は [docs/spec-driven-dev.md](../docs/spec-driven-dev.md) を参照してください。

---

## 次のステップ

[exercises/step2-architecture.md](step2-architecture.md) に進んで、アーキテクチャ設計を行います。
