---
name: reduce-cognitive-complexity
description: Analyze and refactor code to reduce Cognitive Complexity according to the SonarSource model while preserving behavior, improving human understandability, and verifying that complexity is actually reduced rather than merely relocated
priority: high
scope:
  - Cognitive Complexity
  - SonarQube
  - SonarLint
  - detekt complexity findings
  - Kotlin
  - Java
  - control-flow refactoring
  - readability refactoring
license: MIT
compatibility: universal
metadata:
  author: remithzu <remithzu@hotmail.com>
  version: "2.3.0"
  workflow: coding-and-refactoring
  source_model: "SonarSource Cognitive Complexity v1.7, 29 August 2023"
---

# Reduce Cognitive Complexity

## Purpose

This skill exists to prevent and reduce Cognitive Complexity while preserving behavior and improving human understandability.

It is a **contract-driven refactoring skill**.

The agent must not treat Cognitive Complexity as a number to manipulate. The metric is a signal that a function may require more mental effort to understand.

The objective is:

> Reduce unnecessary mental effort by improving control flow, nesting, responsibility boundaries, and domain clarity without introducing artificial indirection or changing behavior.

This skill is based on the SonarSource *Cognitive Complexity: A new way of measuring understandability*, Version 1.7, 29 August 2023.

---

# Skill Contract

This section is the operational contract of this skill.

The sections below define:

- when the skill must be used,
- when it must not be used,
- what the agent must do,
- what the agent must produce,
- how the result must be verified,
- and when the agent must stop instead of forcing a refactor.

The agent must follow this contract before applying the detailed guidance.

---

## Trigger

Use this skill when **any** of the following is true:

### Explicit complexity problem

- SonarQube reports `Cognitive Complexity of functions should not be too high`.
- SonarLint reports a Cognitive Complexity issue.
- detekt reports `CognitiveComplexMethod`.
- detekt reports `NestedBlockDepth`.
- detekt reports `ComplexCondition`.
- Another static analyzer reports excessive control-flow complexity.

### Explicit refactoring request

Use when the user asks to:

- reduce Cognitive Complexity,
- reduce nesting,
- simplify a complex function,
- refactor a large function,
- make control flow easier to understand,
- improve readability of complex logic,
- split a function because it is difficult to understand.

### Preventive use

Use when writing new code if the implementation would otherwise create:

- unnecessary deep nesting,
- substantial control flow inside lambdas/callbacks,
- multiple unrelated responsibilities,
- complex boolean decision trees,
- nested loops with significant internal branching,
- deeply nested exception handling,
- difficult state/type dispatch.

Prevent complexity rather than deliberately introducing it and fixing it later.

---

## Do Not Use When

Do not activate this skill solely because:

- a function is long but its control flow is simple,
- a class has many methods,
- there are many lines of declarative code,
- a function contains many method calls,
- code could theoretically be shorter,
- a user asks for formatting only,
- a user asks for a behavior change unrelated to complexity,
- a metric is low and there is no meaningful readability problem.

Do not refactor code merely to make a numeric metric smaller.

Do not assume that every `if`, `when`, lambda, or boolean expression must be extracted.

Do not use this skill as justification for an architectural rewrite.

---

# Required Actions

When the skill is triggered for an existing function, the agent MUST perform these actions in order.

## 1. Identify the target

Determine:

- file,
- function/method,
- rule,
- analyzer,
- reported location,
- current complexity score if available,
- configured threshold if available.

If the analyzer provides a precise finding, use it.

Do not invent a score or threshold.

---

## 2. Read the complete target function

Do not refactor only the highlighted line.

Read enough surrounding code to understand:

- parameters,
- return value,
- state,
- side effects,
- exception behavior,
- asynchronous boundaries,
- callbacks,
- ordering,
- external calls,
- data transformations,
- persistence,
- user-visible effects.

A warning at line N may be caused by nesting established much earlier.

---

## 3. Map the control flow

