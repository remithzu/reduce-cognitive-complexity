---
name: reduce-cognitive-complexity
description: Refactor code to lower Cognitive Complexity — flatten nesting, extract guard clauses, split long functions, name complex conditions — without changing behavior
license: MIT
compatibility: universal
metadata:
  author: remithzu <remithzu@hotmail.com>
  version: "1.1.0"
  workflow: coding-and-refactoring
---

# Reduce Cognitive Complexity

## Purpose

Write and refactor code that is easy to understand, maintain, and review by proactively controlling Cognitive Complexity.

This skill applies to both:

- **New code** — prevent unnecessary complexity before it is introduced.
- **Existing code** — reduce complexity when modifying, refactoring, or fixing code.

The goal is not to achieve the lowest possible complexity score.

The goal is:

> **Make the code easier for a human to understand while avoiding unnecessary SonarQube and detekt complexity warnings.**

Do not optimize for the metric at the expense of readability.

---

## When to use me

Use this skill whenever the user asks to:

- Create a new class.
- Create a new function or method.
- Implement a new feature.
- Add logic to an existing class.
- Modify existing code.
- Refactor code.
- Fix SonarQube complexity warnings.
- Fix detekt complexity warnings.
- Fix `CognitiveComplexMethod`.
- Fix `NestedBlockDepth`.
- Fix `ComplexCondition`.
- Fix equivalent complexity or readability problems.
- Simplify deeply nested logic.
- Split a large function or class.
- Improve code maintainability.

This skill should be applied **proactively** when writing new code.

Do not wait for SonarQube or detekt to report a problem.

---

# Core Principles

## 1. Prevent complexity before fixing it

When creating new code, consider Cognitive Complexity during design and implementation.

Do not first write highly nested code and then attempt to reduce its complexity.

Prefer a structure that is naturally:

- Flat.
- Focused.
- Explicit.
- Easy to scan.
- Easy to test.
- Easy to modify.

Example:

```kotlin
fun processPayment(request: PaymentRequest) {
    if (!request.isValid) return
    if (!isSupported(request.type)) return
    if (!hasEnoughBalance(request)) return

    executePayment(request)
}
```

Prefer this over:

```kotlin
fun processPayment(request: PaymentRequest) {
    if (request.isValid) {
        if (isSupported(request.type)) {
            if (hasEnoughBalance(request)) {
                executePayment(request)
            }
        }
    }
}
```

The first version communicates the flow more directly and avoids unnecessary nesting.

---

# 2. Flatten nesting

Deep nesting is one of the primary causes of Cognitive Complexity.

Prefer:

```kotlin
if (!condition) return
if (!otherCondition) return

performAction()
```

over:

```kotlin
if (condition) {
    if (otherCondition) {
        performAction()
    }
}
```

When several conditions represent independent prerequisites, use guard clauses.

Avoid nesting code merely because it is syntactically possible.

---

# 3. Prefer guard clauses

Use early returns when they make the main execution path easier to understand.

Example:

```kotlin
fun handleAction(action: Action) {
    if (action !is Action.Submit) return
    if (!state.isValid) return
    if (state.isLoading) return

    submit()
}
```

Instead of:

```kotlin
fun handleAction(action: Action) {
    if (action is Action.Submit) {
        if (state.isValid) {
            if (!state.isLoading) {
                submit()
            }
        }
    }
}
```

Guard clauses are especially useful for:

- Validation.
- Null checks.
- State checks.
- Permission checks.
- Type checks.
- Preconditions.
- Error conditions.

Do not introduce guard clauses when they make the business flow harder to understand.

---

# 4. Keep functions focused

A function should have one clear responsibility.

Be suspicious of functions that effectively do:

```text
validate + transform + persist + notify + navigate
```

Prefer separating meaningful responsibilities:

```kotlin
fun saveTransaction(transaction: Transaction) {
    validateTransaction(transaction)
    val data = transformTransaction(transaction)
    repository.save(data)
}
```

Further extraction is appropriate when the extracted operation has a meaningful name and responsibility.

Do not split code solely to manipulate the complexity score.

---

# 5. Name complex conditions

Complex boolean expressions should communicate intent.

Avoid:

```kotlin
if (
    crash.packageName == targetPackage &&
    (crash.exceptionType.contains("OutOfMemory") ||
        crash.exceptionType.contains("StackOverflow")) &&
    crash.timestamp > cutoff
) {
    flagAsCritical(crash)
}
```

Prefer:

```kotlin
if (isCriticalCrash(crash, targetPackage, cutoff)) {
    flagAsCritical(crash)
}
```

with:

