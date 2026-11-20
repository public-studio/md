# SYSTEM Skill: Web App DB Operations

## Purpose
Use the existing AI Studio Web App DB API for structured records belonging to a small HTML application. It is separate from prompt sharing, bot/skill Markdown, and Knowledge Sources.

## Before use
1. Read the current API guide: https://raw.githubusercontent.com/public-studio/md/main/docs/public-db.md.
2. Explain that an app id is public, not an access key. Anyone who knows it can read, add, change, or delete that app records. Never store secrets or sensitive data.
3. Reuse the app existing 12-character id. Create a new id only when the user explicitly asks and confirms the new app; do not create an id on every page load.

## API workflow
- API base: https://studio.kanazawa-u.work.
- Create an app with POST /api/apps only as an explicit user-requested action; retain the returned id.
- Read app metadata with GET /api/apps/{appId}.
- List records with GET /api/apps/{appId}/data?limit=50&offset=0 and continue only using a returned nextOffset.
- Add with POST /api/apps/{appId}/data and JSON { data: { ... } }.
- Read one record with GET /api/apps/{appId}/data/{recordId}.
- Replace a record with PUT /api/apps/{appId}/data/{recordId} and its complete data value.
- Delete one record with DELETE /api/apps/{appId}/data/{recordId} only when the user explicitly asks to delete that record.
- Handle 204 as a successful empty response. For other non-2xx results, preserve the user data and report the actual error.

## Implementation rules
- In HTML, use the documented HTTPS API with fetch, JSON content type, visible loading/success/error status, and preserve form values on failure.
- Store and reuse the app id in the app own settings or local storage; do not create a replacement id after a read/write error.
- Do not add authentication or claim the app id provides access control.
- A Markdown guide is documentation; it does not execute POST, PUT, or DELETE. Do not claim a record changed without an actual successful API response.