Identify every relevant flow-breaking construct.

At minimum inspect for:

```text
if
else if
else
ternary
when / switch
for / foreach
while / do while
catch
binary logical-operator sequences
recursion
labeled or multi-level jumps
nested methods
lambdas
callbacks
scope-function blocks
async/coroutine blocks
```

Create a mental or written nesting tree when the function is non-trivial.

Example:

```text
sync()
└── when
    └── if
        └── launch
            └── try
                ├── if
                └── catch
```

---

## 4. Find the deepest complexity path

Do not focus only on the number of branches.

Find the path that requires the reader to retain the most surrounding context.

Examples:

```text
when → if → lambda → try → if
```

```text
loop → if → loop → if → catch
```

```text
callback → when → if → if
```

Prioritize unnecessary nesting on these paths.

---

## 5. Identify responsibility boundaries

Determine what the function is actually doing.

Typical responsibilities include:

```text
validation
decision
state preparation
data loading
transformation
persistence
success handling
failure handling
cleanup
navigation
async execution
UI state update
domain calculation
```

Ask:

> Can this group of operations be described by one meaningful responsibility and name?

If yes, it is a candidate for extraction.

---

## 6. Select the smallest structural intervention

Choose the least invasive strategy that genuinely improves understandability.

Preferred order:

1. Flatten unnecessary nesting.
2. Add appropriate guard clauses.
3. Name meaningful domain conditions.
4. Separate unrelated responsibilities.
5. Extract substantial loop/callback/lambda bodies.
6. Simplify state/type branching.
7. Consider moving genuinely misplaced responsibility.
8. Reassess the whole result.

Do not automatically perform all eight steps.

---

## 7. Preserve behavior

Unless the user explicitly requests behavior changes:

> The refactor MUST preserve existing behavior.

Preserve:

- business rules,
- ordering,
- side effects,
- state transitions,
- exception behavior,
- return behavior,
- persistence behavior,
- coroutine behavior,
- dispatcher/threading behavior,
- navigation,
- API behavior.

If a separate bug is discovered:

1. identify it,
2. do not silently fix it,
3. keep behavior unchanged,
4. mention the separate issue if relevant.

---

## 8. Inspect extracted code

After extraction, inspect:

- the original function,
- every extracted function,
- the caller/callee relationship,
- whether complexity merely moved,
- whether the new names communicate intent,
- whether the code became harder to navigate.

Do not consider the refactor successful simply because the original function became shorter.

---

## 9. Verify

When static analysis is available:

1. run relevant tests,
2. run the relevant static-analysis check,
3. inspect the resulting finding,
4. confirm the targeted warning improved or disappeared.

Do not claim a Cognitive Complexity warning is fixed without verification unless execution is genuinely unavailable.

If verification cannot be performed, explicitly report the result as **unverified**.

---

# Output Contract

When this skill is applied to a complexity finding or refactoring request, the final work report should contain, when applicable:

```text
## Complexity Analysis

- Rule:
- Analyzer:
- Function:
- Current score:
- Threshold:
- Main contributors:
- Deepest nesting path:

## Refactoring Decision

- Strategy:
- Responsibility boundary:
- Why this boundary was chosen:
- Behavior preserved:

## Verification

- Tests:
- Static analysis:
- Targeted warning:
- Result:

## Remaining Risk

- Unverified behavior:
- Remaining complexity:
- Follow-up required:
```

Do not fabricate unavailable values.

If a score or threshold is unknown, write:

```text
Unknown / not available
```

rather than guessing.

---

# Failure / Escalation

Do not force a refactor when:

- the complete target function cannot be inspected,
- existing behavior cannot reasonably be established,
- the requested numeric target is unknown and the user demands a specific score,
- static analysis is unavailable when proof of resolution is explicitly required,
- the only apparent solution is artificial function extraction,
- complexity would merely move into another opaque helper,
- the remaining complexity represents one coherent and understandable algorithm,
- solving the issue requires a large architectural change outside the requested scope,
- a behavior change is required but was not requested.

