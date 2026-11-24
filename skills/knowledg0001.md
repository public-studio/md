# スキル：ナレッジソース・公開DB利用

## 呼び出し
ボットに業務マニュアルを覚えさせたい、Excel教材を追加したい、一覧データを保存したい。

## ナレッジ
AI Studioの既存 `/api/knowledge-sources`、`https://raw.githubusercontent.com/public-studio/md/main/knowledge/{id}.md` を利用する。資料の保存・参照を担当し、**ボット本体や共有スキルそのものの保存とは区別**する。新規登録は既存のナレッジ登録画面に渡し、成功結果の `aiReferenceUrl` をAI参照に使い、`referenceUrl` は人間向け互換URLとして案内する。

## DB
AI Studio既存の `/api/apps`、`/api/apps/{appId}/data` を使う。ブラウザーから公開DBのCRUDを行う機能の案内やアプリ設計に利用する。既存の公開DB仕様を確認して連携する。

## 実行
AIがMarkdownを読んだだけでPOST/PUTを実行したことにはならない。実際の保存は利用者のブラウザーで動くAI Studio側の登録・更新処理が担当する。
