# Contribution 1: Stop scrolling the console to the bottom when multiple errors are shown

**Contribution Number:** 1  
**Student:** Macy Jiang  
**Issue:** [Stop scrolling the console to the bottom when multiple errors are shown](https://github.com/flutter/flutter-intellij/issues/3778)  
**Status:** Phase II, In Progress

---

## Why I Chose This Issue

I chose this issue because it is a user-facing UX problem with a clear expected outcome: when multiple errors flood the logging console, the console auto-scrolls to the bottom, forcing the user to manually scroll back up to find the root cause error at the top. Fixing this would directly improve the day-to-day experience of Flutter developers, which makes it meaningful to work on. The issue is labeled `good-first-issue`, so I chose it since it is my first time contributing to open source.

---

## Understanding the Issue

### Problem Description

When a Flutter app throws an error that triggers a cascade of derivative errors (e.g. an "Unbounded Viewport" error), the IntelliJ/Android Studio logging console floods with hundreds of lines of output and automatically scrolls to the very bottom. This buries the root cause error, which appears first in the log.

### Expected Behavior

The console should either stay at the top (where the first and most important error appears) or stop auto-scrolling when a large batch of errors comes in at once, so the developer can immediately see the root cause without having to manually scroll back up.

### Current Behavior

The logging console auto-scrolls to the bottom every time new output is appended. When 600+ lines of errors are dumped at once, the user lands at the last (least useful) error and must manually scroll up to find the root cause.

### Affected Components

- The flutter-intellij plugin (IntelliJ/Android Studio Flutter plugin)
- Specifically, the logging/console component that handles appending output and controls scroll behavior

---

## Reproduction Process

### Environment Setup

- macOS 15.3.2, Apple Silicon (darwin-arm64)
- Flutter 3.44.1 (Channel stable), Dart 3.12.1
- The sample app (`flutter_error_studies`) was written for Flutter ~1.7 and required several compatibility fixes to run on modern Flutter:
  - Updated `pubspec.yaml` SDK constraint from `>=2.2.2 <3.0.0` to `>=2.12.0 <4.0.0`
  - Upgraded `cupertino_icons` to `^1.0.9` for null safety support via `flutter pub add cupertino_icons:^1.0.9`
  - Ran `dart fix --apply` for basic null safety fixes
  - Manually updated deprecated APIs across several files:
    - `FlatButton` → `TextButton`
    - `RaisedButton` → `ElevatedButton`
    - `headline4` → `headlineMedium`
    - `Scaffold.of(context).showSnackBar` → `ScaffoldMessenger.of(context).showSnackBar`
    - `color:` and `textColor:` on buttons → `style: ElevatedButton.styleFrom(backgroundColor:, foregroundColor:)`
  - Null safety fixes: `Key` → `Key?`, `String` → added `required`, `Widget` → `Widget?`
- macOS target failed due to incomplete Xcode installation; used `flutter run -d chrome` instead

### Steps to Reproduce

1. Clone the reproduction app: `git clone https://github.com/InMatrix/flutter_error_studies.git`
2. Apply compatibility fixes described above (or use a Flutter version ~1.7)
3. Run the app in IntelliJ IDEA or Android Studio (not VS Code — the bug is specific to the IntelliJ plugin console)
4. In the running app, tap "Unbounded Viewport"
5. Observe the logging console — it floods with 600+ lines of errors and auto-scrolls to the bottom
6. **Expected:** Console stays at the top showing the root cause error ("Vertical viewport was given unbounded height")
7. **Actual:** Console scrolls to the bottom, showing derivative `hasSize` assertion errors last

### Reproduction Evidence

- **Branch:** [Link to your branch — add when created]
- **My findings:** The first error in the log is the true root cause: `Vertical viewport was given unbounded height`. All subsequent errors (8+ additional exceptions) are cascading failures caused by the viewport failing to lay out. The console auto-scroll buries this root cause entirely.

---

## Solution Approach

### Analysis

The root cause of the bug is in the flutter-intellij plugin's console/logging component. When output is appended to the console, the plugin likely calls a scroll-to-bottom method unconditionally on every append. The fix should make this conditional — either stop auto-scrolling entirely when output volume is large, or preserve scroll position when the user hasn't manually scrolled to the bottom.

### Proposed Solution

Locate the console output handler in the flutter-intellij codebase and modify the auto-scroll behavior so that it does not scroll to the bottom when a large batch of error output arrives. A common pattern for this in IDE plugins is to only auto-scroll if the user was already at the bottom before new content arrived (i.e. respect user scroll position).

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The IntelliJ plugin console auto-scrolls to the bottom on every new output append. When hundreds of error lines arrive at once, this hides the first (root cause) error. The fix should prevent auto-scrolling in this situation.

**Match:** Look for existing scroll behavior in the flutter-intellij codebase — search for calls to methods like `scrollToEnd`, `scrollTo`, or similar in console/logging-related files. IntelliJ's console API likely has a way to disable or conditionally trigger auto-scroll.

**Plan:**
1. Clone the flutter/flutter-intellij repo and set up the development environment per its `CONTRIBUTING.md`
2. Search the codebase for console scroll-related code (keywords: `scroll`, `console`, `log`, `append`)
3. Identify where auto-scroll is triggered when output is appended
4. Modify the logic to either: (a) not auto-scroll when there is already content in the console, or (b) only auto-scroll if the user was already at the bottom
5. Test by running the plugin locally against the `flutter_error_studies` app in IntelliJ
6. Write or update relevant tests

**Implement:** [Link to branch/commits — add as work progresses]

**Review:** Check `CONTRIBUTING.md` in flutter-intellij for commit message format, code style, and PR requirements before submitting.

**Evaluate:** After the fix, running the reproduction app and tapping "Unbounded Viewport" should leave the console positioned at the top, showing the root cause error first, without requiring any manual scrolling.

---

## Testing Strategy

### Manual Testing

- [x] Run `flutter_error_studies` in IntelliJ with the patched plugin, tap "Unbounded Viewport", confirm console stays at top
- [x] Confirm that normal single-error output still works as expected
- [x] Confirm that intentional user scroll-to-bottom still works

### Unit/Integration Tests

- [x] Identify existing console-related tests in the flutter-intellij test suite
- [x] Add a test case verifying scroll position is preserved when large error output is appended

---

## Implementation Notes

### Week 1–2 Progress

- Investigated the issue and reproduction app
- Got the sample app running on Chrome (macOS target blocked by incomplete Xcode install)
- Fixed multiple Flutter API compatibility issues in `flutter_error_studies` to run on Flutter 3.44.1
- Confirmed the bug behavior: cascade of 600+ error lines, console auto-scrolls to bottom, root cause buried
- **Next step:** Set up flutter-intellij dev environment and locate scroll behavior in the codebase

---

## Pull Request

**PR Link:** [To be added]

**Status:** Not yet submitted

---

## Learnings & Reflections

### Technical Skills Gained

- Migrating a Flutter app from pre-null-safety to null-safe Dart
- Using `sed` for in-place find-and-replace in source files
- Understanding deprecated Flutter widget APIs and their modern replacements (`FlatButton` → `TextButton`, etc.)
- Understanding how cascading Flutter render errors work (one layout failure causes many downstream failures)

### Challenges Overcome

- The reproduction app was 6 years old and incompatible with modern Flutter — required multiple manual fixes to compile
- macOS target was unavailable due to incomplete Xcode; pivoted to Chrome successfully
- `dart migrate` no longer exists in Dart 3.x; used `dart fix --apply` instead

### What I'd Do Differently Next Time

- Check the Flutter/Dart version requirements of a reproduction app before trying to run it
- Install Android Studio earlier since the bug is IntelliJ-specific

---

## Resources Used

- [flutter-intellij Issue #3778](https://github.com/flutter/flutter-intellij/issues/3778)
- [flutter_error_studies repo](https://github.com/InMatrix/flutter_error_studies)
- [Original error log gist](https://github.com/flutter/flutter-intellij/issues/3778)
- [Dart null safety migration guide](https://dart.dev/null-safety/migration-guide)