When escalation is required, explain:

1. what is blocking the refactor,
2. what has been established,
3. what can safely be changed,
4. what additional information or decision is required.

---

# SonarSource Cognitive Complexity Context

## Source

Primary metric reference:

> SonarSource, *Cognitive Complexity: A new way of measuring understandability*, Version 1.7, 29 August 2023.

The source describes Cognitive Complexity as a metric intended to better reflect the relative difficulty of understanding and maintaining code than traditional mathematical control-flow measures.

The source is language-neutral and uses object-oriented terminology such as "class" and "method" for convenience.

---

# Three Fundamental Rules

SonarSource defines three basic rules.

## Rule 1 — Ignore readable shorthand

Structures that allow multiple statements to be expressed clearly in a concise form should not automatically be penalized.

The purpose is to encourage readable language features rather than discourage them.

Method calls are a major example.

A well-named method can summarize a complex operation:

```kotlin
validateRequest()
calculateTotals()
persistOrder()
```

The call itself does not add Cognitive Complexity merely because it invokes another method.

---

## Rule 2 — Increment for breaks in linear flow

Cognitive Complexity increases when code breaks the normal linear reading flow.

Examples include:

```text
conditionals
loops
switch/when
catch
logical operator sequences
recursion
jumps
```

The reader must stop following a simple top-to-bottom path and reason about alternative paths.

---

## Rule 3 — Increment for nested flow-breaking structures

Nesting increases mental effort.

A sequence such as:

```text
if
if
loop
```

is harder to reason about when each structure is nested inside the previous one.

The important principle is:

> A deeply nested flow path can be significantly harder to understand than the same number of flow breaks arranged linearly.

---

# Increment Types

The SonarSource model describes four categories.

## Nesting

Additional mental cost caused by nested control flow.

## Structural

A flow-breaking structure that also increases nesting.

## Fundamental

A flow-breaking structure that does not itself increase nesting.

## Hybrid

A structure that increases nesting but is not itself subject to a nesting increment.

The categories help explain the calculation, but each applicable increment contributes to the final score.

---

# Cognitive Complexity Specification

The following is the operational summary of the SonarSource specification.

## B1 — Structures receiving an increment

An increment applies to:

- `if`
- `else if`
- `else`
- ternary operator
- `switch`
- `for`
- `foreach`
- `while`
- `do while`
- `catch`
- `goto LABEL`
- labeled `break`
- labeled `continue`
- multi-level `break`
- multi-level `continue`
- sequences of binary logical operators
- each method in a recursion cycle

The exact syntax varies by language.

---

## B2 — Structures that increase nesting level

The nesting level is increased by:

- `if`
- `else if`
- `else`
- ternary operator
- `switch`
- `for`
- `foreach`
- `while`
- `do while`
- `catch`
- nested methods
- method-like structures such as lambdas

This distinction is critical.

A structure can influence nesting even when it does not itself receive a structural increment.

---

## B3 — Structures receiving nesting increments

The following structures receive a nesting increment according to their depth inside nesting structures:

- `if`
- ternary operator
- `switch`
- `for`
- `foreach`
- `while`
- `do while`
- `catch`

This is why:

```text
if
└── for
    └── if
```

is substantially more expensive than three independent flow breaks.

---

# Important Exceptions and Compensating Principles

## Method structure

A normal method/function declaration does not itself add Cognitive Complexity.

This supports extracting meaningful responsibilities.

However:

> Method extraction is useful only when it improves understandability.

Do not split code into meaningless helpers merely to manipulate the score.

---

## `try`

The SonarSource specification does not assign a structural increment to `try`.

Do not remove `try` merely because it is visible in a complex function.

---

## `finally`

The SonarSource specification does not assign a structural increment to `finally`.

Do not restructure `finally` solely for metric reasons.

---

## `catch`

