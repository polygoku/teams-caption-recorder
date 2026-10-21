# Teams caption compatibility notes

Research checked on October 21, 2026. These notes distinguish documented browser behavior from observed Teams markup. They are engineering references, not a claim that a particular Teams tenant or meeting has been tested.

## Documented behavior

- Microsoft documents that Teams live captions are not saved after the meeting, and that caption speaker names can be unavailable. The extension therefore records observed captions continuously and must allow an unknown speaker.
  - https://support.microsoft.com/en-us/teams/meetings/use-live-captions-in-microsoft-teams-meetings
- Microsoft's current live-caption article describes **More actions > Captions**. Its accessibility and pop-out caption articles still describe **More actions > Language and speech > Show live captions**. The extension supports these UI variants; availability and labels can vary with rollout and language.
  - https://support.microsoft.com/en-us/accessibility/teams/best-practices-for-setting-up-and-running-a-teams-meeting-or-live-event-for-the-deaf-and-hard-of-hea
  - https://support.microsoft.com/en-us/accessibility/teams/use-pop-out-captions-in-microsoft-teams-meetings
- Microsoft's current Windows shortcut table does not document a live-caption toggle. Do not assume that the Mac caption shortcut maps to Windows: `Ctrl+Shift+A` has another documented Teams action. Explicitly identified caption controls are the automation path.
  - https://support.microsoft.com/en-us/accessibility/teams/keyboard-shortcuts-for-microsoft-teams
- Microsoft documents migration from `teams.microsoft.com` to `teams.cloud.microsoft`. Both commercial host families are relevant.
  - https://learn.microsoft.com/en-us/ecdn/technical-documentation/network-requirements
  - https://learn.microsoft.com/en-us/microsoft-365/enterprise/cloud-microsoft-domain?view=o365-worldwide
- Chrome content scripts can read and modify page DOM, run in isolated execution contexts, and message their extension. `all_frames` covers frames matching the declared URL patterns; it does not provide arbitrary cross-origin access.
  - https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts
- Chrome documents extension storage behavior and quotas. IndexedDB is available to extension service workers. Reliable persistence must complete before a save is acknowledged; service-worker memory is not a durable transcript store.
  - https://developer.chrome.com/docs/extensions/reference/api/storage
  - https://developer.chrome.com/docs/extensions/develop/concepts/service-workers/lifecycle

## Observed Teams DOM selectors

The selectors below come from independent, primary open-source implementation evidence. **Microsoft does not document them as a stable public interface.** They are not proof of compatibility with every tenant or future Teams release.

| Purpose | Observed selector |
| --- | --- |
| Caption text | `[data-tid="closed-caption-text"]` |
| Caption list | `[data-tid="closed-caption-v2-virtual-list-content"]` |
| Alternate caption container | `[data-tid="closed-caption-v2-window-wrapper"]` |
| Alternate caption container | `[data-tid="closed-caption-renderer-wrapper"]` |
| Older caption row | `[data-tid="closed-caption-chat-message"]` |
| Newer caption row | `.fui-ChatMessageCompact` / `[class*="ChatMessageCompact"]` |
| Newer speaker, scoped to caption row | `[data-tid="author"]` |
| Older speaker, scoped to caption row | `.ui-chat__message__author` |
| Meeting-title candidate | `[data-tid="call-title"]` |

Primary implementation references:

- https://github.com/thinko/lost-in-transcription/blob/main/content.js
- https://github.com/thinko/lost-in-transcription
- https://github.com/IsmaelMartinez/teams-for-linux/issues/1554

The first project documents caption revisions and a virtualized caption list. Its synthetic per-node IDs are not a Microsoft-provided stable utterance identifier. This project's implementation should be original; the referenced extension is GPL-3.0.

## Required engineering checks

1. Observe text revisions as well as inserted rows; replace the current utterance revision instead of repeating each partial word.
2. Preserve completed rows when Teams removes or recycles their DOM nodes. Repeated genuine speech must remain present; text-only global deduplication is insufficient.
3. Reattach when Teams replaces the caption container.
4. Distinguish waiting for captions from successful capture. An empty caption pane is not proof captions are disabled, and silence is not proof a meeting ended.
5. Bound caption-enable attempts and verify state so an already-enabled toggle is not disabled inadvertently. Never choose a control merely because it contains a broad fragment such as `record`, `transcri`, or `caption`.
6. Keep pause and finish decisions across poll cycles and separate subsequent meetings. Account for multiple Teams tabs or matching frames.
7. Use caption observation times honestly; these are not exact audio timestamps. Text accuracy is inherited from the Teams captions that actually appear.
8. Capture begins when the extension can observe captions. It cannot reconstruct speech before that point or while Teams does not render captions.

## Validation boundary

Automated tests using synthetic Teams fixtures validate our logic and known DOM cases. They do not constitute a live Microsoft Teams validation. A signed-in meeting smoke test must verify the tenant's actual caption controls, speaker extraction, capture updates, pause/resume, meeting exit, and exported transcript.

The initial scope is desktop Chrome/Edge on commercial Teams web hosts. Teams native desktop apps, mobile clients, government/China domains, and pop-out-caption windows need separate compatibility verification unless explicitly covered in the release notes.
