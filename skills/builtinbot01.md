# スキル：チャットボット作成

## 起動条件
新しいチャットボットを作りたい、既存のボットを元に別ボットを作りたい。

## 進行
1. 用途、利用者、望ましい会話の雰囲気を確認（既知の情報は聞き直さない）。
2. 個別本体MD・個別スキル目次MDを `https://studio.kanazawa-u.work/docs/chatbot-factory/templates/bot-main.md` と `https://studio.kanazawa-u.work/docs/chatbot-factory/templates/bot-skills.md` に沿って作る。
3. 必要な共有スキルを目次から選ぶ。存在しないスキルが必要なら作成スキルを提案する。
4. 保存前にプレビューを出し、AI Studioのボット新規登録画面 https://studio.kanazawa-u.work/bots/newに引き渡す。
5. 登録結果から本体URL・目次URL・起動URLを提示する。実際に取得できるか試す。

## 結果
個別ボット2つのMDと、実際の登録結果に基づく固定URL。登録前はあくまで案として示す。


## 実装上の受け渡し
有効なAI生成スキル登録リンクを開くと、利用者のブラウザーがAI StudioのAPIへ1回だけ登録を送信し、保存成功を実際の応答で確認します。通常の `/skills/new` は従来の手動登録フォームです。ChatGPTからAPIへ直接書き込むことはありません。リンクが長すぎる、壊れている、または開けない場合は、Markdownをコピーして手動フォームへ貼り付けてください。