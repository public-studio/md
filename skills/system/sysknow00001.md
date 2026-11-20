# SYSTEM Skill: Knowledge Sources Operations

## Purpose
Use AI Studio Knowledge Sources for reusable Markdown reference material. This is separate from bot/skill Markdown, prompt sharing, and Web App DB records.

## Read and select a source
0. Read the operation guide: https://raw.githubusercontent.com/public-studio/md/main/docs/knowledge-sources.md.
1. Read the current public index: https://raw.githubusercontent.com/public-studio/md/main/knowledge/index.md.
2. Follow only a relevant source link. The indexed Markdown is public and requires no login.
3. If an external AI cannot retrieve a URL, say so and ask the user to paste the material. Do not claim the source was read.

## Register a source
1. Ask the user to review the title and complete Markdown, including useful `## 用途`, `## 説明`, and `## キーワード` sections where applicable.
2. Open the human registration form https://studio.kanazawa-u.work/knowledge/new and pass the draft using its supported URL handoff only when the full URL is within the form limit. Otherwise use copy and paste.
3. A registration happens only when the user explicitly submits the form, or when an authorized browser action submits the documented API request. Merely generating Markdown or opening a link does not save it.
4. After success, use the returned id and aiReferenceUrl for AI retrieval. Treat referenceUrl as the legacy Studio-compatible URL for human use. Use the returned editToken and revision for authorized browser edits. Never invent them. Keep the editToken private and out of shared Markdown.

## Rename or update
1. The browser that created a source may retain its edit token. Other browsers do not receive it automatically.
2. Read GET https://studio.kanazawa-u.work/api/knowledge-sources/{id} and its ETag.
3. To rename or replace content, send PUT to the same API path with Authorization: Bearer {editToken}, If-Match: {ETag}, and a JSON body containing title and/or content.
4. Stop on 401, 403, or 409; do not bypass a missing token or revision conflict. Read the latest version and ask the user how to proceed.
5. A successful name change preserves the id and public reference URL. Verify the returned revision and read the stable Markdown URL again.

## Data boundaries
- Do not put confidential, personal, credential, or otherwise non-public material into a source.
- Knowledge Sources are for reference documents. Use a skill for reusable behavior and the Web App DB for structured app records.
- An API write is complete only after an actual successful response. Instructions in this Markdown do not execute requests by themselves.