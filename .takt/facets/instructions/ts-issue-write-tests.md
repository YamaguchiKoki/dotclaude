# ts-issue-write-tests: テスト先行作成

## 役割

あなたはコーダーです。`issue-plan.md` の受け入れ条件をもとにテストを先行作成します。プロダクションコードは作成しません。

## 前提確認

`{report:issue-plan.md}` を読み込み、受け入れ条件の一覧を把握してください。

## 手順

### 1. 既存テストパターンの確認

プロジェクトの既存テストファイルを調査し、以下を把握してください：

```bash
# テストファイルの検索
find . -name "*.test.ts" -o -name "*.spec.ts" -o -name "*.test.tsx" | head -20
# テストフレームワークの確認
cat package.json
```

- テストディレクトリ構成
- 使用しているテストフレームワーク（Jest、Vitest 等）
- fast-check の導入状況

fast-check が未インストールの場合は、プロダクションコードを作成する前にインストールしてください：

```bash
npm install --save-dev fast-check
```

### 2. EBT（Example-Based Test）の作成

各受け入れ条件に対して、具体的な入出力例を使ったテストを作成してください：

- 正常系: 期待される動作を検証するテスト
- 異常系: エラー処理・バリデーションのテスト
- 境界値: 最小値・最大値・空値のテスト

### 3. PBT（Property-Based Test）の作成

**fast-check** を使い、受け入れ条件をプロパティとして表現してください：

```typescript
import * as fc from 'fast-check';

test('property: ...', () => {
  fc.assert(
    fc.property(fc.string(), (input) => {
      // 不変条件（invariant）を記述
    })
  );
});
```

各受け入れ条件について、以下のいずれかのプロパティを定義してください：
- 冪等性（何度実行しても結果が同じ）
- 交換法則・結合法則
- 入力と出力の関係（逆関数の存在など）
- 不変条件（常に成り立つ制約）

### 4. テストの禁止事項

- **プロダクションコードの作成は禁止**。テストファイルのみ作成してください
- テストが RED（失敗）であることを確認してから次に進む

### 5. ビルド確認

テスト作成後、必ずビルド（型チェック）を実行してテストコードに構文エラーがないことを確認してください：

```bash
# TypeScript の型チェック
npx tsc --noEmit
# または
npm run typecheck
```

型エラーが出た場合はテストコードを修正してください。

## 出力

`{report:test-scope.md}` に作成したテストの一覧を出力してください（フォーマット: `ts-issue-test-scope`）。

## 遷移ルール

- テスト作成完了・ビルド成功 → `implement`
- ビルド失敗・作成不能 → `ABORT`
