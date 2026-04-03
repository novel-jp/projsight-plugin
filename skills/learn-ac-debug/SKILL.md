---
name: learn-ac-debug
description: 05-03 AI デバッグ（必修）。エラー発生時の情報の構造化と、AI への効果的なデバッグ依頼方法を学ぶ。
---

# /learn-ac-debug — AI と一緒にデバッグ（05-03、必修）

エラー発生時の情報の構造化と、AI への効果的なデバッグ依頼方法を学ぶ演習です。

**所要時間**: 約 60 分
**前提**: `/learn-ac-iterate`（05-02）完了済み
**スキル対応**: AI Collaboration（AI との協働）

---

## Step 1: 導入

```
「デバッグは AI が最も価値を発揮する場面の1つです。

ただし、『動きません』だけでは AI も助けられません。
エラー情報の構造化が鍵です。

同じバグでも、伝え方次第で AI の回答品質は劇的に変わります。
このスキルでは、AI に効果的にデバッグを依頼する方法を体験します。」
```

---

## Step 2: 学習環境の準備とバグ入りコードの配置

受講者の作業ディレクトリに `learn-workspace/` がなければ作成する:

```bash
mkdir -p learn-workspace && cd learn-workspace && git init
```

以下の 3 種類のバグを含むコードを `learn-workspace/buggy-code.ts` に配置する:

```typescript
// Bug 1: Off-by-one エラー
function getLastNItems<T>(arr: T[], n: number): T[] {
  return arr.slice(arr.length - n - 1);
}

// Bug 2: 型の不一致（文字列と数値の比較）
function findUser(users: { id: number; name: string }[], id: string) {
  return users.find((u) => u.id === id);
}

// Bug 3: 非同期処理のバグ（await 忘れ）
async function fetchAllData(urls: string[]) {
  const results = urls.map((url) => fetch(url).then((r) => r.json()));
  return results; // Promise[] が返る（解決済みデータではない）
}
```

---

## Step 3: 悪いデバッグ依頼 vs 良いデバッグ依頼

### 悪い依頼の例

Bug 1 に対して以下のように依頼する:

```
「このコード動きません、直してください。」
```

AI の回答を確認する。曖昧な依頼に対して AI がどう応答するかを観察する。

### 良い依頼の例

同じ Bug 1 に対して、構造化されたバグレポートで依頼する:

```
「## バグレポート

**エラー内容**: `getLastNItems([1,2,3,4,5], 2)` が `[3,4,5]` を返す（3要素）

**期待される動作**: `[4,5]`（末尾2要素）

**実際の動作**: `[3,4,5]`（末尾3要素）

**再現手順**:
1. `getLastNItems([1,2,3,4,5], 2)` を呼び出す
2. 戻り値を確認する

**試したこと**: `n` の値を変えても常に1つ多く返る

**関連コード**: `arr.slice(arr.length - n - 1)` の `-1` が怪しい」
```

AI の回答を比較し、**品質の差を体感**してもらう。

### 残りのバグで実践

Bug 2、Bug 3 についても受講者に構造化されたバグレポートを書いてもらい、AI に依頼する。

---

## Step 4: 問題の記録

見つけたバグを `upsert_issue` で ProjSight に記録する。

description には構造化された情報を含める:

- エラーメッセージまたは不正な動作の説明
- 再現手順
- 期待される動作と実際の動作
- 原因と修正内容

---

## Step 5: 振り返り

```
「AI へのデバッグ依頼テンプレートを覚えておきましょう:

1. エラーメッセージ全文 — コピペが基本、要約しない
2. 再現手順 — 最小限のステップで再現できる手順
3. 期待と現実の差 — 何が起きるべきで、何が起きたか
4. 試したこと — 重複調査を避けるため
5. 関連するコード箇所 — AI が探す手間を省く

ポイント:
- 『動きません』は最悪の依頼 — AI は超能力者ではない
- 構造化された情報は AI だけでなく人間のチームメイトにも有効
- ProjSight の Issue にも同じ構造で記録する習慣をつける

次の /learn-ac-code-review では、AI 生成コードのレビュー方法を学びます。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