Each `catch` clause contributes one structural increment, regardless of the number of exception types caught by that clause.

A `catch` also participates in nesting behavior when nested inside applicable structures.

---

## `else if`

`else if` receives a hybrid treatment.

The mental cost of the additional condition is recognized, but it is not treated as an additional nesting level in the same way as a nested `if`.

Do not mechanically rewrite every `else if`.

---

## `else`

`else` receives a hybrid increment but does not receive the same nesting increment as a new nested `if`.

Do not assume that every `else` creates another nesting level.

---

## `switch`

SonarSource treats a switch and its cases as one structural increment.

This differs from Cyclomatic Complexity's treatment.

A clear `when`/`switch` over one discriminating value can therefore be preferable to an equivalent chain of unrelated comparisons.

Do not replace a clear `when` simply to manipulate the metric.

---

# Logical Operator Sequences

Cognitive Complexity does not increment once for every binary logical operator.

Instead, it assesses sequences of binary logical operators.

Examples:

```kotlin
a && b
```

```kotlin
a && b && c && d
```

are treated as a sequence rather than four independent increments.

More difficult expressions can contain multiple sequences:

```kotlin
a && b || c && d
```

The change between operator sequences increases cognitive effort.

## Refactoring rule

Do not mechanically extract every boolean expression.

Instead ask:

1. Is the expression understandable?
2. Does it represent a domain decision?
3. Are operators mixed?
4. Would a named predicate explain the business meaning?
5. Would extraction reduce mental effort or merely add indirection?

Prefer:

```kotlin
if (canRefreshSession(session)) {
    refreshSession()
}
```

when `canRefreshSession` communicates a meaningful domain decision.

---

# Recursion

Cognitive Complexity adds a fundamental increment for each method participating in a recursion cycle.

This includes indirect recursion.

Do not remove legitimate recursion merely because it contributes to the metric.

If recursion is combined with substantial branching or nesting, evaluate whether the algorithm can be expressed more clearly.

---

# Jumps and Early Exits

Cognitive Complexity assigns an increment to:

- `goto`,
- labeled `break`,
- labeled `continue`,
- other multi-level jumps.

An ordinary early return does not receive the same increment.

This is why guard clauses can be a useful readability technique.

Prefer:

```kotlin
fun execute(request: Request) {
    if (!request.isValid) return
    if (!isSupported(request)) return

    executeRequest(request)
}
```

when the conditions are independent prerequisites.

---

# Nested Methods and Lambdas

This is particularly important for Kotlin.

SonarSource specifies that nested methods and method-like structures such as lambdas can increase nesting level even though the method-like structure itself does not receive a structural increment.

Examples:

```kotlin
scope.launch {
    ...
}
```

```kotlin
withContext(Dispatchers.IO) {
    ...
}
```

```kotlin
items.forEach {
    ...
}
```

```kotlin
callback {
    ...
}
```

Do not interpret this as "lambdas are bad".

The rule is:

> Substantial flow control inside nested method-like structures deserves special attention.

---

# Kotlin Async Example

Consider:

```kotlin
fun sync() {
    when (val decision = plan()) {
        is Ready -> {
            if (isValid()) {
                scope.launch {
                    try {
                        val result = fetch()

                        if (result.isEmpty()) {
                            handleEmpty()
                        } else {
                            handleSuccess(result)
                        }
                    } catch (e: Exception) {
                        handleError(e)
                    }
                }
            }
        }
        is Skipped -> handleSkipped()
    }
}
```

The important nesting path is approximately:

```text
when
└── if
    └── lambda
        └── catch
        └── if
```

Do not simply extract the `if`.

Instead identify the responsibilities:

```text
decide whether synchronization should occur
validate prerequisites
start asynchronous work
execute synchronization
handle result
handle failure
finalize synchronization
```

A meaningful decomposition may become:

