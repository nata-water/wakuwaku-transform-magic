# プロジェクトガイドライン

> **📋 Constitution**: このプロジェクトの開発原則は `.specify/memory/constitution.md` に定義されています。  
> このファイルは、実装レベルの具体的なガイドラインを補完するものです。

## 開発環境 (Constitution 準拠)

- 開発環境は Windows11 であり、基本的には PowerShell の利用を想定すること
- Python の実行には、**uv を利用すること** (Constitution: Development Environment)
  - alembic マイグレーション作成時: `uv run alembic revision --autogenerate -m migration_comment`
  - Python スクリプト実行時: `uv run python script.py`
- フロントエンド(React)はポート 3000 で起動されます (Constitution 規定)
  - 起動コマンド: `npm run dev` または `npm start`
- バックエンド(FastAPI)はポート 8000 で起動されます (Constitution 規定)
  - 起動コマンド: `uv run uvicorn main:app --host 0.0.0.0 --port 8000 --reload`

## 情報源の優先順位 (Constitution Principle I 準拠)

tavily-remote MCP を利用してライブラリ選定を行う際は、以下を優先順とすること：

1. **公式ドキュメント・公式リリースノート** (最優先)
2. **GitHub Issues / Discussions / RFC など一次情報**
3. **ブログや Qiita 記事は補助とし、投稿日・更新日が古い場合はその旨を明示すること**
4. **非公式ソースのみを根拠に「ベストプラクティス」と断定しないこと**

## コード品質 (Constitution Principle III 準拠)

- フロントエンド(React)の Linter 及び Formatter は**Biome を利用し、warn や error とならないよう適切に対応すること**
- バックエンド(Python)の Formatter は**Ruff を利用すること**

## 命名規則

- フロントエンドの型定義や変数定義は、基本的には**キャメルケース**の利用を想定している
- ただし、バックエンドや外部 API から受け取る値がスネークケースを前提としている場合、フロントエンド側の項目名はバックエンド側に準じることとし、**スネークケースで書くことを許容する**

## UI/UX テスト (Constitution Principle II 準拠)

> **📋 Constitution Principle II**: UI/UX の変更は自動 E2E テストで検証すること (NON-NEGOTIABLE)

- フロントエンド(React)の画面を修正した場合、playwright(MCP)で UI が適切に実装されているかどうかを確認すること
- playwright の MCP を利用したタスクの後、ブラウザを閉じる必要はない
- playwright の MCP を利用する場合、ウェイトなども playwright の MCP を利用すること
- **サーバーは手動起動済み前提**: ユーザーが事前に `npm run dev`（フロントエンド）と `uv run uvicorn`（バックエンド）を起動しているため、playwright MCP 使用時にサーバー起動コマンドを実行する必要はない

## Copilot / Agent 作業効率化ガイドライン

### 基本方針

* まず最小限の探索で、変更対象レイヤーと触る予定のファイルを特定してください。
* リポジトリ全体を広く探索する前に、既存の実装パターンを1〜2箇所だけ確認してください。
* 同じファイルを何度も全文読みしないでください。必要な関数・コンポーネント・型定義の周辺だけを確認してください。
* 実装前に「触る予定のファイル一覧」と「想定する変更内容」を短く提示してください。
* 実装は差分中心で行い、ユーザーが依頼していない大規模リファクタリングは避けてください。

### 探索範囲の制限

* `.github/skills/tsf-closet-navigator/SKILL.md` は、対象領域が不明な場合のみ確認してください。
* `backend-map.md` / `frontend-map.md` などの詳細資料は、対象ファイルが特定できない場合のみ参照してください。
* 既存実装の確認は、同種パターンを最大2例までにしてください。
* 3例以上を確認したくなった場合は、先に「なぜ追加調査が必要か」を短く説明してください。
* grep/search は目的を明確にして実行し、曖昧な広域検索を繰り返さないでください。

### 実装計画の粒度

* 大きな機能は、以下の単位に分割してください。

  1. Backend model / service / router
  2. Frontend API / Context
  3. UI component / i18n / CSS
  4. Prompt injection / LLM integration
  5. Validation
* いきなり全体実装せず、まずMVPの差分を優先してください。
* 「後で拡張できるが、今は不要」な要素はスコープ外として明記してください。

### 検証範囲の制限

* lint / format / test は、原則として変更ファイルに絞って実行してください。
* 全体lint、全体test、広範囲E2Eは、以下の場合のみ実行してください。

  * 共有基盤を変更した場合
  * Context / Router / DB migration など影響範囲が広い場合
  * ユーザーが明示的に要求した場合
* UI変更時のPlaywright確認は必要ですが、対象画面・対象操作・期待結果を絞って実行してください。
* Playwrightで無関係な画面探索をしないでください。

### よく使う検証コマンド

* Frontend lint:
  `cd frontend; npx eslint <changed-files>`

* Frontend format check:
  `cd frontend; npx prettier --check <changed-files>`

* Backend lint:
  `cd backend; uv run ruff check <changed-files>`

* Backend format:
  `cd backend; uv run ruff format <changed-files>`

* Backend import sanity:
  `cd backend; uv run python -c "from gateway.routes import game_router; print('ok')"`

### Alembic 注意

* Alembicは必ず `backend` ディレクトリで実行してください。

  * 正: `cd backend; uv run alembic revision --autogenerate -m migration_comment`
  * 誤: リポジトリルートで `uv run alembic ...`
* autogenerateで無関係な差分が大量に出た場合、目的の変更だけにmigrationを手で整理してください。
* DB migrationを作成した場合は、upgrade / downgrade が目的の差分だけになっているか確認してください。

### MCP利用方針

* MCPは必要な場合に限定して使用してください。
* 「接続されているから最大限使う」のではなく、公式情報確認・UI確認・外部仕様確認など、目的が明確な場合に使ってください。
* ライブラリ・設定ファイル・外部API仕様を変更する場合は、公式情報を確認してください。
* 既存プロジェクト内の実装パターンで判断できる場合、外部検索を優先しないでください。


#### 確認手順

```
1. tavily-remote: 公式ドキュメントサイトを include_domains で指定して検索
   例: include_domains: ["biomejs.dev"], query: "Biome files configuration includes ignore"

2. deepwiki: GitHub リポジトリのドキュメント構造を確認
   例: repoName: "biomejs/biome", question: "How to configure file exclusion patterns?"
```

#### 禁止事項

- ❌ 学習データの知識のみで「この形式は古い/非推奨」と断定して変更する
- ❌ 裏取りなしで設定ファイルの構文を別形式に書き換える
- ❌ ユーザーが意図していない設定変更を「改善」として勝手に行う

## 言語設定

- **コミュニケーション**: 回答は常に日本語で行う
