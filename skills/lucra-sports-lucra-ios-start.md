> For the complete documentation index, see [llms.txt](https://docs.lucrasports.com/lucra-sdk/llms.txt). Markdown versions of documentation pages are available by appending `.md` to page URLs; this page is available as [Markdown](https://docs.lucrasports.com/lucra-sdk/sdks-and-apis/client-sdks/ios/skills/lucra-ios-start/skill.md).

# Lucra iOS — Starting Point

## Step 0 — State the goal, then route

Establish what the integrator is trying to do, then jump to the matching guide. It helps to pin the goal to one of these before writing integration code:

| Goal                                        | Go to                                                                                                                                                                                |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| First-time setup / build failing            | this skill: "Install", then "Troubleshoot"                                                                                                                                           |
| Show Mini Games, or fetch Mini Games data   | skill `lucra-ios-minigames`                                                                                                                                                          |
| A user's recent matchups on a custom screen | [Headless User Operations](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.11_headless_user_operations.md)                                                                              |
| Tournaments UI (tournaments home)           | `LucraFlow.homePage(location:)` — see [Tournaments Flows](/lucra-sdk/sdks-and-apis/client-sdks/ios/3.1_tournaments_flows.md)                                                         |
| Auth / user linking / demographics          | [Init](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.0_initialize_client.md), [Headless User Operations](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.11_headless_user_operations.md) |

## Install (start here)

Add the SDK from the published release with a package manager:

* **SPM:** add `https://github.com/Lucra-Sports/lucra-ios-sdk`, pin version `5.6.0` or later (verified to include the Mini Games APIs used here), and add the `LucraSDK` product.
* **CocoaPods:** use the published `LucraSDK` pod.
* **Recommended iOS version: iOS 18 or later.**

**Keep code signing enabled on your app target** — select a development Team, or set signing to "Sign to Run Locally" (ad-hoc). The SDK ships binary frameworks (GeoComplySDK, MobileIntelligence) that recent iOS simulators and devices won't load unless they're signed. Don't disable signing (`CODE_SIGNING_ALLOWED = NO`) — the app will still *compile*, but crash at launch with `dyld: Library not loaded … Trying to load an unsigned library`.

Full detail: [Project Setup](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.0.0_project_setup.md).

## Initialize

1. Create `LucraClient` once with `apiKey` + `environment` (use `.sandbox` for integration). The client's tenant/host is resolved from the environment + key — there is no manual API-URL setting. `apiKey`, `environment`, and `urlScheme` are required; `merchantID` and `appearance` are optional (omit them for a default setup).
2. Delegate auth by presenting `LucraFlow.onboarding` — the SDK provides the login UI, so you don't need to build one.

Detail + exact initializer: [LucraClient Initialization](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.0_initialize_client.md). If your app uses Lucra deep links, [Deep Links](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.2_deeplinks.md) covers the full setup — registering your `urlScheme`, providing a shareable-link provider, and handling incoming links.

### Initialization checkpoints — worth verifying before you build features

* [ ] The app builds and launches against the SDK package (no `dyld` errors — see Troubleshoot).
* [ ] `LucraClient` was created without throwing.
* [ ] The `environment` matches the one your API key was issued for (a key can fail authorization against a different environment).
* [ ] *(for signed-in features)* After presenting `.onboarding` and completing it, `lucraClient.user` becomes non-`nil`.

If any of these don't hold, the Troubleshoot section below covers the common causes.

## Troubleshoot (symptom → check → doc)

| Symptom                                                                                     | First thing to check                                                                                                                                                                                                                             | Reopen                                                                                           |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Build can't find `LucraSDK` module                                                          | Package not resolved, or the `LucraSDK` product isn't added to your app target — re-resolve packages and confirm the product is attached to the target                                                                                           | [Project Setup](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.0.0_project_setup.md)                 |
| `dyld: Library not loaded … Trying to load an unsigned library` at launch (build succeeded) | Code signing is disabled. Enable it on the app target (a development Team, or "Sign to Run Locally") so the SDK's binary frameworks get signed — recent iOS simulators reject unsigned dynamic libraries. Don't set `CODE_SIGNING_ALLOWED = NO`. | [Project Setup](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.0.0_project_setup.md)                 |
| Init never completes / no user event                                                        | Network reachable? API key present and non-empty?                                                                                                                                                                                                | [Init](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.0_initialize_client.md)                      |
| Auth/401-style failures                                                                     | Key/environment mismatch (e.g. sandbox key on staging)                                                                                                                                                                                           | [Init](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.0_initialize_client.md)                      |
| `lucraClient.user` stays `nil` after onboarding                                             | Are you observing the published `user` (not reading it once)? Did onboarding actually complete?                                                                                                                                                  | [Headless Interactions](/lucra-sdk/sdks-and-apis/client-sdks/ios/1.2.9_headless_interactions.md) |
| A Mini Games flow shows nothing / says unavailable                                          | Mini Games must be enabled for your tenant on the backend                                                                                                                                                                                        | skill `lucra-ios-minigames`                                                                      |

## Definition of "initialized successfully"

The SDK is initialized once the app launches against the package and `LucraClient` is created — at that point signed-out surfaces like `.miniGamesHome` and the tournaments home already work.

For features that require an authenticated user, add one more checkpoint: after presenting `.onboarding`, confirm `lucraClient.user` becomes non-`nil`.