```kotlin
fun sync() {
    when (val decision = plan()) {
        is Ready -> startSync(decision)
        is Skipped -> handleSkipped()
    }
}

private fun startSync(decision: Ready) {
    if (!isValid()) return

    scope.launch {
        executeSync(decision)
    }
}

private suspend fun executeSync(decision: Ready) {
    try {
        val result = fetch()
        handleResult(result)
    } catch (e: Exception) {
        handleError(e)
    }
}
```

The exact design depends on the surrounding code.

The principle is:

> Extract a coherent asynchronous responsibility, not arbitrary lines.

---

# Control-Flow Analysis Method

For a non-trivial function, use this process.

## Pass 1 — Linear flow

Read the function from top to bottom.

Mark where the reader must stop and ask:

```text
Which branch?
Which loop?
Which state?
Which exception?
Which callback?
Which nested context?
```

---

## Pass 2 — Structural increments

Mark:

```text
if
else if
else
when
loop
catch
logical sequence
recursion
jump
```

Do not assume all visible syntax contributes equally.

---

## Pass 3 — Nesting

For each flow-breaking structure, determine its nesting depth.

Example:

```text
if                  depth 0
└── when             depth 1
    └── launch       nesting context
        └── if       deeper context
            └── catch
```

---

## Pass 4 — Responsibility

Group statements into responsibilities.

Example:

```text
A B C → validation
D E    → data loading
F G H  → transformation
I J    → persistence
K L    → error reporting
```

A group with a meaningful responsibility is a potential extraction boundary.

---

## Pass 5 — Refactoring

Choose the smallest intervention that reduces mental effort.

---

## Pass 6 — Re-analysis

After the refactor:

- inspect the caller,
- inspect extracted functions,
- inspect nesting,
- inspect responsibility boundaries,
- compare static-analysis results.

Do not stop after the first extraction.

---

# Refactoring Strategy

## Strategy 1 — Flatten unnecessary nesting

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

Use this when conditions are independent prerequisites.

---

# Strategy 2 — Guard clauses

Good candidates:

- invalid input,
- null values,
- unsupported states,
- permissions,
- preconditions,
- expected failure states.

Example:

```kotlin
fun handleAction(action: Action) {
    if (action !is Action.Submit) return
    if (!state.isValid) return
    if (state.isLoading) return

    submit()
}
```

Do not use guard clauses mechanically.

Ten unrelated early returns can be harder to understand than a small structured block.

---

# Strategy 3 — Separate responsibilities

Be suspicious of functions that do all of:

```text
validate
transform
persist
notify
navigate
handle error
```

Prefer meaningful boundaries:

```kotlin
fun saveTransaction(transaction: Transaction) {
    validateTransaction(transaction)

    val data = transformTransaction(transaction)

    repository.save(data)
}
```

Extraction should represent a real responsibility.

---

# Strategy 4 — Extract meaningful conditions

Prefer:

```kotlin
if (isCriticalCrash(crash)) {
    flagAsCritical(crash)
}
```

when the predicate expresses a meaningful domain decision.

Avoid:

```kotlin
if (condition1()) {
    ...
}
```

when `condition1` exists only to hide one trivial expression.

---

# Strategy 5 — Extract substantial loops

When a loop body contains multiple nested decisions, consider extracting the operation.

Prefer:

```kotlin
while (reader.hasNext()) {
    processEntry(reader.next())
}
```

when `processEntry` represents a coherent responsibility.

Do not extract a one-line operation merely because it is inside a loop.

---

# Strategy 6 — Extract substantial callback/async bodies

This is especially valuable when the body contains:

```text
try/catch
if/else
when
loops
state transitions
multiple side effects
```

Prefer:

```kotlin
scope.launch {
    executeOperation()
}
```

over placing the entire algorithm inside the lambda.

---

# Strategy 7 — Simplify state/type branching

Use a clear `when` when a domain concept naturally represents mutually exclusive states.

Example:

```kotlin
when (state) {
    is UiState.Loading -> showLoading()
    is UiState.Success -> showContent(state)
    is UiState.Error -> showError(state)
}
```

