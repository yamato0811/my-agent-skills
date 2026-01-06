参考：https://zenn.dev/secondselection/articles/agent_md_tuning

# リポジトリガイドライン

## プロジェクト概要（Overview）


## コーディング規約（Coding Style Guidelines）

- 言語: Python 3.12（`pyproject.toml` の `requires-python` に準拠）
- フォーマット/リント: Ruff（`uv run ruff format .` / `uv run ruff check .`）
- 型チェック: Pyright（`uv run pyright`）
- 命名: `snake_case`（関数/変数）、`PascalCase`（クラス）、定数は `UPPER_SNAKE_CASE`
- 層分離: `src/api`（I/O）→ `src/services`（業務）→ `src/repositories`（DB）を意識し、LLMロジックは `src/agents` に隔離

## セキュリティ（Security considerations）

- 秘密情報（APIキー/認証情報/サービスアカウントJSON）はリポジトリにコミットしない（`.env` はローカル専用、共有は `.env.example`）
- 外部I/Oは入力検証を必須化（FastAPI + Pydantic でスキーマ定義し、型・長さ・許容値を制約）
- 認証/認可は API仕様に合わせて厳格化（例: v3 は JWT 前提。ログにトークン/個人情報を出さない）
- LLM入力はプロンプトインジェクション/機密漏えいを前提に設計（機密をプロンプトへ渡さない、ツール実行範囲を最小化）

## ビルド＆テスト手順（Build & Test）

- セットアップ: `uv sync`（必要に応じて `uv sync --all-extras --dev`）
- 起動: `uv run uvicorn src.main:app --reload --port 8080`
- テスト: `uv run python -m pytest`（カバレッジ: `uv run python -m pytest --cov=src`）
- 主要チェック: `uv run ruff check .` / `uv run ruff format --check .` / `uv run pyright`
- CI（GitHub Actions）: Ruff（check/format）+ Pyright

## ドキュメント運用（AI駆動開発）

### ディレクトリ構造

#### 永続的ドキュメント（`docs/`）

アプリケーション全体の「何を作るか」「どう作るか」を定義します。

##### 下書き・アイデア（`docs/ideas/`）

- 壁打ち・ブレインストーミングの成果物
- 技術調査メモ
- 自由形式（構造化は最小限）
- `/setup-project` 実行時に自動的に読み込まれる

##### 正式版ドキュメント

- `docs/product-requirements.md` - プロダクト要求定義書
- `docs/functional-design.md` - 機能設計書
- `docs/architecture.md` - アーキテクチャ設計書（技術仕様）
- `docs/repository-structure.md` - リポジトリ構造定義書
- `docs/development-guidelines.md` - 開発ガイドライン
- `docs/glossary.md` - ユビキタス言語定義（必要に応じて作成/更新）

##### アーカイブ（`docs/archive/`）

- 過去の検討資料・廃止済み仕様の退避先（現行仕様の参照元にしない）

#### 作業単位のドキュメント（`.steering/`）

- 特定の作業に特化
- 作業ごとに新規作成
- 履歴として保持

## 知識＆ライブラリ（Knowledge & Library）

- 実装前に `Context7 MCP Server` を利用し、`resolve-library-id` → `query-docs` で関連ライブラリの最新情報を取得する。

## メンテナンス_ポリシー（Maintenance policy）

- 会話の中で繰り返し指示されたことがある場合は反映を検討すること
- 冗長だったり、圧縮の余地がある箇所を検討すること
- 簡潔でありながら密度の濃い文書にすること
