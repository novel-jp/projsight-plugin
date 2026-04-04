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

受講者の作業ディレクトリに `learn-workspace/` がなければ作成する（既に存在していれば `mkdir -p` で問題ない）:

```bash
mkdir -p learn-workspace && cd learn-workspace && git init
```

### タスクの作成と開始

1. `list_deliverables(projectId)` で成果物一覧を確認し、学習に紐づく成果物を特定する
2. `upsert_task` でタスクを作成する:
   - **title**: `05-05 セキュリティ演習`
   - **deliverableId**: 学習成果物の ID
   - **description**: 以下の内容で作成

```markdown
## 背景

AI Collaboration セクションの最終スキル。AI 利用時のセキュリティリスクを実践的に学ぶ。

## 対応内容

- 機密情報の安全な取り扱いパターンを実装する
- AI が生成しやすい脆弱性パターンを検出・修正する

## 完了条件

- [ ] .env.example と config.ts を learn-workspace/ に作成した
- [ ] SQL インジェクションの修正版コードを実装した
- [ ] パストラバーサルの修正版コードを実装した
- [ ] 振り返りを実施した
```

3. `start_work(taskId)` で作業を開始する

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

2. 環境変数を使った安全なコードを `learn-workspace/config.ts` に実装する。以下のスケルトンを提示し、受講者に完成させる（まず自分で考え、行き詰まったら AI に相談してよい）:

```typescript
// learn-workspace/config.ts

/** 必須の環境変数を取得する。未設定なら起動時にエラーにする */
function getRequiredEnv(key: string): string {
  const value = process.env[key];
  // TODO: 未設定時にどうするか実装する
  return value;
}

export const config = {
  databaseUrl: getRequiredEnv('DATABASE_URL'),
  apiKey: getRequiredEnv('API_KEY'),
  jwtSecret: getRequiredEnv('JWT_SECRET'),
  // TODO: 省略可能な環境変数にはデフォルト値を設定する
  port: parseInt(process.env.PORT ?? '3000', 10),
};
```

**完了条件**: `getRequiredEnv` が未設定時に明確なエラーメッセージを throw すること。

3. `.gitignore` に `.env` を追加する

受講者に「なぜ `.env.example` パターンが有効か？」を質問し、議論する。

### 発展: AI ツール固有の防御

余力がある受講者には、以下の AI 特有の防御策も紹介する:

- **`.claudeignore`**: Claude Code に読ませたくないファイル（`.env`、`credentials/`、秘密鍵）を指定できる。`.gitignore` と同じ構文
- **プロンプト衛生**: 「このコードを修正して」と依頼する前に、ハードコードされたシークレットをプレースホルダーに置換してから渡す習慣
- **コンテキスト最小化**: AI にプロジェクト全体を渡すのではなく、必要なファイルだけを渡すことで意図しない情報漏洩を防ぐ

---

## Step 4: 脆弱性パターンの検出と修正演習

以下の 4 つの脆弱性パターンを1つずつ提示する。パターン 1・4 は修正版コードを `learn-workspace/` に実装する。パターン 2・3 は問題点と修正方法を議論する。

### パターン 1: SQL インジェクション 【ハンズオン】

```typescript
// 脆弱なコード
function getUser(name: string) {
  return db.query(`SELECT * FROM users WHERE name = '${name}'`);
}
```

**AI がこのコードを生成しやすい理由**: テンプレートリテラルによる文字列組み立ては TypeScript の基本機能であり、AI は「動くコード」を最短で生成しようとするため、セキュリティ考慮のないシンプルな実装を出力しがち。

受講者に「このコードの問題は？」と問いかけた後、**修正版コードを `learn-workspace/secure-query.ts` に実装させる**:

- パラメータ化クエリ（プレースホルダー `$1` や `?`）を使った安全な実装に書き直す
- テスト用の攻撃入力（`' OR '1'='1`）で元のコードと修正版の違いを確認する

### パターン 2: XSS（クロスサイトスクリプティング）

```typescript
// 脆弱なコード
function renderComment(comment: string) {
  return `<div class="comment">${comment}</div>`;
}
```

**AI がこのコードを生成しやすい理由**: AI は HTML テンプレートを要求されると、フレームワーク（React 等）の自動エスケープに頼る実装を想定し、素の文字列結合でもエスケープを省略した「見た目が正しい」コードを返しやすい。

受講者に「`comment` に `<script>alert('XSS')</script>` が入ったらどうなるか？」と問いかけ、エスケープ処理の必要性を議論する。

### パターン 3: SSRF（Server-Side Request Forgery）

```typescript
// 脆弱なコード
async function fetchExternalData(url: string) {
  const response = await fetch(url);
  return response.json();
}
// 攻撃: url = "http://169.254.169.254/latest/meta-data/"
```

**AI がこのコードを生成しやすい理由**: AI は「URL を受け取って fetch する」というシンプルな要件に対して、URL の検証なしにそのまま使うコードを返す。特に AWS メタデータエンドポイントへのアクセスは、AI の学習データに含まれにくい防御パターン。

受講者に「内部ネットワークへのアクセスを防ぐにはどうするか？」を考えてもらい、URL のホワイトリスト検証やプロトコル制限を議論する。

### パターン 4: パストラバーサル 【ハンズオン】

```typescript
// 脆弱なコード
function readUserFile(filename: string) {
  return fs.readFileSync(`./uploads/${filename}`);
}
// 攻撃: filename = "../../etc/passwd"
```

**AI がこのコードを生成しやすい理由**: ファイル読み込みのコード生成時、AI はパスの結合を単純な文字列連結で処理し、`..` による親ディレクトリ遡上を考慮しないことが多い。

受講者に「どう防御するか？」を考えてもらった後、**修正版コードを `learn-workspace/safe-file-read.ts` に実装させる**:

- `path.resolve()` でパスを正規化する
- 正規化後のパスが許可ディレクトリ配下にあるかチェックする
- 危険な入力（`../../etc/passwd`）でテストして防御を確認する

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
- AI が生成したコードほど「動くけど安全か？」を疑う

【AI Collaboration セクション総まとめ】
05-01: AI の出力を批判的に評価する
05-02: イテレーティブに品質を高める
05-03: 構造化された情報でデバッグを依頼する
05-04: 体系的なレビューチェックリストを使う
05-05: セキュリティを最初から意識する

これで AI Collaboration セクションは完了です。
AI は強力なツールですが、最終的な品質の責任は人間にあります。」
```

### 発展課題（任意）

- **振り返り質問**: 「今回学んだ4つのパターンで、一番意外だったものは？ なぜ？」
- **チーム展開**: 学んだことをもとに「チーム向けセキュリティ3行ガイドライン」を書いてみよう（例: 「1. AI にシークレットを渡すな　2. 生成コードの入力値検証を確認せよ　3. `.env` は `.gitignore` と `.claudeignore` に入れよ」）

---

## Step 6: 完了

受講者のタスクを `complete_work(taskId)` で完了にする。