```kotlin
private fun isCriticalCrash(
    crash: CrashEntry,
    targetPackage: String,
    cutoff: Long,
): Boolean {
    val isTargetApp = crash.packageName == targetPackage
    val isMemoryOrStackIssue =
        crash.exceptionType.contains("OutOfMemory") ||
            crash.exceptionType.contains("StackOverflow")
    val isRecent = crash.timestamp > cutoff

    return isTargetApp && isMemoryOrStackIssue && isRecent
}
```

Use names that explain **why** the condition matters, not merely **what** operators it contains.

Good:

```kotlin
val isEligibleForRetry = ...
```

Less useful:

```kotlin
val condition = ...
```

---

# 6. Avoid mixed boolean expressions when they obscure intent

Be careful with expressions containing combinations of:

```kotlin
&&
||
!
```

For example:

```kotlin
if (isActive && isAuthenticated || isAdmin && !isSuspended) {
    ...
}
```

If the business meaning is not immediately obvious, extract meaningful predicates:

```kotlin
val canAccess = isRegularUserAllowed || isAdminAllowed

if (canAccess) {
    ...
}
```

or:

```kotlin
if (canAccessFeature()) {
    ...
}
```

Do not combine conditions simply to reduce the number of branches.

Clarity is more important than minimizing operators.

---

# 7. Keep nesting shallow

Avoid unnecessary nesting from:

- `if`
- `when`
- loops
- `try/catch`
- lambdas
- scope functions
- callbacks
- nested functions
- nested collections operations

Kotlin scope functions can hide complexity.

For example, avoid turning straightforward logic into deeply nested:

```kotlin
user?.let {
    repository.find(it.id)?.let { result ->
        result.data?.let { data ->
            if (data.isValid) {
                ...
            }
        }
    }
}
```

Prefer a flatter structure where appropriate:

```kotlin
val user = user ?: return
val result = repository.find(user.id) ?: return
val data = result.data ?: return
if (!data.isValid) return

process(data)
```

Use idiomatic Kotlin, but do not use scope functions merely because they reduce visible lines of code.

---

# 8. Use `when` appropriately

Prefer `when` when handling distinct states or types.

For example:

```kotlin
when (state) {
    is UiState.Loading -> showLoading()
    is UiState.Success -> showContent(state.data)
    is UiState.Error -> showError(state.message)
}
```

A `when` over a sealed hierarchy can make state handling clearer than a long chain of type checks.

Avoid replacing every `if` with `when` simply to reduce a metric.

Choose the construct that best communicates intent.

---

# 9. Prefer sealed types for mutually exclusive states

When a domain concept has a known set of states, consider a sealed hierarchy.

Instead of:

```kotlin
if (state == 0) {
    ...
} else if (state == 1) {
    ...
} else if (state == 2) {
    ...
}
```

Prefer:

```kotlin
sealed interface PaymentState {
    data object Idle : PaymentState
    data object Processing : PaymentState
    data class Failed(val message: String) : PaymentState
    data object Success : PaymentState
}
```

Then:

```kotlin
when (state) {
    PaymentState.Idle -> ...
    PaymentState.Processing -> ...
    is PaymentState.Failed -> ...
    PaymentState.Success -> ...
}
```

Use this when the domain naturally represents mutually exclusive states.

Do not introduce sealed classes only to reduce a complexity score.

---

# 10. Extract loop bodies when appropriate

Loops containing multiple levels of logic can quickly become difficult to read.

Instead of:

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

Prefer:

```kotlin
while (reader.readLine().also { line = it } != null) {
    val current = line ?: continue
    handleLogLine(current)
}
```

with meaningful extraction:

```kotlin
private fun handleLogLine(line: String) {
    if (line.contains("Process:") && buffer.isNotEmpty()) {
        flushEntry(buffer.toString())
        buffer.clear()
    }

    buffer.appendLine(line)
}
```

The loop should remain easy to scan.

---

# 11. Avoid unnecessary nesting in exception handling

Exception handling can become complex when business logic is deeply nested inside `try/catch`.

Avoid:

```kotlin
try {
    if (condition) {
        if (otherCondition) {
            repository.save()
        }
    }
} catch (exception: Exception) {
    ...
}
```

Prefer:

```kotlin
if (!condition) return
if (!otherCondition) return

try {
    repository.save()
} catch (exception: Exception) {
    handleError(exception)
}
```

Keep the `try` block focused when possible.

Do not catch broad exceptions unless the surrounding architecture requires it.

---

# 12. Do not hide complexity

Do not "solve" Cognitive Complexity by moving complicated code somewhere else without improving its readability.

Avoid:

```kotlin
private fun a() = ...
private fun b() = ...
private fun c() = ...
private fun d() = ...
```

when every function contains one trivial line and the reader must jump between many locations.

Extraction is useful when it gives the code a meaningful abstraction.

The objective is:

> Reduce mental effort, not merely reduce the number reported by a tool.

---

# 13. Avoid over-fragmentation

Do not extract every `if`, expression, or line into a function.

Bad:

