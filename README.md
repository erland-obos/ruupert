> [!IMPORTANT]
> This repository is intended to be used as a set of Claude Code instruction files. It is not a standalone application.
> The instruction files in `claude` (including all subfolders) are only supported for Claude Code and should not be
> used to configure other AI agents or models.

# Claude Code Instruction Repository

## Purpose

This repository provides standardized instructions, best practices, and configuration templates for Claude Code. It
enables consistent, high-quality development workflows when using Claude Code as your AI assistant.

## How to use

### Option 1: Add to an existing project

Copy the instruction files to your project root to configure Claude Code for your project.

```bash
# Copy the Claude Code control files
cp -r claude /path/to/your/project/

# Optional: Copy CLAUDE.md to project root for easier access
cp claude/typescript-node/CLAUDE.md /path/to/your/project/CLAUDE.md
```

### Option 2: Start a new project

Clone this repository to start a fresh project with Claude Code instructions pre-configured:

```bash
git clone https://github.com/erland-obos/ruupert.git my-project
cd my-project
rm -rf .git       # Remove existing git history
git init          # Initialize new git repository
```

Then instruct Claude Code to read the control file:

```text
Read claude/typescript-node/CLAUDE.md to understand the project context and instructions.
```

Or, if copied to the project root:

```text
Read CLAUDE.md to understand the project context and instructions.
```

A slightly fuller, more detailed example (will use more of Claude Code's context memory):

```text
Load CLAUDE.md as the primary instruction file, and also load any referenced instruction files.
Follow CLAUDE.md unless conflicts arise, then ask for clarification.
```

**Note:** These example prompts are specifically for Claude Code. The instruction files in `claude` are not supported
by other AI agents or models.

## Project-Specific Overrides

Projects may define custom rules that extend or override the base instructions in `claude`. These are documented in
`claude/PROJECT-OVERRIDES.md`.

This file allows projects to:

- Adapt instructions to specific architectural constraints
- Override base rules when justified by project requirements
- Document project-specific conventions and exceptions

Overrides are intentionally minimal and only introduced when actual development constraints require them. See
`claude/PROJECT-OVERRIDES.md` for guidance on when and how to define project-specific rules.

## Using AI to Improve Instructions for Your Project

To evolve your instruction files systematically:

1. **Audit current state**: Review project structure, tech stack, and actual development patterns
2. **Evidence-based analysis**: Prompt Claude Code to identify gaps between instructions and implementation
3. **Preserve intent**: Ensure changes maintain original principles while adapting to project realities
4. **Validate with stakeholders**: Get senior developer input on proposed modifications
5. **Justify changes**: Apply only updates backed by concrete project requirements or constraints
6. **Document rationale**: Record what changed, why, and reference original rules in `claude/PROJECT-OVERRIDES.md`

### Example prompt:

```text
Analyze the current project structure and development patterns in this codebase, then identify gaps between the Claude Code instructions (in README.md and claude/ directory) and the actual implementation. For each gap found:

1. Document the specific discrepancy with file references (path:line_number format)
2. Determine if the gap represents:
   - Instructions that are outdated/misaligned with current code
   - Implementation that deviates from intended patterns
   - Missing guidance for existing patterns
3. For outdated instructions: propose concrete updates that preserve the original principles while reflecting current project realities
4. For implementation gaps: suggest whether code should align with instructions or vice versa
5. Provide justification for each proposed change, citing specific project requirements or constraints

Present findings in priority order, focusing on discrepancies that most impact development consistency. Format recommendations as specific instruction updates that could be added to claude/PROJECT-OVERRIDES.md with clear rationale.
```

## What's Inside

### Core Claude Code Control

- **claude/typescript-node/CLAUDE.md**: Main instruction file defining Claude Code's behavior, operational rules, and
  communication protocols for TypeScript/Node.js projects

### Development Instructions (`claude/`)

**Common Guidelines**

- Research methodology and documentation standards
- Technology evaluation and API integration workflows

**TypeScript/Node.js Specific**

- Coding standards (type safety, error handling, async patterns)
- Testing practices (TDD workflow, mocking, coverage requirements)
- Docker setup (development and production containerization)

## Structure

```
.
└── claude/
    ├── common/
    │   ├── CLAUDE.md         # Primary Claude Code control file for general development guidelines
    │   └── instructions/
    │       ├── coding.md     # coding guidelines for general development guidelines
    │       ├── testing.md    # TDD and testing guidelines for general development guidelines
    │       ├── docker.md     # Container setup guide general development guidelines
    │       └── research.md       # General research methodology
    └── typescript-node/      # TypeScript/Node.js best practices
        ├── CLAUDE.md         # Primary Claude Code control file for TypeScript/Node.js
        └── instructions/
            ├── coding.md     # coding guidelines for TypeScript/Node.
            ├── testing.md    # TDD and testing guidelines for TypeScript/Node.js
            └── docker.md     # Container setup guide for TypeScript/Node.
```

## Key Principles

These principles guide Claude Code's behavior when using these instructions:

- **Accuracy over assumptions**: Request clarification when requirements are unclear
- **Safety first**: Minimal, necessary changes with explicit user approval
- **Project awareness**: Use existing code patterns as the source of truth
- **Test-driven development**: Write tests first, maintain high coverage
- **Clear documentation**: Structured, actionable guidance

## Usage

These instructions guide Claude Code to:

- Read and understand codebases correctly
- Make safe, precise modifications
- Follow established patterns and conventions
- Ask questions when requirements are ambiguous
- Generate well-tested, production-ready code
- Research technologies systematically
- Document decisions and rationale

## Application

Claude Code applies instructions in layers, from most general to most specific:

1. **Claude Code control file** (claude/typescript-node/CLAUDE.md): Primary instructions for TypeScript/Node.js
   projects
2. **Category-specific** (claude/common): Apply when relevant (research, testing)
3. **Stack-specific** (claude/typescript-node/instructions): Apply for TypeScript/Node.js projects
4. **User directives**: Override everything when specified

The repository provides technology-agnostic and extensible instruction modules for Claude Code.
