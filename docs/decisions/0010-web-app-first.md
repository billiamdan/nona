# ADR-0010: The phone app is a web app first

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

The app must run on iPhones, an iPad, Android phones and desktops, with no app store and no fee.

What the research found (November 2026):

- A web app served by the hub and installed to the home screen is the only option that needs no store, no code signing and no developer account on either platform. It covers the control UI, push-to-talk microphone capture and keeping a wall panel's screen awake. All of that requires HTTPS.
- A free native iPhone build expires after 7 days. Apple's paid developer programme is a yearly fee, which conflicts with [ADR-0001](0001-local-only.md).
- The web app falls short on background behaviour: on iOS a WebSocket dies when the app is backgrounded, nothing can listen continuously, there is no Web Bluetooth, and background notifications always pass through the browser vendor's push service.
- On Android, installing our own app over ADB needs no account and is untouched by Google's developer-verification programme.

## Decision

- The app is an installable web app served by the hub itself, from the same origin as the API. Every device uses it, including wall panels.
- HTTPS and secure WebSocket from the first version, with certificates from our own private certificate authority ([ADR-0021](0021-network-and-security.md)).
- Live state arrives over a WebSocket with aggressive reconnect and an HTTP fallback, because iOS drops sockets in the background.
- The UI is generated from the device registry: one card component per capability type. Adding a device adds its card with no change to the app ([ADR-0013](0013-self-describing-devices.md)).
- When background features are needed (an always-listening satellite, phone sensors, battery reporting, notifications with no third party), we add a small Android shell: a WebView plus a foreground service, installed over ADB. No native iPhone app is planned.
- Proposed toolkit: Svelte with Vite and TypeScript, built to static files. Wall panels will be old phones, so the app must stay light and is tested on the oldest device first.

## Consequences

- One codebase for every screen, and nothing to sign or renew.
- Each phone or tablet needs a one-time step to trust our certificate authority.
- iPhones get notifications only while the app is open. Whether to allow encrypted Web Push through Apple's push service is an open question that needs its own record, because it is an exception to [ADR-0001](0001-local-only.md) (see [open-questions.md](../open-questions.md)).
- Always-listening voice is Android-only by platform design ([ADR-0007](0007-voice-input-in-stages.md)).

## Revisit when

A needed feature cannot be delivered by the web app plus the Android shell.

Evidence: [app and security notes](../../research_notes/Local%20smart%20home%20stack%20options/app_remote_security.md), sections 1, 2, 3 and 7.