```kotlin
if (isUserValid()) {
    saveUser()
}

private fun isUserValid() = user != null
```

if the extraction adds no meaningful abstraction.

Prefer:

```kotlin
if (user == null) return

saveUser()
```

unless the condition has domain meaning that deserves a name.

---

# New Code Mode

When creating new classes or functions, apply the following process.

## Step 1 — Identify responsibilities

Before implementing the class/function, determine:

- What is its primary responsibility?
- What inputs does it receive?
- What output does it produce?
- What validation is required?
- What states or branches exist?
- Which operations can be separated?

Avoid putting unrelated responsibilities into one function.

---

## Step 2 — Design the happy path

Make the primary execution path easy to identify.

Prefer:

```kotlin
fun execute(request: Request) {
    validate(request)
    if (!isAllowed(request)) return

    val result = process(request)

    save(result)
}
```

rather than nesting the entire business flow inside multiple conditions.

---

## Step 3 — Handle exceptional paths early

Handle:

- Invalid input.
- Null values.
- Unsupported states.
- Permission failures.
- Error states.
- Preconditions.

as early as possible when doing so improves readability.

---

## Step 4 — Review complexity before finishing

Before considering new code complete, inspect for:

- Deeply nested `if`.
- Long `when`.
- Multiple `else if`.
- Mixed `&&` and `||`.
- Nested loops.
- Nested `try/catch`.
- Nested lambdas.
- Large functions.
- Functions doing multiple responsibilities.
- Repeated conditional logic.

Refactor when complexity is unnecessarily high.

---

# Existing Code / Refactoring Mode

When modifying existing code, first understand the current behavior.

## Step 1 — Identify the target

Determine whether the issue is:

- SonarQube Cognitive Complexity.
- detekt `CognitiveComplexMethod`.
- detekt `NestedBlockDepth`.
- detekt `ComplexCondition`.
- Another equivalent warning.
- General readability or maintainability.

---

## Step 2 — Preserve behavior

Unless the user explicitly requests a behavior change:

> **The refactor must preserve existing behavior.**

Do not silently fix unrelated bugs.

If a bug is discovered during refactoring:

1. Mention it.
2. Keep the behavior unchanged.
3. Propose a separate fix if appropriate.

---

## Step 3 — Establish a safety net

If tests already exist:

- Understand relevant tests.
- Run or update them as necessary.

If there are no tests and the function is sufficiently important or behavior-sensitive, consider creating a characterization test before structural changes.

The test should capture existing behavior rather than redefine it.

---

## Step 4 — Refactor incrementally

Prefer small structural changes:

1. Flatten nesting.
2. Extract meaningful conditions.
3. Extract meaningful responsibilities.
4. Simplify type/state handling.
5. Reassess the resulting structure.

Avoid performing a large unrelated rewrite.

---

## Step 5 — Verify

After refactoring:

- Run relevant unit tests.
- Run relevant static analysis when available.
- Re-check SonarQube/detekt findings.
- Confirm that the intended complexity warning is reduced or resolved.

Do not claim that a warning is fixed if it has not been verified.

---

# Kotlin-Specific Guidance

When working with Kotlin, pay particular attention to complexity hidden inside idiomatic constructs.

## Prefer

```kotlin
val value = input ?: return
```

over unnecessary nesting:

```kotlin
if (input != null) {
    ...
}
```

when the surrounding logic supports an early return.

Prefer:

```kotlin
when (result) {
    is Success -> handleSuccess(result)
    is Error -> handleError(result)
}
```

when states are mutually exclusive.

Prefer named functions for meaningful domain decisions:

```kotlin
if (shouldRefreshSession(session)) {
    refreshSession()
}
```

instead of embedding complicated business rules directly inside UI or domain code.

---

# Android / Compose Guidance

For Android and Jetpack Compose code, complexity can accumulate quickly in:

- ViewModels.
- UseCases.
- Repositories.
- Event handlers.
- Navigation handlers.
- Composable functions.
- State reducers.
- UI state mapping.
- Permission handling.
- WebView configuration.
- Callback handlers.

Do not allow a single function to simultaneously:

```text
read state
validate state
perform business logic
transform data
update UI state
emit events
navigate
handle errors
```

Split meaningful responsibilities.

For Compose specifically, avoid a large composable containing all conditional UI logic.

Prefer:

```kotlin
@Composable
fun Screen(state: ScreenUiState) {
    when (state) {
        is ScreenUiState.Loading -> LoadingContent()
        is ScreenUiState.Error -> ErrorContent(state)
        is ScreenUiState.Content -> Content(state)
    }
}
```

and extract substantial UI sections into meaningful composables when that improves readability.

Do not blindly extract every small UI element.

---

# SonarQube and detekt Awareness

This skill should consider both the human-readable structure and static-analysis rules.

Relevant examples include:

