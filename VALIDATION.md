# Current validation — Teams Caption Recorder v0.2.3

Date: October 21, 2026. Device: KyFu-XPS, installed Chrome extension (same extension identity; existing transcripts preserved).

## Signed-in Teams smoke test

- PASS: Chrome extension details showed v0.2.3 and zero extension error entries.
- PASS: a private, muted real Teams meeting automatically enabled native Live Captions without any manual caption click.
- PASS: clicking Teams Leave ended the call and the extension marked its transcript Saved (0 entries) in the local library.
- PASS: the extension produced no audio, meeting announcements, overlay, or notifications.
- LIMIT: this deliberately silent smoke test does not independently validate nonempty speech capture under v0.2.3. Earlier signed-in meetings saved nonempty captions.

## Focused isolated Chromium browser tests

The test (node tests/lifecycle.test.cjs) used a separate temporary profile and intercepted Teams URL fixtures.

- PASS: automatic caption activation (direct menu and checkbox-style Language and speech submenu), without native Teams recording/transcription.
- PASS: Hangup auto-finished with final caption preserved in approximately 2.2 seconds.
- PASS: organizer-ended call auto-finished in approximately 4.4–4.8 seconds.
- PASS: no phantom restarts with leftover caption rows after Leave, no overlay, no page JavaScript errors.
- PASS: all 21 core Node tests.

The older broad browser suite passed its initial checks but failed the replaced-caption-iframe continuation regression during this run. This remains an unresolved separate integration test and is NOT counted as a full suite pass.

## Archived v0.1.0 validation and fingerprints

The historical record below concerns v0.1.0. Historical statements about untested live Teams and original source hashes are superseded by the v0.2.3 results above.

---

# Validation — Teams Caption Recorder 0.1.0

Date: October 21, 2026.

## Release result

**21 core tests and 21 real-browser integration checks passed.**

Browser used: 136.0.7103.25. The extension was loaded as an actual Manifest V3 extension in a separate temporary browser profile. All Teams URLs were intercepted and fulfilled with synthetic fixture HTML. No signed-in Teams account or real meeting was used.

**Live Teams compatibility is unverified.** Installation on the user’s laptop and a real meeting smoke test remain. This release is ready to install and evaluate; the synthetic test results do not prove compatibility with every Teams tenant or future UI release.

## Core test coverage

The 21 Node tests cover partial-caption revisions; silence; identified and anonymous repeated speech; recycled and remounted nodes; restored transcript tails; late speaker attribution; Unicode joiners; pause baselines; rolling text; empty sessions; sorted and immutable export inputs; safe filenames; all four formats; JSON schema; WebVTT escaping; literal markup text; and invalid inputs.

Command: `node --test tests/core.test.mjs` (the direct `node tests/core.test.mjs` runner also reports all 21 individual cases).

## Browser integration checks

1. PASS — No transcript starts on Teams prejoin/chat screen.
2. PASS — Auto-detect meeting and enable new Captions menu.
3. PASS — Live partial caption is revised in one committed transcript entry.
4. PASS — Late speaker name corrects existing entry.
5. PASS — Distinct repeated phrases, Unicode, and caption-only scope are preserved.
6. PASS — Duplicate rendered caption panes are scoped to one capture surface.
7. PASS — Pause excludes speech during pause; resume captures subsequent text.
8. PASS — Finish commits pending text and prevents automatic restart.
9. PASS — Transcript viewer safely renders caption text, renames, and fits mobile viewport.
10. PASS — Actual browser downloads work for TXT, Markdown, JSON, and WebVTT.
11. PASS — Finished state survives same-meeting page reload.
12. PASS — Teams SPA meeting transition creates a separate transcript.
13. PASS — Active reload reconciles persisted tail without duplicating it.
14. PASS — Older caption menu works; recording and Teams transcription are never clicked.
15. PASS — Active deletion is rejected; finished transcript deletion removes its data.
16. PASS — Extension message boundary rejects caption writes from the viewer.
17. PASS — Actual caption iframe takes ownership from empty waiting top frame.
18. PASS — Replaced caption iframe resumes immediately without duplicating saved tail.
19. PASS — Different meeting in replacement iframe is separated from prior transcript.
20. PASS — Popup and library load with no page JavaScript errors.
21. PASS — Saved transcript and renamed title survive a complete browser restart.

