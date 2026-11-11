# Spec: issue-6-expo-modules-core — import from `expo` instead of `expo-modules-core`

**Status:** Reviewed (pre-approved by Kurt in chat 2026-11-11, thread
1570070494768664606: "All" — prepare ready-to-review branch, push on go)
**Issue:** equationalapplications/expo-two-way-audio#6

## Problem

The package imports `expo-modules-core` directly (`requireNativeModule`,
`createPermissionHook`, `PermissionResponse`) without declaring it in
`dependencies` or `peerDependencies`. It resolves today only because `expo`
hoists `expo-modules-core` to the app root. Strict installers (pnpm without
hoisting, Yarn PnP) fail with `Cannot find module 'expo-modules-core'`.

## Root cause / evidence

- `src/ExpoTwoWayAudioModule.ts:1`, `src/hooks.ts:1`, `src/core.ts:1` import
  from `expo-modules-core` (issue #6 audit, confirmed in the working tree).
- Compiled `build/` output keeps the runtime imports.

## Proposed change (issue option 1 — preferred)

Import the three symbols from `expo` instead. `expo` re-exports them from
`expo-modules-core`; verified live against installed `expo@57.0.27`
(`node_modules/expo/build/Expo.d.ts` lines 5, 15, 16:
`requireNativeModule`, `type PermissionResponse`, `createPermissionHook`).
This leaves `expo` as the only native-capability peer — the current
create-expo-module convention. No `package.json` change required.

## Files touched

- `src/ExpoTwoWayAudioModule.ts` — import line
- `src/hooks.ts` — import line
- `src/core.ts` — import line

## Test plan

1. `npm run typecheck` (tsc --noEmit) — must pass.
2. `npm run build` — must pass (compiled output imports `expo`).
3. `npx biome check .` — must pass (repo format gate).
4. `grep -rn expo-modules-core src/` — must return no matches.
5. Release note: ship as 0.2.2 via the standard release recipe (version-bump
   PR → merge → signed tag); CHANGELOG entry rides the version-bump PR, not
   this fix.

## Out of scope

- Version bump / CHANGELOG / tag (release recipe handles these).
- Issue #6's option 2 (peerDependencies entry) — superseded by option 1.
