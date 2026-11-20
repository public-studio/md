# AI Studio 公開DBの使い方

この公開DBを使うと、単一HTMLのJavaScriptからデータを登録し、再表示後や別の端末から読み込めます。備品一覧、簡単な記録帳、学習用アプリなどの試作に使えます。

## 重要：データとアプリIDは公開扱いです

- ログイン、APIキー、MCP接続は不要です。HTMLのJavaScriptがHTTPS APIを呼び出します。
- アプリIDを知っている人は、そのアプリのデータを誰でも読み取り、追加、更新、削除できます。アプリIDは認証情報でもアクセス制御でもありません。
- HTMLのソース、ブラウザーの通信、共有メッセージなどからIDが知られる可能性があります。パスワード、個人情報、機密情報、重要な業務データは保存しないでください。
- アプリIDを一括で一覧・検索するAPIはありませんが、それだけでデータが非公開になるわけではありません。
- Notepad Studioのプロンプト作品保存とは別の仕組みです。

## 最初にアプリIDを一度だけ作る

新しいアプリを作るときだけ、次のリクエストを1回送ります。返ってきた12文字の `id` を画面に表示し、コピーできるようにしてください。

```js
const API_BASE = "https://studio.kanazawa-u.work"; // この説明ページではアクセス時の公開API先が入ります

async function createApp(title) {
  const response = await fetch(`${API_BASE}/api/apps`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title }),
  });
  const result = await response.json();
  if (!response.ok) throw new Error(`${result.error}: ${result.message}`);
  return result; // { id, title, public, createdAt, dataUrl }
}

// 利用者が「新しいアプリIDを作成」ボタンを押した時だけ呼び出す。
const app = await createApp("備品管理");
localStorage.setItem("inventoryAppId", app.id);
console.log("アプリID:", app.id);
```

**ページを開くたびに `POST /api/apps` を実行しないでください。** 新しいIDが毎回発行され、以前のデータと別の領域になります。作ったIDをブラウザー内に保存し、次回は同じIDを使います。`localStorage` の値はそのブラウザー内だけに保存されます。他の端末や別の人と同じデータを使う場合は、表示したアプリIDをコピーして渡し、相手側のアプリに入力してください。IDをアプリの設定欄やHTMLへ記録して共有する方法も使えます。

## APIの一覧

`{appId}` は発行済みの12文字ID、`{recordId}` は登録時に返る12文字のレコードIDです。

| メソッド | パス | 用途 |
| --- | --- | --- |
| GET | `/api/public-db/health` | DBの疎通確認 |
| POST | `/api/apps` | アプリ領域の作成。本文例 `{"title":"備品管理"}` |
| GET | `/api/apps/{appId}` | アプリ情報の取得 |
| GET | `/api/apps/{appId}/data?limit=50&offset=0` | レコード一覧。最大100件ずつ |
| POST | `/api/apps/{appId}/data` | レコード登録。本文は `{"data":{...}}` |
| GET | `/api/apps/{appId}/data/{recordId}` | レコード1件の取得 |
| PUT | `/api/apps/{appId}/data/{recordId}` | レコード全体の更新。本文は `{"data":{...}}` |
| DELETE | `/api/apps/{appId}/data/{recordId}` | レコード1件の削除 |

アプリごとの上限は1,000レコード、一覧は1回につき最大100件、レコード登録・更新リクエスト本文の上限は32 KiBです。登録時は `201`、通常の取得・更新は `200`、削除成功は本文のない `204` が返ります。

## HTMLアプリから登録・一覧・更新・削除する例

アプリIDは画面や設定から取得し、入力時に形式を確認します。以下の `appId` は一度作成して保存した値に置き換えてください。

