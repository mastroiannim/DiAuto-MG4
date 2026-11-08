# Android 9 compiler-warning remediation plan

## Context and constraints

The `carDebug` and `carRelease` Kotlin compilations report warnings but no
compilation errors in the reviewed logs. This plan orders the follow-up work by
risk; it does not treat every warning as a defect or propose suppressing warnings
globally.

The car head unit runs **Android 9 (API 28)** and will not receive an Android
upgrade. API 28 compatibility is therefore a fixed requirement for every change.
The deprecated protocol fields used for navigation are a separate concern from
deprecated Android framework APIs.

## P1 — Investigate possible logic defects

1. **Duplicate `when` label** —
   `app/src/main/java/com/andrerinas/openheadunit/aap/protocol/MsgType.kt:44`.
   Check the protocol values and the affected branches. Merge or remove a branch
   only after confirming which message type it is intended to handle, preserving
   the existing protocol mapping.

2. **Unread assignments in the audio mixing loop** —
   `app/src/main/java/com/andrerinas/openheadunit/decoder/AudioMixer.kt:375-397`.
   Trace how `hasData` and `isPreRolling` affect the loop. Restore the intended
   control flow if an assignment is being ignored accidentally; otherwise remove
   only the demonstrably dead writes. Validate audio playback, pre-roll and
   navigation prompts.

3. **Possible unhandled Java enum null** —
   `app/src/main/java/com/andrerinas/openheadunit/aap/AapService.kt:401`.
   Determine whether the Java API can return null at this call site and handle
   that case explicitly without changing the behavior of valid enum values.

4. **Condition that is always true** —
   `app/src/main/java/com/andrerinas/openheadunit/aap/AapProjectionActivity.kt:1195`.
   Confirm the intended nullability of `cv`, then remove the redundant branch or
   correct the condition. Check projection view creation and teardown for
   regressions.

## P2 — Functional and preventive corrections

5. **Deprecated navigation fields** —
   `AapNavigation.kt`, `AapNavigationHelper.kt` and `SaicNavigationMapper.kt`.
   Verify behavior when `nextTurnDetail` or `nextTurnDistance` is not sent by the
   device. Retain compatible fallbacks and use supported replacement data where
   available; do not merely suppress these warnings.

6. **Ambiguous block-level annotation** —
   `app/src/main/java/com/andrerinas/openheadunit/decoder/VideoDecoder.kt:1512`.
   Reformat or parenthesize the expression so the annotation target is explicit
   and stable across Kotlin compiler versions. Keep the Android 9-compatible
   decoder path.

7. **Unreachable Elvis fallback** —
   `app/src/main/java/com/andrerinas/openheadunit/utils/Settings.kt:1526`.
   Decide from the settings contract whether null is possible. Restore a real
   fallback if required, or remove the redundant operator if the value is always
   non-null, and verify theme selection.

## P3 — Low-risk cleanup

8. Rename the shadowed variables in `AapProjectionActivity.kt:1193` and
   `RemoteControlReceiver.kt:51` so each reference is unambiguous.

9. While working in the affected files, remove redundant initializers, unused
   variables and parameters, unnecessary safe calls and other dead code only
   where doing so does not change public APIs, callbacks or Android 9 behavior.
   Do not expand this into a broad rewrite unrelated to the warning being fixed.

## P4 — Deprecated Android APIs

There will be **no global deprecation migration**. A deprecated API that works
on Android 9 is not, by itself, a reason to change working code.

- Keep the existing API when it is required for API 28 compatibility.
- Consider a replacement only for a concrete correctness, security or
  maintainability reason, and only when that replacement supports API 28.
- If another supported device needs a newer API, retain the Android 9 path and
  guard the newer path with an explicit runtime version check.
- Evaluate each warning individually; do not hide whole classes of warnings with
  blanket suppression annotations.

## Final verification

For each implementation change, run the relevant existing unit tests and the
project checks covering that code. Before closing the plan:

1. Run `./gradlew.bat :app:testCarDebugUnitTest :app:compileCarReleaseKotlin`.
2. Build the installable car package with `scripts/build_mg4.ps1`; do not assume
   that the ordinary Gradle `carRelease` APK is platform-signed.
3. Follow the safety and installation procedure in
   [MG4_TESTING.md](MG4_TESTING.md) and test on the Android 9 head unit while
   parked.
4. Exercise the affected behavior, including audio mixing and prompts,
   projection, navigation data/fallbacks and theme settings as applicable.
5. Confirm wireless Android Auto connection, audio, touch, navigation prompts
   and disconnect/reconnect still work on Android 9.
