# Skill: Create a Skill

## Trigger
The user asks to create, improve, or register a reusable skill.

## Workflow
1. Clarify the skill's purpose, intended users, activation conditions, required inputs, and expected outputs. Ask only for information that is not already known.
2. Unless the user explicitly requests another language, write the complete reusable skill Markdown body in English by default, including its headings and operating instructions.
3. If the user requests Japanese or another language, write the skill Markdown in that requested language. The conversation with the user, displayed skill title, short description, and AI Studio interface may remain in Japanese.
4. Preserve the requested purpose, activation conditions, workflow, constraints, and output behavior when translating or drafting. Do not translate original knowledge-source material unless the user asks for that translation.
5. Distinguish actions that require a real API from actions that Markdown instructions alone can perform. Never claim an API action succeeded unless its result is available.
6. When the user wants to register the skill, prepare a browser handoff for AI Studio. Generate a fresh, unique 32-character lowercase hexadecimal `handoffId` and a 6–8 digit `receiptCode`; encode `receiptCode` as a JSON string so leading zeroes are preserved. Build a JSON object with exactly these handoff fields, populated with the reviewed content:

   ```json
   {
     "type": "skill",
     "title": "...",
     "description": "...",
     "markdown": "...",
     "receiptCode": "123456",
     "handoffId": "0123456789abcdef0123456789abcdef"
   }
   ```

   Serialize the object as UTF-8 JSON, encode those bytes as unpadded Base64URL, then place the result after `#factory=` on the existing `https://studio.kanazawa-u.work/skills/new` URL. Keep the displayed title in the user's requested language (Japanese is fine); keep the Markdown body in English unless the user explicitly requested another language. If the completed link is too long, malformed, or cannot be opened reliably, use the copy-and-paste fallback and the ordinary manual registration form. Do not omit or silently truncate skill content to force it into a link.
7. Explain that clicking the generated AI Studio link submits the registration request once in the browser. ChatGPT itself does not call the AI Studio API. Keep the generated receipt code in the conversation so the result can be checked later. Do not say the skill is registered until the browser reports success.
8. On browser success, tell the user to use the success page's copy button and paste its verification prompt back into this conversation. The server response is `{id,title,markdownUrl,revision}`; use the returned `title` and `markdownUrl` verbatim, never a guessed URL. Compare the prompt's receipt code exactly with the saved `receiptCode`. Only after they match, fetch the actual `markdownUrl` when the current environment permits and confirm the title and Markdown from that resource. If fetching or inspection is unavailable, report that browser registration succeeded and the receipt code matched, but the Markdown content could not be independently verified. Never imply that an inaccessible URL means registration failed, and never invent a skill ID, Markdown URL, receipt, or successful result.
9. Handle the outcomes clearly:
   - **Receipt pasted:** match the code, verify the actual Markdown URL when accessible, and report exactly what was verified.
   - **Registration failed:** report the failure based on the browser result, retain the draft and receipt code, and offer a safe retry or the manual form. Do not report success.
   - **Link could not be opened:** provide the copy-and-paste/manual-form fallback without claiming that registration occurred.
10. After a confirmed registration, use the returned stable skill URL in the shared index. If the user wants, also prepare an update for a bot's private skill index.

## Updating an Existing Skill

When the user asks to improve an already registered skill while keeping its identity, follow this update flow instead of creating a new skill:

If the user has provided only a display name, or that name could refer to more than one skill, first read the live shared index at `https://raw.githubusercontent.com/public-studio/md/main/skills/index.md`. Show matching candidates with their title, description, ID, and stable Markdown URL. Ask the user to choose when the match is ambiguous; do not infer an ID from a name or start an update handoff before the target is clear. After the user chooses, verify the selected ID against `GET /api/skills/{id}`.

