# Internal extension contract

All runtime messages use a `type` beginning `TC_`. All replies are `{ok:true,...}` or `{ok:false,error}`. No remote endpoints. Settings are `autoStart:true`, `autoEnableCaptions:true`, `showOverlay:true` by default.

## Data

Session: `{id,title,startedAt,endedAt,status,updatedAt,lastSavedAt,segmentCount,url,tabId,frameId,meetingKey,diagnostics}`. Times are Unix milliseconds. Status is `capturing`, `waiting`, `paused`, `finished`, or `interrupted`. The UI must never imply that waiting is successful capture. `updatedAt` includes heartbeat metadata; only `lastSavedAt` is the last committed caption save.

Segment: `{id,speaker,text,startMs,endMs,firstSeenAt,lastSeenAt}`. `id` is unique within its session and stable over live caption revisions. `startMs` and `endMs` are elapsed milliseconds from session start. Export timestamps are caption observation times, not audio-aligned measurements. Preserve Unicode. User text is displayed using textContent, never innerHTML.

## Content -> worker

- `TC_GET_SETTINGS` -> `{settings}`
- `TC_BEGIN_SESSION`: `{clientId,meetingKey,title,url,startedAt,hasCaptions?,forceNew?}` -> `{sessionId,session,resumed}`. Worker supplies trusted sender tab/frame. Content must adopt the returned session.startedAt for offsets on resume and honor paused/finished status. `forceNew` is only for explicit user Start after finishing. Auto-start may return a paused/finished session to preserve user intent after reload. Actual-caption frames may replace an empty waiting owner. A non-owner receives `code:'NOT_OWNER'`.
- `TC_UPSERT_SEGMENTS`: `{sessionId,segments,diagnostics?}` -> `{savedAt,segmentCount}`. Retry failures with identical segment IDs. Worker acknowledges only after IndexedDB transaction commits.
- `TC_SET_SESSION_STATE`: `{sessionId,state,endedAt?}` -> `{session}`
- `TC_HEARTBEAT`: `{sessionId,status,diagnostics?}` -> `{session}`
- `TC_OPEN_LIBRARY`: `{sessionId?}` -> opens extension library.html with optional `?session=<id>`.

## Extension UI -> worker

- `TC_SET_SETTINGS`: `{settings:<partial>}` -> `{settings}`
- `TC_LIST_SESSIONS` -> `{sessions}` metadata sorted newest first
- `TC_GET_SESSION`: `{sessionId}` -> `{session}` including `segments` sorted startMs, firstSeenAt, id
- `TC_UPDATE_SESSION`: `{sessionId,title}` -> `{session}`
- `TC_DELETE_SESSION`: `{sessionId}` -> `{deleted:true}`. UI asks inline confirmation, and active sessions cannot be deleted.
- `TC_UI_STATUS`: `{tabId?}` -> `{status,session,diagnostics,tabId}`. Status may be `unavailable`.
- `TC_UI_COMMAND`: `{tabId?,command:'start'|'pause'|'resume'|'finish'|'enable-captions'}` -> worker sends `{type:'TC_COMMAND',command}` to appropriate Teams frame, returning content reply.

## Worker -> content

- `TC_COMMAND`: `{command}` -> `{ok:true,status,sessionId}` or error.
- `TC_STATUS` -> `{ok:true,status,sessionId,diagnostics}`. Include `diagnostics.meetingActive` for frame arbitration. Status examples: `idle`, `waiting`, `capturing`, `paused`, `finished`, `error`.
- `TC_SETTINGS_CHANGED`: `{settings}` -> apply new settings.

## Files / ownership

- Root: extension/manifest.json, extension/background.js, extension/core/storage.js, icons, tests, packaging, README.
- Content agent: extension/content.js, extension/core/caption-engine.js, extension/core/teams-adapter.js. Use classic scripts in that order in manifest; namespace `globalThis.TeamsCaptionRecorder`. Content only runs on manifest Teams hosts.
- UI agent: extension/popup.html, extension/popup.js, extension/library.html, extension/library.js, extension/ui.css, extension/core/export.js. UI scripts and export.js are ES modules. Clean modern light UI, teal/dark navy accents, accessible forms.

## Behavior

Resumed `TC_BEGIN_SESSION` replies include `recentSegments` (up to 50 persisted entries) for reconciling already-visible caption tails after reload. New replies omit it. Explicit observed meeting boundaries may also set `forceNew:true`.

Auto-start only on detected in-meeting UI (visible hang-up / leave button), or visible known live-caption DOM; never on chat or prejoin pages. Automatic caption enabling gets one bounded attempt per active meeting; user can retry. Click only explicit live-caption controls, never recording, transcription, meeting chat, or joining controls. Display exact manual fallback instructions when unsupported. User pause/finish must survive immediate polling and not auto-restart in the same meeting. Collect only known caption text elements. No microphone/audio recording, accounts, APIs, tracking, or hidden network calls. Avoid Teams chat scraping.
