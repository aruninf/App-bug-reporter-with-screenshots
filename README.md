# App-bug-reporter-with-screenshots

> By **Infinity Soft System** · maintained by **aruninf**

Shake-triggered in-app feedback overlay for Expo / React Native — bug reports with screenshots, feature voting, and auto-captured JS crash reports.

## Install

```bash
npm install app-bug-reporter-with-screenshots
npx expo prebuild --clean
```

The Expo config plugin injects the iOS native sources (`FlayDeck`, `FlayBridge`) on every prebuild — never copy them manually.

## Integrate

**1. Register the plugin** in `app.config.ts`:

```ts
export default {
  expo: {
    plugins: ["app-bug-reporter-with-screenshots"]
  }
};
```

**2. Wrap your root:**

```jsx
import { FlayProvider } from "app-bug-reporter-with-screenshots";

export default function App() {
  return (
    <FlayProvider
      config={{
        appName: "MyApp",
        appId: "myapp",
        version: "1.0.0",
        endpoint: "https://api.example.com/api/bug-reports",
      }}
    >
      <AppInner />
    </FlayProvider>
  );
}
```

`endpoint` must end in `/api/bug-reports`. Uncaught JS errors auto-POST to `<apiBase>/api/app-errors` with a screenshot, once per error per session.

**3. Backend contract:**

POST `{appId, version, note, screenshot, source, ts}` → expect `{ "ok": true, "id": "<bug-id>" }`. Failures queue in-memory (cap 10) and replay on app foreground.

**4. Triggers:** shake · screenshot · 3-finger pinch. Programmatic: `useFlay()` → `{ open, close }`, or `setFlaySuppressed(true)`.

If the endpoint is plain `http://`, add `NSAppTransportSecurity: { NSAllowsArbitraryLoads: true }` to `ios.infoPlist`, or ATS will block submits.

## Compatibility

| Package | Expo SDK | React Native | React |
|---------|----------|--------------|-------|
| v0.7.x  | 53+      | 0.74+        | 18+   |

## Publish (maintainers)

```bash
npm login              # one-time, as a user with publish rights
npm version patch      # 0.7.8 -> 0.7.9
npm publish --access public
```

## License

MIT © Infinity Soft System (aruninf)
