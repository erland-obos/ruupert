# PROJECT-OVERRIDES.md

This document defines how this project may adapt, extend, or override the general Claude Code instructions provided in the
`claude` directory. It is intentionally minimal and assumes a senior audience.

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should behave when assisting with this project. They are not designed or supported for use with other AI agents or models.

## Purpose

The `claude` directory provides the authoritative instruction set for Claude Code when assisting with development. This file describes when and how project-specific rules may supplement or override those instructions.

**Note:** In this project, all `claude` instructions and the overrides defined here are used exclusively to configure and control Claude Code's behavior. Other AI agents or models are not supported by this instruction set.

## Guiding Principles

1. The `claude` instructions are the default authority and must be followed unless this file explicitly states
   otherwise.
2. Project-specific adaptations should be introduced only when required by actual constraints discovered during
   development.
3. Overrides must be narrow, explicit, and justified. Avoid general or speculative rules.
4. Changes to `claude` behaviour must not contradict global safety, research, or testing principles.

## When to Extend or Override

Extensions or overrides may be introduced only after the project is bootstrapped and the following becomes known:

- Project type and stack
- Architectural constraints or external integrations
- Repository structure and conventions
- Testing, tooling, or CI-specific requirements

Until the project reveals these needs, no divergence from `claude` is permitted.

## How to Extend or Override

Use the following rules:

### 1. Keep `claude` as the base layer

All adaptation must reference a specific `claude` file or rule being extended or overridden.

### 2. Document overrides explicitly

Each override must include:

- A short description of the new rule
- The `claude` rule it modifies
- The reason the override is necessary
- Scope and limitations

Example format:

### Override: Example Rule

- Overrides: `claude/typescript-node/instructions/testing.md` section "Test Data Setup".
- Reason: Project uses an external sandbox API that cannot be mocked locally.
- Scope: Integration tests only.
- Limitation: Unit testing rules remain unchanged.

### 3. Prefer additive rules over replacements

Add clarifications or constraints instead of rewriting large instruction sections.

### 4. Avoid project wide abstractions until patterns stabilise

Do not create global project rules early. Add them only after patterns emerge through real development.

#### When Not to Override

**Do not override claude for:**

- Developer preference
- Unverified assumptions about future requirements
- Temporary workarounds
- Speculative architectural choices

#### Change Control

**Any addition or override must:**

- Be reviewed by senior developers
- Be merged only when justified by project evidence
- Be small and deliberate
- Maintain compatibility with the general instruction philosophy

## Future Sections

**This file may be expanded later with:**

- Project-specific coding standards
- Architectural constraints
- Dependency policies
- Testing or CI exceptions
- Documentation structure decisions

... only after these areas become concrete through actual development.

---

## How Claude Code Must Read and Apply This File

Claude Code must treat `claude/PROJECT-OVERRIDES.md` as an execution-time modifier of the `claude` instruction set. The
following rules
apply:

### 1. Load Order

1. Load the relevant `claude` files as the primary instruction set.
2. After loading `claude`, load `claude/PROJECT-OVERRIDES.md`.
3. Apply the rules in this file as explicit, higher-priority overrides.

If a conflict is detected between `claude` and `claude/PROJECT-OVERRIDES.md`, this file takes precedence.

### 2. Interpretation Rules

- Treat every section of `claude/PROJECT-OVERRIDES.md` as normative unless explicitly marked as an example.
- Do not infer additional rules beyond what is written.
- Do not generalize patterns from overrides; apply them only to the described scope.

### 3. When Executing Tasks

When performing any operation (analysis, refactor, code generation, documentation, research):

- First check if the action is fully governed by `claude`.
- If the action touches an area where an override or extension exists in this file, apply the override exactly as
  written.
- If no explicit override exists, fall back to `claude` without introducing assumptions.

### 4. Safety and Boundaries

Claude Code must:

- Avoid reasoning outside the explicit content of this file and `claude`.
- Request clarification from the user if a task requires a new override or if the existing overrides do not
  unambiguously apply.
- Avoid global project-wide reinterpretations unless they are explicitly defined.

### 5. Change Detection

If the project evolves and new folders, tools, or conventions appear:

- Do not modify your behavior automatically.
- Request updated guidance so this file can be amended with new overrides or extensions.
- Treat the absence of updates here as a signal to continue following `claude` strictly.

### 6. Enforcement

This file constrains both autonomous and user-directed Claude Code behavior. When a user requests an operation that conflicts
with an existing override:

- Claude Code must highlight the conflict.
- Claude Code must request confirmation before deviating from the documented rules.

This ensures deterministic, auditable behavior and stable collaboration between developers and Claude Code.
