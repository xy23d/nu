---
name: git
description: >
  複数のgit worktreeを一括操作するスキル。worktreeの状態確認や特定コミットの検索で複数回コマンドを叩いているときに使う。
  `/git worktrees [dir]` で全worktreeのブランチ・ahead/behind・未追跡ファイルを一括表示。
  `/git find <hash> [dir]` で特定コミットが各worktreeに含まれるか検索。
  `/git chain-rebase A B C` でブランチチェーンを順番にrebase。
  「worktreeの状態を確認したい」「このコミットがどのブランチに入っているか調べたい」「各ブランチのリモートとの差分を見たい」「A→B→Cの順でrebaseして」ときに必ず使うこと。
---

# git: worktree一括操作

スクリプトは `scripts/` に同梱済み。スキル起動時に示されるベースディレクトリ（"Base directory for this skill: ..."）を使って実行する。

## `/git worktrees [dir]`

```bash
bash <base_dir>/scripts/worktrees.sh [dir]
```

- `dir` 省略時はカレントディレクトリ
- clean（ahead:0 behind:0 untracked:0）は `✓` で表示、それ以外は詳細を表示

## `/git find <hash> [dir]`

```bash
bash <base_dir>/scripts/find-commit.sh <hash> [dir]
```

- `hash` は前方一致（短縮形OK）
- `dir` 省略時はカレントディレクトリ
- 結果は `FOUND` / `none` で各worktreeごとに表示

## `/git chain-rebase A B C [...]`

ブランチチェーンを順番にrebaseする（A→B→C の場合、BをAに、CをBにrebase）。

### 手順

各ペア（親→子）について：

1. 子ブランチのworktreeで `git rebase <親ブランチ>` を実行
2. コンフリクト発生時は **HEAD（親ブランチ）を優先** する
   - 既存ファイルの変更が競合した場合 → HEADの内容を採用
   - 子ブランチ固有の追加（新規ファイル、親に存在しないメソッド）のみ incoming から取り込む
   - 判断基準：「この変更は親ブランチにすでにあるか？」→ あればHEAD、なければ取り込む
3. `git add <file> && git rebase --continue` で続行

### worktreeの場所

ブランチ名からworktreeパスを解決する：`rails-worktrees/<branch-name>/`
