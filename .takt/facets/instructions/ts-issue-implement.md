# ts-issue-implement: プロダクションコード実装

## 役割

あなたはコーダーです。`write_tests` で作成済みのテストがパスするようにプロダクションコードを実装します。

## 前提確認

- `{report:issue-plan.md}`: 受け入れ条件と実装方針を確認
- `{report:test-scope.md}`: 作成済みテストファイルの一覧を確認

## 手順

### 1. テストの確認

まず、現在のテスト状態（全 RED）を確認してください：

```bash
npm test
# または
npm run test
```

### 2. プロダクションコード実装

テストがパスするように実装してください。TDD サイクルを守ります：

1. 失敗するテストを確認（RED）
2. テストをパスさせる最小限の実装（GREEN）
3. コードを整理（REFACTOR）

### 3. ビルド検証（型チェック）

実装後、ビルド（型チェック）を実行して型エラーがないことを確認してください：

```bash
npx tsc --noEmit
# または
npm run typecheck
```

### 4. テスト実行

ビルド成功後、テストを実行して全テストがパスすることを確認してください：

```bash
npm test
```

### 5. frontend の E2E 確認（frontend のみ）

frontend パッケージを変更した場合は、EBT + PBT パス後に **playwright CLI** を使った動作確認を実施してください：

```bash
playwright-cli open <url>        # 対象ページを開く
playwright-cli snapshot          # 要素参照を取得
playwright-cli click <ref>       # クリック操作
playwright-cli fill <ref> <text> # テキスト入力
playwright-cli screenshot        # スクリーンショットで結果確認
```

- 受け入れ条件を満たすことをブラウザ操作で確認
- E2E 確認が完了するまで実装・修正を繰り返す

### 6. コミット

issue 単位でコミットしてください。コミットメッセージ形式：

```
fix: #番号 受け入れ条件の概要
```

例: `fix: #42 ユーザー登録フォームのバリデーション追加`

### 7. implementation-coverage.md の出力

以下を `{report:implementation-coverage.md}` に記述してください：

- 受け入れ条件 → テスト → 実装箇所の対応表
- 担保できていない箇所（あれば理由とともに）
- 人間による動作確認手順（ローカル起動方法・操作手順など）

## 出力

- `{report:coder-scope.md}`: 変更ファイル一覧
- `{report:implementation-coverage.md}`: 受け入れ条件の担保状況（フォーマット: `ts-issue-coverage`）
- `{report:commit-log.md}`: コミット一覧（フォーマット: `ts-issue-commit-log`）

## 遷移ルール

- 実装完了・全テストパス → `ai_review`
- 実装不能 → `ABORT`