Do not replace a clear `when` with another structure merely to reduce a metric.

---

# Strategy 8 — Use sealed types only when the domain supports them

If a concept naturally has a finite set of mutually exclusive states, a sealed hierarchy can improve readability.

Do not introduce a sealed hierarchy solely because an `if/else if` chain has a high score.

Architecture and domain modeling must have independent justification.

---

# Complexity Redistribution Test

This test is mandatory after meaningful extraction.

Ask:

> Did complexity disappear, or did I only move it?

Example:

```text
Before

largeFunction()
└── complex algorithm

After

largeFunction()
└── complexHelper()
```

If `complexHelper()` is still difficult to understand, the refactor has not necessarily solved the problem.

Inspect:

```text
original function
extracted function
new helper relationships
```

A good refactor distributes complexity according to responsibilities.

A bad refactor distributes complexity according to line count.

---

# Abstraction Quality Test

An extracted function should answer at least one of these:

- Does it represent a meaningful responsibility?
- Does it make the caller easier to scan?
- Does it express useful domain language?
- Does it isolate a substantial algorithm?
- Does it create a useful test boundary?
- Does it remove unnecessary nesting from the caller?

If the answer is no, reconsider the extraction.

---

# Complexity vs Function Length

Do not equate line count with Cognitive Complexity.

A long function can be understandable when it is mostly linear:

```kotlin
val a = loadA()
val b = loadB()
val c = transform(a, b)
save(c)
notify(c)
```

A shorter function can be cognitively expensive:

```kotlin
if (...) {
    when (...) {
        ...
    }
}
```

Therefore:

> Function length is a supporting signal, not the metric itself.

---

# Complexity vs Method Count

Do not create many methods simply because method calls are not counted by Cognitive Complexity.

This is metric gaming.

Good:

```kotlin
validate()
calculate()
persist()
```

when each operation is meaningful.

Bad:

```kotlin
checkA()
checkB()
checkC()
step1()
step2()
step3()
```

when each helper merely hides one line of the original function.

---

# Kotlin Scope Functions

Kotlin scope functions can improve or reduce readability depending on context.

Do not use:

```kotlin
let
run
also
apply
with
```

merely to avoid visible `if` statements.

Avoid excessive nesting such as:

```kotlin
user?.let {
    repository.find(it.id)?.let { result ->
        result.data?.let { data ->
            if (data.isValid) {
                process(data)
            }
        }
    }
}
```

A flatter form can be clearer:

```kotlin
val user = user ?: return
val result = repository.find(user.id) ?: return
val data = result.data ?: return

if (!data.isValid) return

process(data)
```

Choose based on understandability, not line count.

---

# Boolean Conditions

When a condition is complex, first determine whether it contains:

- multiple concepts,
- mixed operators,
- precedence that requires careful reading,
- domain rules,
- repeated predicates.

Then choose between:

```kotlin
val canProcess = ...
```

or:

```kotlin
if (canProcess(request)) {
    ...
}
```

or a structured sequence of guards.

Do not split a clear expression into multiple helpers simply because it contains operators.

---

# Exception Handling

Keep `try` blocks focused when possible.

Prefer:

```kotlin
if (!isEligible()) return

try {
    repository.save()
} catch (e: Exception) {
    handleFailure(e)
}
```

over:

```kotlin
try {
    if (isEligible()) {
        if (isValid()) {
            repository.save()
        }
    }
} catch (e: Exception) {
    handleFailure(e)
}
```

Remember:

```text
try    -> no direct increment
catch  -> structural increment
finally -> no direct increment
```

Do not remove error handling solely for Cognitive Complexity.

---

# New Code Contract

When writing a new function:

## Required

Before considering it complete:

