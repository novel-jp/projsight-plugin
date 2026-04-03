---
name: learn-ac-security
description: 05-05 セキュリティ。AI 利用時の機密情報の取り扱いと、生成コードの脆弱性パターンを学ぶ。
---

# /learn-ac-security — セキュリティ意識（05-05）

AI 利用時の機密情報の取り扱いと、生成コードの脆弱性パターンを学ぶ演習です。

**所要時間**: 約 60 分
**前提**: `/learn-ac-code-review`（05-04）完了済み
**スキル対応**: AI Collaboration（AI との協働）

---

## Step 1: 導入

```
「AI にコードを書かせる時、2つのセキュリティリスクがあります:

1. 機密情報を AI に渡してしまう
   — API キー、本番DB接続情報、個人情報を含むログ

2. AI が脆弱なコードを生成する
   — SQL インジェクション、XSS、ハードコードされた秘密鍵

どちらも『うっかり』で発生します。
このスキルでは、両方のリスクに対する防御パターンを学びます。」
```

---

## Step 2: 学習環境の準備

受講者の作業ディレクトリに `learn-workspace/` がなければ作成する:

```bash
mkdir -p learn-workspace && cd learn-workspace && git init
```

---

## Step 3: 機密情報の取り扱い演習

### やってはいけないこと

以下のアンチパターンを受講者に提示し、なぜ危険かを議論する:

| アンチパターン                    | リスク                        |
| --------------------------------- | ----------------------------- |
| API キーを直接プロンプトに含める  | AI サービスのログに残る可能性 |
| 本番 DB の接続情報を渡す          | 意図しない接続・データ漏洩    |
| 個人情報を含むログを貼り付ける    | プライバシー違反、法的リスク  |
| `.env` ファイルをそのまま共有する | 全シークレットが一括漏洩      |

### 安全な方法

受講者に以下の安全なパターンでコードを書いてもらう:

1. `.env.example` ファイルを作成する（値はプレースホルダー）

```
# learn-workspace/.env.example
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
API_KEY=your-api-key-here
JWT_SECRET=your-jwt-secret-here
```

2. 環境変数を使った安全なコードを `learn-workspace/config.ts` に実装する:
   - `process.env` から読み取り
   - 未設定時のエラーハンドリング
   - デフォルト値の安全な設定

3. `.gitignore` に `.env` を追加する

受講者に「なぜ `.env.example` パターンが有効か？」を質問し、議論する。

---

## Step 4: 脆弱性パターンの検出演習

以下の 4 つの脆弱性パターンを1つずつ提示し、問題と修正方法を確認する。

### パターン 1: SQL インジェクション

```typescript
// 脆弱なコード
function getUser(name: string) {
  return db.query(`SELECT * FROM users WHERE name = '${name}'`);
}
```

受講者に「このコードの問題は？」と問いかけ、修正方法（パラメータ化クエリ）を議論する。

### パターン 2: XSS（クロスサイトスクリプティング）

```typescript
// 脆弱なコード
function renderComment(comment: string) {
  return `<div class="comment">${comment}</div>`;
}
```

受講者に「悪意のある入力が来たらどうなるか？」と問いかけ、エスケープ処理を議論する。

### パターン 3: ハードコードされた秘密鍵

```typescript
// 脆弱なコード
const JWT_SECRET = 'super-secret-key-12345';
const API_KEY = 'sk-proj-abc123def456';
```

Step 3 で学んだ環境変数パターンとの対比を確認する。

### パターン 4: パストラバーサル

```typescript
// 脆弱なコード
function readUserFile(filename: string) {
  return fs.readFileSync(`./uploads/${filename}`);
}
// 攻撃: filename = "../../etc/passwd"
```

受講者に「どう防御するか？」を考えてもらい、パスの正規化とバリデーションを議論する。

### 脆弱性の記録

見つけた脆弱性を `upsert_issue(category: 'bug', severity)` で ProjSight に記録する。
severity は脆弱性の深刻度に応じて設定する。

---

## Step 5: 振り返り

```
「セキュリティは後付けではなく、最初から意識するものです。

AI 利用時のセキュリティまとめ:

【機密情報の保護】
- API キー・接続情報は絶対にプロンプトに含めない
- .env.example + 環境変数パターンを標準にする
- 個人情報はマスキングしてから AI に渡す

【生成コードの検証】
- OWASP Top 10 のチェックを習慣にする
- 特に入力値の検証（SQL, XSS, パストラバーサル）
- 秘密鍵のハードコードは git-secrets 等のツールで自動検出

【AI Collaboration セクション総まとめ】
05-01: AI の出力を批判的に評価する
05-02: イテレーティブに品質を高める
05-03: 構造化された情報でデバッグを依頼する
05-04: 体系的なレビューチェックリストを使う
05-05: セキュリティを最初から意識する

これで AI Collaboration セクションは完了です。
AI は強力なツールですが、最終的な品質の責任は人間にあります。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
