# App-bug-reporter-with-screenshots

Shake-triggered in-app feedback overlay for Expo / React Native — bug reports with screenshots, feature voting, and auto-captured JS crash reports. Shake, screenshot, or 3-finger pinch to open the overlay (handled by the FlayBridge native module).

> Keywords: expo bug reporter, react-native bug report with screenshots, in-app feedback SDK, shake to report bug, expo config plugin, crash reporting with screenshot, feature request board.

![Bug report with screenshot demo](https://images.openai.com/static-rsc-4/RNwSQFnm7oYPOOhbqiVKqOOAPLnsnnxg0s5eUb7WSiDKAuJHFlU6D8ZeopQW_wMqNHchTFXx6slMf9juW8udSyuNZf4O3aiUiBOZBKkYOZScbb8LbRpFJxRSZsqCBtWB5kvfF_Wef7UPwAG9c6WxWHBVqKuB8Pt19_e4Io-1Hik?purpose=inline)

## Install

```bash
npm install app-bug-reporter-with-screenshots
# or pin to GitHub:
# npm install github:aruninf/App-bug-reporter-with-screenshots#main
npx expo prebuild --clean
```

The Expo config plugin (`./app.plugin.js`) injects the iOS native sources (`FlayDeck.{h,m}`, `FlayBridge.{h,m}`) into the generated `ios/` project on every prebuild — never copy them manually.

## Integrate (host app, 4 steps)

### 1. Register the Expo plugin

In `app.config.ts` / `app.json`:

```ts
export default {
  expo: {
    plugins: ["app-bug-reporter-with-screenshots"]
  }
};
```

Then run `npx expo prebuild --clean` so `FlayDeck` + `FlayBridge` land in the native project.

### 2. Wrap your root with `FlayProvider`

```jsx
import { FlayProvider } from "app-bug-reporter-with-screenshots";

function AppInner() { /* ... */ }

export default function App() {
  return (
    <FlayProvider
      config={{
        appName: "MyApp",
        appId: "myapp",
        version: "1.0.0",
        endpoint: "https://api.example.com/api/bug-reports",
        // abInfo: async () => [...], // optional: extra rows in the overlay
      }}
    >
      <AppInner />
    </FlayProvider>
  );
}
```

Config fields: `appName`, `appId`, `version`, `endpoint` (required). `endpoint` must be the canonical server route ending in `/api/bug-reports` — pass it via config, never hardcode a default. Uncaught JS errors auto-POST to `<apiBase>/api/app-errors` with a screenshot (once per error per session).

### 3. Stand up the backend (contract)

POST `{appId, version, note, screenshot, source, ts}` to `<endpoint>`. Response shape:

```json
{ "ok": true, "id": "<bug-id>" }
```

Anything else (HTTP non-2xx, network error, JSON parse fail) triggers the retry queue: capped at 10 in-memory entries, replayed on every app foreground (`AppState` → `active`), with a toast per attempt.

### 4. Trigger it on device

- **Shake**: open overlay
- **Screenshot**: open overlay (iOS user-triggered screenshot detection)
- **Pinch (3-finger)**: developer fallback

All handled in `FlayBridge.m` — no JS-side gesture handlers required. Programmatic control: `useFlay()` → `{ open, close }`, or `setFlaySuppressed(true)` to silence it (e.g. on login screens).

## Required Info.plist key

If the endpoint is `http://` (Tailscale-private), add to `app.config.ts`:

```ts
ios: {
  infoPlist: {
    NSAppTransportSecurity: { NSAllowsArbitraryLoads: true }
  }
}
```

Otherwise ATS blocks the request and the submit silently queues for retry.

## Compatibility matrix

| Package   | Expo SDK | React Native | React |
|-----------|----------|--------------|-------|
| v0.7.x    | 53+      | 0.74+        | 18+   |
| v0.4.0    | 53+      | 0.74+        | 18+   |
| v0.3.0    | 52+      | 0.73+        | 18+   |

`peerDependencies` enforce the floor — `npm install` fails loudly on older runtimes rather than producing a runtime crash.

## Publish to npm (maintainers)

Package name on npm (lowercase): `app-bug-reporter-with-screenshots`. GitHub repo keeps capitals; npm does not allow them.

```bash
npm login
# bump version (pick one):
npm version patch   # 0.7.8 -> 0.7.9
# npm version minor # 0.7.8 -> 0.8.0
npm publish --access public
npm view app-bug-reporter-with-screenshots version
```

Consumers then get it via `npm install app-bug-reporter-with-screenshots` — no GitHub tag needed. Keep the GitHub release tag in sync (`git tag v0.7.9 && git push --tags`) so the `github:aruninf/...#vX` install path keeps working.

## License

MIT — see `LICENSE`.