1. Identify its primary responsibility.
2. Identify major decision points.
3. Identify error paths.
4. Identify async/callback boundaries.
5. Avoid unnecessary nesting.
6. Keep the primary path easy to scan.
7. Name meaningful business decisions.
8. Avoid artificial helper extraction.
9. Review the function for likely complexity before finishing.

## Review questions

Ask:

```text
Can I understand the main path in one scan?

Do I need to remember several surrounding conditions?

Is substantial logic hidden inside a lambda?

Are unrelated responsibilities mixed?

Are boolean expressions expressing business rules clearly?

Would a guard clause make an independent prerequisite clearer?

Is a when/switch representing a natural state dispatch?

Did I introduce unnecessary abstraction?
```

---

# Existing Code Contract

When fixing an existing warning:

## Required

1. Establish current behavior.
2. Identify the exact warning.
3. Map the control flow.
4. Identify the deepest nesting.
5. Identify responsibility boundaries.
6. Refactor incrementally.
7. Preserve behavior.
8. Inspect extracted functions.
9. Run relevant tests.
10. Run static analysis when available.
11. Confirm the targeted warning.
12. Report verification accurately.

---

# Verification Contract

Verification has three levels.

## Level 1 — Static inspection

Minimum when execution is unavailable:

```text
- inspect complete function
- inspect extracted functions
- reason about control flow
- identify remaining complexity
```

Result:

```text
UNVERIFIED
```

Do not call this a successful static-analysis fix.

---

## Level 2 — Tests

Run relevant tests when available.

Report exact results.

Example:

```text
Tests:
./gradlew :core:test
PASS
```

---

## Level 3 — Static analysis

When SonarQube, SonarLint, detekt, or equivalent tooling is available, rerun the relevant check.

Report:

```text
Before:
Cognitive Complexity = X

Threshold:
Y

After:
Cognitive Complexity = Z

Result:
Resolved
```

If only the warning disappeared without an exposed score:

```text
Targeted warning:
Resolved

Exact score:
Not available
```

Never invent the score.

---

# Definition of Done

A complexity refactor is complete only when all applicable conditions are satisfied:

- [ ] Target function was fully understood.
- [ ] Actual complexity contributors were identified.
- [ ] Deepest nesting path was considered.
- [ ] Responsibility boundaries were considered.
- [ ] Unnecessary nesting was reduced.
- [ ] Behavior was preserved.
- [ ] Extracted code has meaningful responsibilities.
- [ ] Complexity was not merely relocated.
- [ ] Code was not artificially fragmented.
- [ ] Relevant tests were run when available.
- [ ] Static analysis was rerun when available.
- [ ] The targeted warning was confirmed resolved/reduced, or explicitly marked unverified.
- [ ] No unrelated refactor was introduced.

---

# Anti-Patterns

## Metric gaming

Do not optimize:

```text
score ↓
```

at the expense of:

```text
understandability ↓
```

---

## Function splitting by line count

Do not create helpers merely because a function is long.

---

## Function splitting by nesting syntax

Do not extract every nested `if`.

---

## Helper explosion

Avoid:

```text
step1()
step2()
step3()
step4()
step5()
step6()
```

when the helpers do not represent meaningful concepts.

---

## Hiding a complex algorithm

Avoid:

```text
caller
  ↓
helper
  ↓
helper2
  ↓
helper3
```

when the reader must jump through several trivial layers to understand one operation.

---

## Boolean compression

Do not turn several understandable conditions into one dense expression:

```kotlin
if (a && b || c && d || e && !f) {
    ...
}
```

merely to reduce visible branching.

---

## Scope-function abuse

Do not turn linear logic into nested:

```kotlin
let { }
run { }
also { }
apply { }
```

blocks merely because they are idiomatic Kotlin.

---

## Unnecessary architecture

Do not introduce:

- interfaces,
- repositories,
- use cases,
- managers,
- coordinators,
- new modules,
- design patterns,

only to reduce Cognitive Complexity.

If architecture must change, the reason must be structural and independently defensible.

---

## Silent behavior change

Do not change behavior while claiming a refactor.