1. Read the skill's actual stable Markdown URL (for AI retrieval, use the URL shown in the Raw shared index) and request `GET /api/skills/{id}`. Use the real returned ID, title, description, Markdown, and revision; do not infer them from a display name or an old draft. If the skill is a built-in immutable skill, do not attempt to update it—offer to copy it into a new skill ID.
2. Show the proposed changes and preserve the existing skill's language and content. The English-by-default rule above applies to newly created skill bodies; do not wholesale translate an existing skill unless the user explicitly asks for translation.
3. Create a fresh unique 32-character lowercase hexadecimal `handoffId` and a 6–8 digit `receiptCode` serialized as a JSON string. Build the update payload with the reviewed draft:

   ```json
   {
     "type": "skill-update",
     "id": "existing-skill-id",
     "title": "...",
     "description": "...",
     "markdown": "...",
     "receiptCode": "123456",
     "handoffId": "0123456789abcdef0123456789abcdef",
     "expectedRevision": 7
   }
   ```

   Include `expectedRevision` as a number when `GET /api/skills/{id}` supplies a revision. If no revision is available, omit it and include `sourceSnapshot` with the exact source title, description, and Markdown read from the live skill:

   ```json
   "sourceSnapshot": {
     "title": "exact live title",
     "description": "exact live description",
     "markdown": "exact live Markdown"
   }
   ```

   Do not fabricate either concurrency value. If the live skill cannot be read, do not generate an update handoff; use the manual editor or ask the user to provide the current source.
4. Serialize the payload as UTF-8 JSON, encode it as unpadded Base64URL, and place it after `#factory=` on `/skills/{id}/edit`. Measure the complete finished URL, including its origin, path, and fragment; it must be no longer than 65,553 characters. If it is longer or invalid, do not truncate or simplify away skill content; use copy-and-paste/manual editing instead.
5. Explain that opening the link triggers one browser update request to the same skill ID. AI Studio checks the expected revision with `If-Match`, or compares the exact `sourceSnapshot` with the current live skill when no revision was supplied, before updating. The stable Markdown URL remains the same. ChatGPT does not directly call the API. If the handoff link cannot be opened, offer the manual editor and make no claim that an update occurred.
6. On browser success, have the user use the success page's copy button and paste its verification prompt here. Keep and compare the original `receiptCode` exactly. Use the actual server-returned Markdown URL and revision from the prompt; fetch the Markdown and `GET /api/skills/{id}` where accessible, then verify the URL still identifies the same skill, the revision matches, and the live Markdown reflects the reviewed update. If the receipt code does not match, or the live resource cannot be inspected, state precisely what is unverified. Do not invent a URL, revision, or success result.
7. If the browser reports a failed or conflicting update, report that result, retain the source and draft, and offer review/retry or manual editing. Do not claim success. For a distinct variant, preserve the original skill and use the copy workflow to create a new ID.

## Output
A human-facing title and short description, plus the complete skill Markdown body in the requested language (English by default). Do not invent a skill URL or claim that it is registered before the save result is confirmed.

## Language example
For a request to create a skill named 「3行要約」, the displayed title may remain Japanese while the generated Markdown body is English, for example:

~~~md
# Skill: Three-Line Summary

## Trigger
The user asks for a concise summary in three lines.

## Workflow
1. Identify the source text and its main point.
2. Write exactly three concise lines without adding unsupported facts.

## Output
Return exactly three summary lines.
~~~

## 実装上の受け渡し
有効なAI生成スキル登録リンクを開くと、利用者のブラウザーがAI StudioのAPIへ1回だけ登録を送信し、保存成功を実際の応答で確認します。通常の `https://studio.kanazawa-u.work/skills/new` は従来の手動登録フォームです。ChatGPTからAPIへ直接書き込むことはありません。リンクが長すぎる、壊れている、または開けない場合は、Markdownをコピーして手動フォームへ貼り付けてください。

## 既存スキルの更新
更新時は対象の固定Markdown URLと `https://studio.kanazawa-u.work/api/skills/{id}` の実データを取得し、実際の `revision` と本文に基づいて `https://studio.kanazawa-u.work/skills/{id}/edit#factory=...` のリンクを作ります。リンクを開いたブラウザーが最新revisionを再取得し、`If-Match` を付けた更新要求を一度だけ行います。revisionが取れないときは、読んだ元データと現在の保存内容が完全一致する場合だけ更新します。競合・不一致なら上書きせず、手動編集へ切り替えます。更新後もIDと固定Markdown URLは維持されます。ChatGPTがAPIへ直接書き込むことはありません。