```text
Cognitive Complexity
CognitiveComplexMethod
NestedBlockDepth
ComplexCondition
LongMethod
LargeClass
```

The exact thresholds are project-dependent.

Do not assume a universal threshold.

If project configuration is available, follow the project's configured thresholds.

If it is not available, focus on avoiding clearly excessive complexity rather than inventing a project-specific score.

---

# Complexity Reduction Priority

When reducing complexity, generally prefer the following order:

1. **Flatten unnecessary nesting.**
2. **Use guard clauses.**
3. **Separate unrelated responsibilities.**
4. **Name meaningful complex conditions.**
5. **Simplify state/type branching.**
6. **Extract meaningful loop or callback logic.**
7. **Use sealed types or polymorphism when the domain supports it.**
8. **Simplify expressions that obscure intent.**

Do not start by aggressively extracting functions.

First determine whether the underlying control flow can be made simpler.

---

# Verification Checklist

Before finishing work, ask:

## For new code

- Is the main execution path easy to identify?
- Is nesting reasonably shallow?
- Are guard clauses appropriate?
- Are complex conditions named?
- Does each function have a clear responsibility?
- Are state branches represented clearly?
- Is there unnecessary `if/else` nesting?
- Is Kotlin scope-function nesting making the code harder to read?
- Would SonarQube/detekt likely flag obvious complexity?
- Did I introduce abstractions only where they improve readability?

## For existing code

- Did I understand the existing behavior first?
- Did I preserve behavior?
- Did I avoid unrelated changes?
- Did I reduce unnecessary nesting?
- Did I simplify complex conditions?
- Did I extract meaningful responsibilities?
- Did I avoid over-fragmentation?
- Did I run relevant tests?
- Did I re-run static analysis when available?
- Did the targeted warning actually improve?

---

# What NOT to do

## Do not optimize only for the score

Bad:

```kotlin
private fun condition1() = ...
private fun condition2() = ...
private fun condition3() = ...
```

when the extraction only exists to reduce Cognitive Complexity.

The resulting code may technically have a lower score while being harder to navigate.

---

## Do not change behavior silently

A complexity refactor is not an excuse to change:

- Business rules.
- Error handling.
- Ordering.
- Side effects.
- State transitions.
- Navigation.
- API behavior.

Unless the user explicitly asks for those changes.

---

## Do not combine unrelated conditions

Do not transform readable logic into:

```kotlin
if (a && b || c && d || e && !f) {
    ...
}
```

just because it reduces visible branches.

If the conditions represent different concepts, preserve those concepts.

---

## Do not overuse guard clauses

Guard clauses are useful, but ten sequential returns may be harder to understand than a small structured block.

Choose the structure that best communicates the business flow.

---

## Do not replace everything with `when`

`when` is useful for state and type dispatch.

It is not automatically better than `if`.

Use the construct that makes the intent clearest.

---

## Do not introduce unnecessary architecture

Do not create:

- New interfaces.
- New classes.
- New abstractions.
- New layers.
- New design patterns.

only to reduce Cognitive Complexity.

Architecture should solve an actual structural problem.

---

## Do not hide complexity behind functions

This is not sufficient:

```kotlin
fun process() {
    step1()
    step2()
    step3()
}
```

if each extracted function merely contains another large, difficult-to-understand block.

Complexity should be **distributed according to responsibility**, not merely moved.

---

## Do not refactor unrelated code

When fixing a complexity warning, keep the change focused.

Avoid mixing:

- Formatting changes.
- Naming refactors.
- Architecture changes.
- Dependency upgrades.
- Bug fixes.
- Feature changes.

unless they are necessary for the requested work.

---

# Definition of Done

Code is considered complete when:

### New code

- The implementation is designed with low Cognitive Complexity from the beginning.
- Control flow is easy to follow.
- Nesting is kept shallow.
- Responsibilities are appropriately separated.
- Complex conditions communicate their intent.
- No unnecessary abstraction was introduced.
- The code is unlikely to produce avoidable SonarQube/detekt complexity warnings.

### Existing code

- The original behavior is preserved unless change was explicitly requested.
- The complexity problem has been addressed structurally.
- The code remains readable.
- The code is not over-fragmented.
- Relevant tests pass.
- Static analysis is re-run when available.
- The targeted warning is confirmed to be resolved or reduced.

---

# Guiding Principle

Always optimize for this hierarchy:

```text
Human readability
       ↓
Clear responsibilities
       ↓
Simple control flow
       ↓
Maintainability
       ↓
Static-analysis compliance
```

Never reverse the hierarchy into:

```text
SonarQube score
       ↓
Artificial extraction
       ↓
More indirection
       ↓
Harder code to understand
```

The purpose of this skill is not merely to make the complexity number smaller.

The purpose is to **prevent complex code from being created and to make existing complex code simpler without sacrificing readability or behavior.**