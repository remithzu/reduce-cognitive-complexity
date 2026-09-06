---
name: reduce-cognitive-complexity
description: Refactor code to lower Cognitive Complexity — flatten nesting, extract guard clauses, split long functions, name complex conditions — without changing behavior
license: MIT
compatibility: universal
metadata:
  author: remithzu <remithzu@gmail.com>
  version: "1.0"
---

## What I do
- Identify functions that are hard to follow because of nesting, long
  bodies, or tangled conditions
- Refactor them using a fixed set of techniques (below) that reduce
  Cognitive Complexity specifically — not the same thing as Cyclomatic
  Complexity, see "What this metric actually measures"
- Leave behavior identical — this is a refactor, not a feature change or
  bug fix

## When to use me
Use this when a function is flagged by a linter (SonarQube, detekt's
`CognitiveComplexMethod`/`NestedBlockDepth`/`ComplexCondition` rules, or
equivalent for your language), or when a function is simply hard to hold in
your head — deep nesting, many booleans, or doing several unrelated things
at once.

Don't use this to preemptively restructure code that's already simple and
flat just because a rule exists — see "What NOT to do."

This skill only assumes you can read and edit source files — no
agent-specific tools are required, so it applies the same way in any AI
coding agent.

## What this metric actually measures
Cognitive Complexity (SonarSource's metric) penalizes what actually makes
code hard to *read*, which is different from what Cyclomatic Complexity
counts:
- Each break in linear flow (`if`, `else if`, `else`, ternary, `switch`/
  `when`, loops, `catch`) adds +1
- **Nesting adds an extra penalty on top of that** — an `if` inside an
  `if` inside a loop costs more than three sequential top-level `if`s, even
  though a naive branch-count metric would score them the same
- A sequence of the *same* logical operator (`a && b && c`) counts once;
  mixing `&&`/`||` in one expression adds a penalty
- Straightforward constructs that don't add mental burden (e.g. a plain
  `when` covering a sealed type, a null-coalescing `?:`) aren't penalized
  the way an equivalent `if`/`else if` chain would be

The practical implication: **flattening nesting matters more than reducing
the number of branches.** A function with ten sequential guard clauses is
often lower complexity than one with three deeply nested `if`s.

## Refactoring techniques

### 1. Guard clauses instead of nested conditionals
Before:
```kotlin
fun onToggle(state: DashboardUiState) {
    if (action is DashboardUiAction.ToggleService) {
        if (state.readLogsGranted) {
            if (!state.isServiceRunning) {
                startService()
            } else {
                stopService()
            }
        } else {
            emitEvent(DashboardUiEvent.ShowMessage("Grant READ_LOGS first"))
        }
    }
}
```
After:
```kotlin
fun onToggle(state: DashboardUiState) {
    if (action !is DashboardUiAction.ToggleService) return
    if (!state.readLogsGranted) {
        emitEvent(DashboardUiEvent.ShowMessage("Grant READ_LOGS first"))
        return
    }
    if (state.isServiceRunning) stopService() else startService()
}
```
Same behavior, no nesting penalty — every check is sequential.

### 2. Extract complex conditions into a named function or variable
Before:
```kotlin
if (crash.packageName == targetPackage &&
    (crash.exceptionType.contains("OutOfMemory") || crash.exceptionType.contains("StackOverflow")) &&
    crash.timestamp > cutoff
) {
    flagAsCritical(crash)
}
```
After:
```kotlin
if (isCriticalCrash(crash, targetPackage, cutoff)) flagAsCritical(crash)

private fun isCriticalCrash(crash: CrashEntry, targetPackage: String, cutoff: Long): Boolean {
    val isTargetApp = crash.packageName == targetPackage
    val isMemoryOrStackIssue = crash.exceptionType.contains("OutOfMemory") ||
        crash.exceptionType.contains("StackOverflow")
    val isRecent = crash.timestamp > cutoff
    return isTargetApp && isMemoryOrStackIssue && isRecent
}
```
The mixed `&&`/`||` penalty and the mental effort of parsing the expression
both move into a well-named function; the call site now reads as intent,
not logic.

### 3. Extract a loop's body when it contains a nested conditional
Before:
```kotlin
while (reader.readLine().also { line = it } != null) {
    val current = line ?: continue
    if (current.contains("Process:")) {
        if (buffer.isNotEmpty()) {
            flushEntry(buffer.toString())
            buffer.clear()
        }
    }
    buffer.appendLine(current)
}
```
After:
```kotlin
while (reader.readLine().also { line = it } != null) {
    val current = line ?: continue
    handleLogLine(current)
}

private fun handleLogLine(line: String) {
    if (line.contains("Process:") && buffer.isNotEmpty()) {
        flushEntry(buffer.toString())
        buffer.clear()
    }
    buffer.appendLine(line)
}
```
Collapsing the nested `if` into one combined condition removes a nesting
level, and moving the loop body into a named function keeps the loop
itself trivial to scan.

### 4. Replace type-checking chains with polymorphism / sealed types
A `when` over a `sealed interface`/`sealed class` that covers every case is
not penalized the way an equivalent `if (x is A) ... else if (x is B) ...`
chain is. If you find yourself writing that chain, check whether the type
should be sealed instead.

### 5. Split functions that do several unrelated things
If a function's name needs "and" to describe it ("validates and saves and
notifies"), its complexity is likely additive across three concerns. Split
along those boundaries — each resulting function should be describable
without "and."

## Process
1. Identify the target — a linter finding, or a function that's genuinely
   hard to follow.
2. If no tests cover the function, write a characterization test first
   (capture current behavior) before touching the structure — a refactor
   with no safety net is a behavior change waiting to happen.
3. Apply one technique at a time from the list above; re-run tests after
   each step.
4. Re-measure (re-run the linter) — confirm the complexity actually
   dropped, don't rely on it "feeling" simpler.
5. If the refactor changed function boundaries or file structure in a way
   worth remembering, apply the `change-log` skill; a pure mechanical
   flattening with no structural change doesn't need a log entry.

## What NOT to do
- Don't extract a single trivial one-line condition into its own function
  just to shave a point off the score — that adds indirection without
  adding clarity, which is the opposite of the actual goal
- Don't combine unrelated conditions into one boolean just to reduce branch
  count if it obscures what's actually being checked
- Don't change behavior while refactoring — if you spot an actual bug
  while doing this, note it and fix it as a separate, explicit change, not
  folded silently into the refactor
- Don't over-fragment — ten tiny single-call functions chasing each other
  trades Cognitive Complexity for navigation complexity, which has its own
  readability cost. The goal is a human reading the code faster, not a
  lower number in isolation