No page JavaScript errors were reported. Desktop and narrow-screen transcript layouts and the popup were visually inspected. The checks exercised real Blob downloads and real IndexedDB storage, including a full browser-process restart. Screenshots and temporary profiles are not included in this release.

## Known limits

- English menu labels are used for automatic caption assistance. Other interface languages need manual caption enablement.
- Teams DOM selectors are observed implementation details, not a supported Microsoft caption API.
- Captions provide the wording and speaker attribution; missing speaker names remain unknown.
- Timing is based on caption observation and is not audio-aligned.
- Transcript reconciliation and route-based meeting identity are heuristic when stable identifiers are absent. An unobserved meeting boundary or a severely changed DOM can require manual Finish/Start.
- Previously unrendered speech, paused speech, and speech lost during a disconnected browser cannot be reconstructed. Abrupt closure may lose the final unacknowledged update.
- The actual managed Chrome/Edge installation and user’s live Teams tenant have not been exercised.

## Extension source fingerprint

The following SHA-256 values identify the extension source verified by this release.

| File | SHA-256 |
| --- | --- |
| `extension/background.js` | `df7b79c1f23d1a2c743842aa77b9b670be6f82f0977f82fe23d015bc41267f99` |
| `extension/content.js` | `06a018e317e72e2eb2a931c6dd76b7b71ce96a77c81765ac9c5e6dc38f013ca3` |
| `extension/core/caption-engine.js` | `d0607a7a06f76a02e562dbb6a633f4fac5dfc534460f2b6a1d22a6d13f4b6da2` |
| `extension/core/export.js` | `44f5a71964b9d86c0a3f5b5c82a7502932abb9aa455703d88a495606519ed7f4` |
| `extension/core/storage.js` | `33e7a957057478041ba3bf941c09488a3b25e20a8a598b129167a7472576473b` |
| `extension/core/teams-adapter.js` | `3f9ac2d4c0468c2fe92e632388f343b2bc61863055b03056107f8d8ca3745455` |
| `extension/icons/icon-128.png` | `0f04274e5f53696b59ebb65dfba8767cbdcbbc8059926c24a0af601da6073806` |
| `extension/icons/icon-16.png` | `9f9cd0c666a2c7da2b51f978370b72ccda33f508eb128fe2cf4663e4163a7b7b` |
| `extension/icons/icon-32.png` | `c2acbb24153a991a7dc4fd0dc4f4ecaca7a44700b73390154a7fbc55cd97f7cc` |
| `extension/icons/icon-48.png` | `e0bbff1a6e8a3532d98f4703960afb6730a34a23cf8d1ed559e80b3fcf8a0858` |
| `extension/library.html` | `071c87faafe38be9f091ea75980e6aa67ccd639ad16cb73ee58844c37f5000ee` |
| `extension/library.js` | `b4f11125c34a1b4ac73e0285b3196a08d7e87e745d7d38ef0f2b1479eb5853f1` |
| `extension/manifest.json` | `78a28ee11196f426aba5562ea5ac59e5396456af523623c016d51a0cc905573e` |
| `extension/popup.html` | `38fd94959d6518555448ee29acab37b64fd06301dee12625b244c2236184e733` |
| `extension/popup.js` | `abf5f0d65b8d01cd44e46a6adb40f2fb2c215f4394c87e1515aa609d2f204012` |
| `extension/ui.css` | `0c1bef2d7795318dd84d8df289b57f3087b4f6b9db331f6ef418c375a5674f33` |
