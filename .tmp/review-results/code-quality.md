# コード品質レビュー結果

## 判定

- **NEEDS_FIX**

## サマリー

- 指摘事項数: 8件
- 重要度内訳: Critical: 2, High: 3, Medium: 3, Low: 0
- 全指摘がブロッキング（新規追加ファイルのため、全問題が「今回の変更で導入された問題」）

---

## 指摘事項

---

### [test-code/example.js:4] オフバイワンエラーによる実行時例外

**finding_id**: `CQ-001`
**状態**: `new`
**重要度**: Critical
**カテゴリ**: ロジックバグ / エラーハンドリング

ループ条件が `i <= items.length` となっており、`i` が `items.length` に達したとき `items[i]` は `undefined` になる。`undefined.price` にアクセスすると `TypeError: Cannot read properties of undefined` が実行時に発生する。

**対象コード**:
```javascript
for (var i = 0; i <= items.length; i++) {  // i === items.length のとき items[i] は undefined
  total += items[i].price;  // TypeError が発生する
}
```

**推奨対応**:
`<=` を `<` に修正する。

```javascript
for (let i = 0; i < items.length; i++) {
  total += items[i].price;
}
```

---

### [test-code/example.js:11] 機密情報のハードコーディング

**finding_id**: `CQ-002`
**状態**: `new`
**重要度**: Critical
**カテゴリ**: セキュリティ

`API_KEY` にシークレットキー（`sk-1234567890abcdef`）がソースコードに直接埋め込まれている。コーディングスタイル規則「No hardcoded values」違反。ソースコード管理システムに機密情報が残存し、漏洩リスクが生じる。

**対象コード**:
```javascript
const API_KEY = "sk-1234567890abcdef";  // セキュリティ問題
```

**推奨対応**:
環境変数経由で取得するよう変更する。

```javascript
const API_KEY = process.env.API_KEY;
if (!API_KEY) {
  throw new Error('API_KEY environment variable is required');
}
```

---

### [test-code/example.js:2] 新規振る舞いに対するテストが存在しない

**finding_id**: `CQ-003`
**状態**: `new`
**重要度**: High
**カテゴリ**: テスト欠如

`calculateTotal` 関数が新規追加されているが、対応するテストファイルが存在しない。ポリシー「テストがない新しい振る舞い」に該当し、無条件 REJECT。

**推奨対応**:
`test-code/example.test.js`（またはプロジェクトの規約に従ったパス）に以下を含むテストを追加する。

```javascript
// test-code/example.test.js
const { calculateTotal } = require('./example');

describe('calculateTotal', () => {
  test('正常ケース: 合計金額を返す', () => {
    expect(calculateTotal([{ price: 100 }, { price: 200 }])).toBe(300);
  });
  test('空配列: 0を返す', () => {
    expect(calculateTotal([])).toBe(0);
  });
  test('null入力: 適切なエラーをスロー', () => {
    expect(() => calculateTotal(null)).toThrow();
  });
});
```

---

### [test-code/example.js:2-7] 入力バリデーションが存在しない

**finding_id**: `CQ-004`
**状態**: `new`
**重要度**: High
**カテゴリ**: 入力検証

`calculateTotal(items)` は `items` が `null`/`undefined` の場合、またはリスト内の要素に `price` プロパティが存在しない場合に無防備なまま実行される。コーディングスタイル規則「ALWAYS validate user input」違反。

**対象コード**:
```javascript
function calculateTotal(items) {
  var total = 0;
  for (var i = 0; i <= items.length; i++) {
    total += items[i].price;  // items が null のとき TypeError; items[i] に price がなければ NaN
  }
  return total;
}
```

**推奨対応**:
引数の型・構造を事前バリデーションする。

```javascript
function calculateTotal(items) {
  if (!Array.isArray(items)) {
    throw new TypeError('items must be an array');
  }
  return items.reduce((total, item) => {
    if (typeof item?.price !== 'number') {
      throw new TypeError('Each item must have a numeric price property');
    }
    return total + item.price;
  }, 0);
}
```

