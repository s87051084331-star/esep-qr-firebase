# Esep QR — Firebase edition

Математика, алгебра және геометрия үшін қазақша бағалау кабинеті. Толық іске қосу нұсқаулығы: **[BASTAU_KZ.md](BASTAU_KZ.md)**.

## Stack

React 19 + TypeScript + Vite. Firebase Authentication (Google), Firestore, Cloud Storage, optional Firebase AI Logic / Gemini and reCAPTCHA Enterprise App Check. No ChatGPT or Cloudflare backend is required. Firebase modular SDK 13.1.0 is loaded from Google's official CDN at runtime; model name is configurable.

## Run

```bash
npm install
node scripts/setup.mjs
npm test
npm run build
```

`dist/` is included in the downloadable archive for an initial deployment without a local build. `node scripts/setup.mjs` updates its public configuration when present. The Git repository normally ignores generated `dist/`; CI rebuilds it.

## Data and access

Records are stored under `users/{firebaseAuthUid}/{classes|assessments|submissions}/{recordId}` as bounded JSON payloads. A Firestore transaction updates changed records and a workspace revision, rejecting stale writes. Reads require a consistent server revision. Each document is capped below Firestore's 1 MiB limit. Images live under `users/{uid}/files/{randomId}`. Storage access uses authenticated `getBlob`; no public download token URLs are generated. Apply the supplied Storage CORS configuration.

Rules restrict each user's data to that user. Rules are included but have not been executed against a live project or emulator here. The client validates assessment schemas and score bounds. Users can edit their own records; this is a teacher workspace, not a tamper-proof external exam system.

## AI behavior

AI is off by default. Configure your own Firebase AI Logic project and App Check, then enable it in `public/firebase-config.json`. Each page is sent with the corresponding teacher task and rubric. Runtime validation rejects unexpected or duplicate criteria, noninteger or excessive scores. Uncertain criteria remain unscored. AI results remain a draft until the teacher explicitly applies and approves them. The bundled default model follows Firebase's documentation as of 2026-10-28; change it when your project uses a different supported model.

Live Firebase sign-in, security-rule behavior, CORS and actual Gemini requests require the owner's project configuration and were not exercised in this environment. Browser visual QA was unavailable. Source type checking, production compilation, QR decoding and domain/AI validation tests were run.

## Third-party assets

- `public/jsqr.mjs`: cozmo/jsQR, upstream blob `99ea9df26907009e5553233ffe03c529c1521739`, MIT. The UMD receiver was changed to `globalThis` and an ESM default export added. License: `public/JSQR-LICENSE.txt`.
- `public/pdf.mjs` and `public/pdf.worker.min.mjs`: matched PDF.js distribution, Apache-2.0. License: `public/PDFJS-LICENSE.txt`.
- React, Lucide, Zod and build dependencies retain their upstream licenses.

The sample tasks are technical demonstrations, not approved official БЖБ/ТЖБ materials. No payment/subscription or electronic gradebook integration is included.
