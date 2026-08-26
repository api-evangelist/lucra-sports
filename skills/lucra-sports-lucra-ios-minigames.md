> For the complete documentation index, see [llms.txt](https://docs.lucrasports.com/lucra-sdk/llms.txt). Markdown versions of documentation pages are available by appending `.md` to page URLs; this page is available as [Markdown](https://docs.lucrasports.com/lucra-sdk/sdks-and-apis/client-sdks/ios/skills/lucra-ios-minigames/skill.md).

# Lucra iOS — Mini Games

**Prerequisite:** the Lucra iOS SDK is initialized (the `lucra-ios-start` "initialized successfully" bar). A signed-in user is **not** required to browse — `.miniGamesHome` renders signed-out. Sign-in is required only for user-scoped headless data (matchups/records) and interactive actions (entering a matchup, claiming rewards); the SDK prompts for it at that point. Requires LucraSDK `5.6.0` or later (the published release that exposes the Mini Games flows and `fetchUserProfileMatchups`), added via SPM/CocoaPods.

**Tenant note:** Mini Games are available to any tenant the backend has provisioned for them (they are *not* restricted to a single tenant). If a Mini Games flow renders empty or reports unavailable, the first thing to check is that Mini Games are enabled for your tenant.

## Choose your integration style

* Want Lucra's screens as-is, fastest path → **Non-headless (UI)**.
* Want your own landing/lists and only Lucra's data → **Headless**.
* Mixed is the norm and is supported: headless powers *your* lists, a flow handles the *interactive* part when the user taps in.

## Product vocabulary → API

| Product term          | Maps to                                                              |
| --------------------- | -------------------------------------------------------------------- |
| "Battle Mode" / 1v1   | `MiniGameMode.oneVsOne` (there is no "battle" case in the mode enum) |
| "Minigames Profile"   | `LucraFlow.miniGamesProfile`                                         |
| "Completed" matchup   | `MatchupStatus.result` (there is no `.completed` case)               |
| "In progress" matchup | `MatchupStatus.inProgress`                                           |

## Non-headless (UI)

You present an SDK flow; it renders and navigates itself. The flow cases:

* `.miniGamesHome` — the hub (browse games, start sessions, profile/rewards/tournament entry points).
* `.miniGame(gameId:gameMode:amount:matchupId:handlePostNavigation:)` — launches a specific session. All five arguments are required (`handlePostNavigation: Bool` included). Set `handlePostNavigation: true` to let the SDK route to matchup details on session end.
* `.miniGamesMatchupDetails(matchupId:)` — a single matchup's result/pending state.
* `.miniGamesProfile` — the user's Mini Games profile.
* `.miniGamesRewards` — achievements/rewards.

Present with `.lucraFlow($flow, client:)` (SwiftUI) or `lucraClient.ui.flow(_:)` (embedded/UIKit). Exact signatures + examples: [Mini Games Flows](/lucra-sdk/sdks-and-apis/client-sdks/ios/6.3_mini_games_flows.md), [Lucra Flows setup](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.7_lucraflows.md).

### Non-headless checkpoints

* [ ] Presenting a flow shows Lucra UI, not a blank screen (session valid? tenant provisioned?).
* [ ] Dismissal returns control to your app.

## Headless (data only)

You call `lucraClient.api.*`, get data, render it yourself.

* **Recent matchups for a custom landing/profile screen:** `fetchUserProfileMatchups(limit:)` returns matchups across **all** types + aggregate stats. Section client-side on `status` (`.inProgress` vs `.result`). Tapping a row → the type-agnostic `.matchupDetails(matchupId:)`; only route to `.miniGamesMatchupDetails(matchupId:)` when the row is a mini game matchup (other types land on an error screen there). → [Headless User Operations](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.11_headless_user_operations.md)
* **Launch a session yourself (own WebView):** `startMiniGame(gameId:gameMode:amount:matchupId:onProgress:)` returns a `MiniGameSession` whose `iframeURL` you load. Pre-warm geo with `preloadGeoToken(...)`. → [Mini Games Headless](/lucra-sdk/sdks-and-apis/client-sdks/ios/6.1_mini_games_headless.md)
* **Tenant catalog:** `getMiniGames()` lists the tenant's enabled games + config. → [Mini Games Headless](/lucra-sdk/sdks-and-apis/client-sdks/ios/6.1_mini_games_headless.md)
* **React to session end:** listen for mini game events (e.g. finished) rather than polling. → [Mini Game Events](/lucra-sdk/sdks-and-apis/client-sdks/ios/6.2_mini_game_events.md)

### Headless checkpoints

* [ ] A signed-in user with history returns non-empty matchups.
* [ ] In-progress vs completed are distinguishable (`.inProgress` vs `.result`).
* [ ] A headless-rendered row opens the correct matchup details (no "not available in Mini Games" error — that error means a non-mini-game row was routed to `.miniGamesMatchupDetails`).

## The bridge (the thing docs don't say in one place)

Headless and non-headless are designed to combine: `fetchUserProfileMatchups` fills *your* recent- matchups list; tapping a row presents the matching details flow (`.matchupDetails`, or `.miniGamesMatchupDetails` for mini game rows); a "play" button presents `.miniGamesHome` or a specific `.miniGame(...)`. Wiring exactly this — a custom landing plus the flows and the headless list — is the intended end state of an integration.