```js
const API_BASE = "https://studio.kanazawa-u.work";
const appId = localStorage.getItem("inventoryAppId");
if (!/^[A-Za-z0-9_-]{12}$/.test(appId || "")) {
  throw new Error("アプリIDを入力するか、新しいIDを一度作成してください。");
}
const dataUrl = `${API_BASE}/api/apps/${encodeURIComponent(appId)}/data`;

async function api(url, options = {}) {
  const headers = { ...options.headers };
  if (options.body !== undefined) headers["Content-Type"] = "application/json";
  const response = await fetch(url, { ...options, headers });
  if (response.status === 204) return null;
  const result = await response.json().catch(() => ({}));
  if (!response.ok) {
    const code = result.error || `http_${response.status}`;
    const message = result.message || response.statusText || "通信に失敗しました。";
    throw new Error(`${code}: ${message}`);
  }
  return result;
}

// 登録。返された id をその行の編集・削除ボタンにひも付けます。
const created = await api(dataUrl, {
  method: "POST",
  body: JSON.stringify({ data: { name: "プロジェクター", status: "貸出可" } }),
});
console.log("登録されたレコードID:", created.id);

// 一覧。次のページがある時は nextOffset を使って続きも取得できます。
const page = await api(`${dataUrl}?limit=50&offset=0`);
console.table(page.items.map((item) => ({ id: item.id, ...item.data })));

// 1件取得。
const recordUrl = `${dataUrl}/${encodeURIComponent(created.id)}`;
const current = await api(recordUrl);

// 更新はそのレコードの data 全体を置き換えます。
await api(recordUrl, {
  method: "PUT",
  body: JSON.stringify({ data: { ...current.data, status: "貸出中" } }),
});

// 削除。
await api(recordUrl, { method: "DELETE" }); // 成功時は null（HTTP 204）
```

画面では通信中・成功・失敗を利用者に表示してください。通信エラー時にデータを消したり、別のアプリIDを自動作成したりせず、入力内容を保ったまま再試行できるようにします。APIエラーは `{ "error": "...", "message": "..." }` 形式です。主な例は、入力不正 `400`、アプリ／レコードなし `404`、件数上限 `409`、本文サイズ超過 `413`、DB利用不可 `503`、サーバーエラー `500` です。

ブラウザーから別ドメインのHTMLを開く場合もCORSに対応しています。JSON送信に使う `Content-Type` と `GET`、`POST`、`PUT`、`DELETE` の各メソッドを利用できます。

## APIベースURLとローカルHTML

このページのコード例では、配信時に `API_BASE` へ実行環境の公開APIベースURLが挿入されます。設定がある場合は `STUDIO_PUBLIC_URL`、ない場合はこのページを開いたリクエストのoriginが使われます。生成するHTMLではこの値を定数として設定してください。

- 同じWebサイト上で配信するHTMLなら `location.origin` を使えます。
- 別のWebサイト上で配信するHTMLは、この説明ページで表示された `API_BASE` を設定します。
- サンプルHTMLは配信時にデプロイ環境のAPIベースURLが挿入されます。公開ページからダウンロードしたHTMLは、その接続先を引き継ぎます。GitHub上の原稿を直接使う場合は、画面のAPIベースURL欄に公開API先を入力してください。
- `file://` で開くHTMLでは `location.origin` はAPIのoriginにならず、通常は `"null"` です。上記の公開APIベースURLをHTMLに設定してください。ブラウザーによってはローカルファイルからの通信自体を制限するため、その場合はHTMLをlocalhostのHTTPサーバーで開いてください。
- GitHub上の原稿を直接使う場合は、例に記載されたAPIベースURLを実際の接続先に設定します。このリポジトリのソースには公開ホスト名を固定していません。

## AIにHTMLアプリを依頼する例

この説明ページの本文をChatGPTまたはMicrosoft 365 Copilotへコピーしてから、次のように依頼してください。

> この公開DBの仕様を使い、備品名と状態を表で表示する単一HTMLの備品管理アプリを作ってください。新しいアプリIDは利用者がボタンを押したときだけ作成し、IDを画面に表示してコピーできるようにしてください。既存IDを貼り付けて再利用でき、登録・一覧・編集・削除を実際に動くJavaScript fetchで実装してください。再読み込み後に同じIDのデータを読み込み、別端末にもIDを渡せるようにしてください。通信中・成功・エラーを画面に表示し、API_BASEにはこの説明に記載された値を設定してください。

## 開発者向け

データは既存Cloudflare D1の `public_apps`・`public_app_records` テーブルに分離して保存します。マイグレーション `worker/migrations/0002_public_app_data.sql` を確認してから適用してください。ローカルテストはNode.js 22以上で次を実行します。

```sh
node --experimental-sqlite worker/tests/public-db.test.mjs
```
