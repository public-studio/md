# AI Studio ナレッジソース

ナレッジソースは、Markdown資料を公開参照URLで取得する機能です。プロンプト作品（/api/prompts）とHTMLアプリ用公開DB（/api/apps）とは、D1テーブル・API・URLを分けています。

## 使い方

1. Notepad Studioの「編集」→「ナレッジソースを追加…」→「＋ 新しいソース」を開きます。
2. 資料をMarkdownにして入力します。短い資料はAI Studio登録画面へのリンクから下書きを受け取れます。長い資料はChatGPTからMarkdown本文をコピーし、登録画面へ貼り付けます。
3. 「公開して登録」を押すと、Markdown本文を返す参照URLを発行します。
4. Notepad Studioの選択画面で1件以上選び、「プロンプトに追加」を押します。プロンプトには本文ではなく公開参照URLと短い案内を追加します。
5. AIが参照する公開目次は [GitHub Rawのナレッジ目次](https://raw.githubusercontent.com/public-studio/md/main/knowledge/index.md) です。目次は保存済みのMarkdownから自動生成されます。人間向けの従来URL `https://raw.githubusercontent.com/public-studio/md/main/knowledge/index.md` も互換用に維持されます。

公開参照URLは、URLを知る人がログインなしで取得できます。機密情報や公開できない資料は登録しないでください。登録した資料のタイトルを変更してもID・参照URL・本文は変わりません。

## API

APIベースURL: https://studio.kanazawa-u.work

### 新規登録

利用者が明示的に「公開して登録」を選んだときだけ呼び出します。

~~~js
const response = await fetch("https://studio.kanazawa-u.work/api/knowledge-sources", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    title: "研修資料",
    content: "# 研修資料\n\nMarkdown本文",
    contentFormat: "markdown"
  })
});
const source = await response.json();
// source: { id, title, referenceUrl, aiReferenceUrl, editToken, revision, ... }
~~~

editTokenは新規登録時に一度返ります。ブラウザー内に保管し、URLや共有プロンプトに含めないでください。サーバーにはトークンのSHA-256ダイジェストだけを保存します。

### メタデータと本文の取得

~~~js
const source = await fetch("https://studio.kanazawa-u.work/api/knowledge-sources/SOURCE_ID").then(r => r.json());
// Use aiReferenceUrl when the URL will be given to an external AI.
// referenceUrl remains the AI Studio compatibility URL for people and existing clients.
const markdownResponse = await fetch(source.aiReferenceUrl);
const markdown = await markdownResponse.text();
~~~

参照URL /knowledge/{id}.md は text/markdown; charset=utf-8 で本文だけを返します。

### AIが参照する公開目次

GitHub Rawの `knowledge/index.md`（互換URL `GET https://raw.githubusercontent.com/public-studio/md/main/knowledge/index.md`）は保存済みソースのタイトルと安定参照URLを一覧にします。AIが資料をたどる場合はRaw URLを使います。Markdown本文に次の見出しがある場合は、用途・説明・キーワードも索引に表示します。

~~~md
## 用途
この資料が役立つ場面や対象利用者

## 説明
資料の内容を短く説明

## キーワード
検索や選択に役立つ語句
~~~

見出しのない既存ソースは、用途などを推測せず、既知のタイトルとRaw参照URLだけを掲載します。索引に編集トークンは含まれません。ChatGPTなどは索引から資料を選び、必要な資料だけ `https://raw.githubusercontent.com/public-studio/md/main/knowledge/{id}.md` を取得する想定です。外部AIがURLを取得できるとは限らないため、取得できない場合は本文を貼り付けてください。

ChatGPTに資料のMarkdown化を頼むときは、用途が未確認なら短い選択肢を提案させ、利用者が選んだ用途を実際の `## 用途` 本文に含めます。必要に応じて `## 説明` と `## キーワード` も作ります。既存の `/knowledge/new` 登録フォームにリンクで下書きを渡せるのは、タイトルと本文を含むURL全体が8,000文字以下の場合です。それを超えるときはコピー貼り付けを使います。フォームを開いただけでは保存されません。

### 名前または本文の更新

作成時の編集トークンと現在のETagが必要です。titleだけを送れば本文は保持されます。contentの更新も可能です。

~~~js
const current = await fetch("https://studio.kanazawa-u.work/api/knowledge-sources/SOURCE_ID");
const updated = await fetch("https://studio.kanazawa-u.work/api/knowledge-sources/SOURCE_ID", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer EDIT_TOKEN",
    "If-Match": current.headers.get("ETag")
  },
  body: JSON.stringify({ title: "新しい表示名" })
});
~~~

同一ブラウザーの「自分が管理するソース」一覧はローカルに保存されます。別のブラウザーへ編集トークンや編集一覧は自動共有されません。公開一覧 `https://raw.githubusercontent.com/public-studio/md/main/knowledge/index.md` は参照用であり、編集トークンを返しません。

## AIでの参照について

プロンプトに参照URLを追加しても、ChatGPT・Claude・Copilotなどが毎回そのURLを自動取得するとは限りません。各AI環境でURLを取得できる場合に参照し、取得できない場合はその旨を明示するようプロンプトへ案内を追加します。