---

## Unverified success

Do not say:

> "SonarQube issue is fixed"

unless the relevant analysis confirms it.

---

# Decision Matrix

Use this matrix before choosing a refactoring.

| Problem | Preferred response |
|---|---|
| Independent nested prerequisites | Guard clauses |
| Deep nested branch | Flatten control flow |
| Complex business predicate | Name meaningful predicate |
| Large coherent algorithm | Extract meaningful operation |
| Async lambda with substantial control flow | Extract async responsibility |
| Complex loop body | Extract meaningful loop operation |
| Mutually exclusive domain states | Clear `when` / state model |
| Mixed boolean operators | Clarify domain conditions |
| Complexity only moved to helper | Continue analysis |
| Helper has no meaningful responsibility | Do not extract |
| Warning requires architectural redesign | Escalate / reconsider scope |
| Static analysis unavailable | Mark result unverified |

---

# Example Analysis Template

For a warning on:

```text
Function: syncGitCommits
Rule: Cognitive Complexity
```

produce an analysis such as:

```text
Current structure:

syncGitCommits
└── when
    └── Ready
        └── if
            └── launch
                └── try
                    └── if
                    └── catch

Primary issue:

The problem is not simply the number of conditions.
The main issue is the depth of the control-flow path combined with
substantial asynchronous work inside a lambda.

Responsibility boundaries:

1. Decide whether sync should occur.
2. Validate repository availability.
3. Prepare synchronization state.
4. Execute synchronization.
5. Handle result.
6. Handle failure.
7. Finalize synchronization state.

Preferred direction:

Keep the top-level function focused on orchestration.
Extract the substantial asynchronous synchronization responsibility.
Do not extract arbitrary individual statements.
```

Then verify the result.

---

# SonarSource Model: Practical Reference

Use this table as a quick reference.

| Construct | Structural increment | Nesting behavior |
|---|---:|---|
| `if` | Yes | Increases nesting |
| `else if` | Hybrid | Increases nesting level but no separate nesting increment |
| `else` | Hybrid | Increases nesting level but no separate nesting increment |
| ternary | Yes | Increases nesting |
| `switch` / equivalent | Yes | Increases nesting |
| `for` / `foreach` | Yes | Increases nesting |
| `while` / `do while` | Yes | Increases nesting |
| `catch` | Yes | Increases nesting |
| `try` | No | No |
| `finally` | No | No |
| method call | No | No |
| nested method/lambda | No structural increment | Increases nesting level |
| binary logical-operator sequence | Yes | Depends on surrounding nesting |
| recursion cycle method | Yes | Fundamental increment |
| labeled/multi-level jump | Yes | Fundamental increment |
| ordinary early `return` | No equivalent jump increment | No |

This table is an operational summary of the SonarSource specification. Language-specific analyzers may have implementation details that differ; when an actual analyzer result is available, the analyzer result is authoritative for the final score.

---

# Important Interpretation Rule

Cognitive Complexity is a measure of understandability, not a universal definition of "bad code".

Therefore:

```text
High score
    ↓
Investigate
```

not:

```text
High score
    ↓
Automatically split function
```

Likewise:

```text
Low score
    ↓
Automatically good code
```

is not valid.

The agent must use the metric together with actual code structure and responsibility boundaries.

---

# Final Principle

Optimize in this order:

```text
Human understandability
        ↓
Clear responsibility boundaries
        ↓
Simple control flow
        ↓
Shallow unnecessary nesting
        ↓
Maintainable structure
        ↓
Static-analysis compliance
```

Never reverse this into:

```text
SonarQube number
        ↓
Artificial extraction
        ↓
More indirection
        ↓
Harder code
```

The correct outcome is not:

> "The function has a lower Cognitive Complexity score."

The correct outcome is:

> **"The code requires less mental effort to understand, the responsibilities are clearer, behavior is preserved, and the static-analysis result confirms the intended improvement."**