---

### [test-code/example.js:2-8] エラーハンドリングが存在しない

**finding_id**: `CQ-005`
**状態**: `new`
**重要度**: High
**カテゴリ**: エラーハンドリング

関数全体に try-catch が存在せず、実行時エラーが呼び出し元に無加工で伝播する。コーディングスタイル規則「ALWAYS handle errors comprehensively」違反。エラーメッセージが不明確なまま上位に伝播するとデバッグが困難になる。

**推奨対応**:
エラーを捕捉し、有用なメッセージを付与してスローする。

```javascript
function calculateTotal(items) {
  try {
    if (!Array.isArray(items)) {
      throw new TypeError('items must be an array');
    }
    return items.reduce((total, item) => total + item.price, 0);
  } catch (error) {
    console.error('calculateTotal failed:', error);
    throw new Error(`Failed to calculate total: ${error.message}`);
  }
}
```

---

### [test-code/example.js:3,4] `var` の使用（`let`/`const` を使うべき）

**finding_id**: `CQ-006`
**状態**: `new`
**重要度**: Medium
**カテゴリ**: コードスタイル

`var` はブロックスコープではなく関数スコープを持つため、意図しない変数の参照が発生しうる。モダンな JavaScript では `let`/`const` を使用すること。

**対象コード**:
```javascript
var total = 0;              // line 3
for (var i = 0; ...         // line 4
```

**推奨対応**:
```javascript
let total = 0;
for (let i = 0; i < items.length; i++) {
```
`total` は再代入が発生するので `let`、ループ変数 `i` も `let` を使用する。

---

### [test-code/example.js:4,5,10,13] 既知バグをコメントで注釈するだけで修正していない

**finding_id**: `CQ-007`
**状態**: `new`
**重要度**: Medium
**カテゴリ**: コードスタイル / ポリシー違反

以下のコメントは、問題を認識しながらソースコードに放置している「What/How コメント」または未 Issue 化 TODO コメントに相当する。ポリシー「説明コメント（What/How のコメント）」「TODO コメント（Issue化されていないもの）」に違反。

| 行 | コメント |
|----|---------|
| 4  | `// オフバイワンエラー` |
| 5  | `// null チェックなし` |
| 10 | `// パスワードのハードコーディング` |
| 13 | `// テストなし` |
| 11 | `// セキュリティ問題` |

**推奨対応**:
- コメントで問題を指摘するのではなく、問題を修正する
- 即座に修正できない場合は GitHub Issue を起票し `// TODO: [#123] ...` 形式でトラッキングする
- `// テスト用のサンプルコード`（line 1）も What コメントであり削除する

---

### [test-code/example.js:11] `API_KEY` が未使用のまま宣言されている

**finding_id**: `CQ-008`
**状態**: `new`
**重要度**: Medium
**カテゴリ**: 未使用コード

`API_KEY` は宣言されているが、このファイル内で使用されておらず `module.exports` にも含まれていない。ポリシー「未使用コード（「念のため」のコード）」に該当。

**対象コード**:
```javascript
const API_KEY = "sk-1234567890abcdef";
// ...
module.exports = { calculateTotal };  // API_KEY は export されていない
```

**推奨対応**:
使用しないのであれば削除する。使用する場合は `CQ-002` の対応と合わせて環境変数から取得し、明示的に利用箇所を実装する。

---

## 総評

`test-code/example.js` は新規追加ファイルであり、全指摘がブロッキング対象。

- **Critical × 2**: オフバイワンエラー（実行時クラッシュ）、APIキーのハードコーディング（セキュリティ漏洩リスク）
- **High × 3**: テスト不在（ポリシー REJECT 基準）、入力検証なし、エラーハンドリングなし
- **Medium × 3**: `var` 使用、注釈コメント放置、未使用変数

ファイル全体を再実装するか、上記すべての指摘を修正した上で再レビューを依頼すること。
