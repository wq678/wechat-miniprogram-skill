---
name: wechat-miniprogram
description: Build, extend, debug, and prepare WeChat Mini Programs for release using native Mini Program APIs, TypeScript or JavaScript, and the user’s existing project conventions. Use for Mini Program pages, components, cloud capabilities, login, payments, platform APIs, performance, testing, or release readiness; not for ordinary web sites.
---

# WeChat Mini Program

Deliver a working Mini Program change that follows the repository’s tooling and architecture. Preserve the user’s chosen stack and avoid introducing framework migrations unless requested.

## Start with the project

- Inspect `project.config.json`, `app.json`, `package.json`, and the relevant page/component before editing.
- Determine whether the project is native Mini Program, Taro, uni-app, or another framework. Follow its conventions; do not mix framework APIs into a native project.
- Identify the target Mini Program base library and account capabilities when the task depends on platform APIs, plugin permissions, cloud services, payments, phone-number retrieval, or login.
- When an existing UX or data-access pattern is present, extend it instead of creating a parallel one.

## Build and change features

- Keep view markup, styles, behavior, and configuration aligned: `.wxml`/`.wxss`/`.js` or `.ts`/`.json` for native pages and components.
- Put user-visible states in the UI: loading, empty, failure, retry, and permission-denied states when applicable.
- Validate inputs before submitting and make repeated actions idempotent where the backend supports it.
- Prefer component properties, events, slots, and behaviors over reaching into child component internals.
- Use `wx.nextTick` only when a measurement genuinely depends on completed rendering. Clean up timers, event listeners, and observers in lifecycle teardown hooks.
- Keep client-side calculations and presentation separate from security-sensitive decisions, prices, inventory, roles, and final order status, which must be confirmed by a trusted server or cloud function.

## Platform and data boundaries

- Never embed AppSecret, payment merchant keys, cloud administrator credentials, third-party private keys, or long-lived API tokens in Mini Program source, configuration, or build artifacts.
- Treat `wx.login()`’s `code` as a short-lived one-time value. Send it to the application server over HTTPS; the server exchanges it with WeChat and issues the app’s own authenticated session.
- Treat `openid`, `unionid`, session keys, payment signatures, and server access tokens as server-side data. Do not log them or store them in client-readable files.
- Store only a short-lived, app-issued session marker client-side when needed, and handle expiry by re-authenticating. Do not assume `wx.login()` itself is a durable login session.
- Make outbound network requests through configured HTTPS domains and the project’s shared request layer. Normalize response and error handling there.
- Ask for confirmation before adding account-bound configuration, creating cloud resources, publishing, submitting a release, or making any payment-related call.

## Common flows

### Authentication

1. Call `wx.login()` only when a session is absent or expired.
2. POST the returned `code` to the app’s backend through the shared request client.
3. Let the backend verify/exchange the code and return the minimum app-session data needed by the client.
4. Update client state, retry the original request once if appropriate, and show a clear recovery state if login fails.

### Authorization and privacy

- Explain why a capability is needed immediately before requesting it.
- Check authorization state before prompting. Handle denial and later recovery through settings without blocking unrelated use.
- For user profile, phone number, location, camera, album, and other sensitive data, use the current officially supported flow and minimize collection.

### Payments

- The client requests an order from the application backend; it never calculates the trusted amount or signs payment parameters.
- The backend creates the payment transaction and returns only the client invocation parameters.
- Invoke the WeChat payment API, then confirm the final result by querying the backend or relying on its verified payment callback—not solely on the client callback.

## Quality checks

- Run the project’s existing lint, type-check, test, and build commands when available.
- Use WeChat Developer Tools or the project’s simulator to exercise the changed path on a supported base library version.
- Check narrow and wide device layouts, safe-area spacing, navigation behavior, error states, and repeated-tap behavior.
- For networked paths, test offline, timeout, malformed response, expired session, and permission-denied cases that are relevant to the change.
- Report what was verified and what requires the developer account, a device, a backend environment, or manual release review.
