# reduce-cognitive-complexity

A cross-agent AI coding skill for refactoring code to lower **Cognitive
Complexity** while preserving behavior. It focuses on making control flow
easier to read—not merely reducing branch counts or satisfying a metric by
introducing indirection.

It works in any agent that reads `SKILL.md` files (Anthropic skill spec),
including **opencode**, **Claude Code**, **Antigravity**, and agents that
follow the `.agents/skills/` standard. It has no agent-specific tools or
runtime dependencies.

## What it does

- Identifies deep nesting, long functions, and tangled conditions.
- Applies guard clauses, named predicates, loop-body extraction, sealed-type
  polymorphism, and focused function extraction.
- Preserves behavior and asks for characterization tests when coverage is
  missing.
- Re-measures complexity with the project's existing linter or analyzer.

## When to use it

Use it when a linter reports a Cognitive Complexity issue, such as
SonarQube's cognitive-complexity rules or detekt's
`CognitiveComplexMethod`, `NestedBlockDepth`, or `ComplexCondition` rules.
It is also useful when a function is difficult to understand because it
contains deep nesting, many boolean conditions, or several unrelated
responsibilities.

Do not use it to restructure code that is already simple, or to reduce a
score at the expense of readability. The goal is faster human comprehension,
not a lower number in isolation.

## Install

### One command

Using the [skills CLI](https://github.com/antfu/skills-cli) (or
`vercel-labs/skills`):

```bash
# opencode
npx skills add remithzu/reduce-cognitive-complexity -a opencode

# Claude Code
npx skills add remithzu/reduce-cognitive-complexity -a claude-code

# Antigravity
npx skills add remithzu/reduce-cognitive-complexity -a antigravity
```

Add `-g` for a global install; omit it for a project-scoped install. For
Claude Code, prefer project-scoped installation because globally installed
skills in `~/.agents/skills/` may not be discovered by its Skill tool.

### Manual copy

Copy `skills/reduce-cognitive-complexity/` into the skill directory used by
your agent:

| Agent | Project | Global |
|---|---|---|
| opencode | `.opencode/skills/` | `~/.config/opencode/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Antigravity | `.agents/skills/` | `~/.gemini/config/skills/` |
| AGENTS.md-standard agents | `.agents/skills/` | `~/.agents/skills/` |

```bash
cp -r skills/reduce-cognitive-complexity .agents/skills/
```

## Process

1. Identify the target function and the existing behavior to preserve.
2. Add a characterization test first when coverage is missing.
3. Apply one suitable refactoring technique at a time.
4. Run the project's existing tests after each meaningful step.
5. Re-run the project's existing linter or analyzer and confirm that
   complexity actually dropped.

See [`skills/reduce-cognitive-complexity/SKILL.md`](skills/reduce-cognitive-complexity/SKILL.md)
for the complete guidance, examples, and anti-patterns.

## Repository layout

```text
skills/
└── reduce-cognitive-complexity/
    └── SKILL.md
```

## License

MIT — see [LICENSE](LICENSE).
