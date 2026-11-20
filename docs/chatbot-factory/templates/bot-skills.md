# {{BOT_NAME}} — 利用スキル目次

本体Markdown: {{BOT_MAIN_URL}}
共通スキル一覧: {{GLOBAL_SKILL_INDEX_URL}}

## このボットで使うスキル

<!-- 登録済みの実際のスキルURLで置き換える。以下は構造例。 -->

- **自己改善・他ボット改善** — 自分/他のボットのMDを更新するための変更案を作る。URL: {{EVOLVE_BOT_SKILL_URL}}
- **スキル作成** — 新しいスキルMDを生成し登録画面に渡す。URL: {{CREATE_SKILL_URL}}
- **スキル改善・コピー** — 既存スキルの改良、派生スキルの新規作成。URL: {{EVOLVE_SKILL_URL}}
- **保存と起動URL** — 固定URLとq起動URLを扱う。URL: {{SAVE_LAUNCH_URL}}
- **ナレッジソース操作** — 公開資料の参照・登録・更新を既存機能へつなぐ。URL: {{KNOWLEDGE_SKILL_URL}}
- **Webアプリ用DB操作** — Webアプリの構造化データを既存APIへつなぐ。URL: {{WEB_APP_DB_SKILL_URL}}

## 呼び出しルール
最初は目次だけを確認する。依頼に関係するスキルを選び、そのMD本文を必要なときだけ読み込む。使えないスキルについては読めた・実行したと装わない。新規スキルは共通目次の登録内容に従って追加する。
