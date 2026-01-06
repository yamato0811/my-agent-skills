---
name: git-branch-creator
description: >-
  Gitで「他のブランチ命名規則に従って」新規ブランチを作成する手順を定型化する。Use when: (1) 「ブランチを作って/切って」と依頼された、(2) 既存ブランチの命名規則を踏襲したい、(3) feature/hotfix 等のプレフィックスとスラッグ規則を揃えたい、(4) 作成〜pushまで一気にやりたい。
---

# git-branch-creator

## 目的

- 既存ブランチの命名規則（例：`feature/hara/#NN_title`）を観測して、新規ブランチ名を安全に決める。
- ベースブランチから切って、必要なら `origin` に `-u` で push する。

## 手順

### 1) 他の命名規則を確認する

- まず「既存がどうなっているか」を見る:
  - `git for-each-ref --format='%(refname:short)' refs/heads refs/remotes | rg '^(feature|hotfix)/' | head -n 50`
- このリポジトリの観測例:
  - feature: `feature/<owner>/#<NN>_<slug>`（例: `feature/hara/#37_dependency_version_upgrade`）
  - hotfix: `hotfix/<slug>`

### 2) 変更内容を見て、ブランチ名（slug）を決める

- 変更内容の把握（例）:
  - `git status -sb`
  - `git diff --stat`
- `<slug>` は短い英小文字（`_` / `-` は既存に合わせる）
  - 例: 依存ライブラリの更新なら `dependency_version_upgrade` など

### 3) Issue番号（NN）は必ず連番にする（feature の場合）

- `feature/<owner>/#NN_` の `NN` は **ローカル + リモート(refs/remotes)** の最大値+1にする。
- 可能なら事前に `git fetch --prune origin` で最新化してから採番する（ネットワークが必要な環境では承認/権限が必要な場合あり）。
- 次番号を出す（例: owner=`hara`）:
  - `OWNER=hara; MAX=$(git for-each-ref --format='%(refname:short)' refs/heads refs/remotes | rg \"^((origin/)?feature/${OWNER}/#[0-9]+_)\" | rg -o '#[0-9]+_' | rg -o '[0-9]+' | sort -n | tail -n 1); MAX=${MAX:-0}; echo $((MAX+1))`

### 4) ベースブランチから切る

- ベースブランチ（通常 `develop`）へ移動して作成:
  - `git switch develop`
  - （可能なら）`git pull --ff-only`
  - `git switch -c <branch>`
- もし `Operation not permitted` / `cannot lock ref` などで失敗する場合:
  - 実行環境（例：Codex CLIのサンドボックス/権限制限）が `.git` 配下への書き込みを阻害している可能性があるため、権限昇格（承認付きで再実行）する。
