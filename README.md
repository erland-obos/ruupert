# AI Agent Instruction Repository

## Purpose

This repository provides standardized instructions, best practices, and configuration templates for AI agents. It
enables consistent, high-quality AI-assisted development workflows.

## How to use

### Option 1: Add to an existing project

Copy the instruction files to your project root.

```bash
# Copy the agent control files
cp -r .agents /path/to/your/project/

# Optional: Copy CLAUDE.md to project root for easier access
cp .agents/typescript-node/CLAUDE.md /path/to/your/project/CLAUDE.md
```

### Option 2: Start a new project

Clone this repository to start a fresh project with instructions pre-configured:

```bash
git clone https://github.com/your-org/ai-instructions.git my-project
cd my-project
rm -rf .git       # Remove existing git history
git init          # Initialize new git repository
```

Then instruct your AI assistant to read the control file:

```text
Read .agents/typescript-node/CLAUDE.md to understand the project context and instructions.
```

Or, if copied to the project root:

```text
Read CLAUDE.md to understand the project context and instructions.
```

A slightly fuller, more detailed example (will use more of your agent's context memory):

```text
Load CLAUDE.md as the primary instruction file, and also load any referenced instruction files.
Follow CLAUDE.md unless conflicts arise, then ask for clarification.
```

## Project-Specific Overrides

Projects may define custom rules that extend or override the base instructions in `.agents`. These are documented in
`.agents/PROJECT-OVERRIDES.md`.

This file allows projects to:

- Adapt instructions to specific architectural constraints
- Override base rules when justified by project requirements
- Document project-specific conventions and exceptions

Overrides are intentionally minimal and only introduced when actual development constraints require them. See
`.agents/PROJECT-OVERRIDES.md` for guidance on when and how to define project-specific rules.

## Using AI to Improve Instructions for Your Project

To evolve your instruction files systematically:

1. **Audit current state**: Review project structure, tech stack, and actual development patterns
2. **Evidence-based analysis**: Prompt an AI agent to identify gaps between instructions and implementation
3. **Preserve intent**: Ensure changes maintain original principles while adapting to project realities
4. **Validate with stakeholders**: Get senior developer input on proposed modifications
5. **Justify changes**: Apply only updates backed by concrete project requirements or constraints
6. **Document rationale**: Record what changed, why, and reference original rules in `.agents/PROJECT-OVERRIDES.md`

### Example prompt:

```text
Analyze the current project structure and development patterns in this codebase, then identify gaps between the AI agent instructions (in README.md and .agents/ directory) and the actual implementation. For each gap found:

1. Document the specific discrepancy with file references (path:line_number format)
2. Determine if the gap represents:
   - Instructions that are outdated/misaligned with current code
   - Implementation that deviates from intended patterns
   - Missing guidance for existing patterns
3. For outdated instructions: propose concrete updates that preserve the original principles while reflecting current project realities
4. For implementation gaps: suggest whether code should align with instructions or vice versa
5. Provide justification for each proposed change, citing specific project requirements or constraints

Present findings in priority order, focusing on discrepancies that most impact development consistency. Format recommendations as specific instruction updates that could be added to .agents/PROJECT-OVERRIDES.md with clear rationale.
```

## What's Inside

### Core Agent Control

- **.agents/typescript-node/CLAUDE.md**: Main instruction file defining agent behavior, operational rules, and
  communication protocols for TypeScript/Node.js projects

### Development Instructions (`.agents/`)

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
└── .agents/
    ├── typescript-node/      # TypeScript/Node.js best practices
    │   ├── CLAUDE.md         # Primary agent control file for TypeScript/Node.js
    │   └── instructions/
    │       ├── coding.md     
    │       ├── testing.md    # TDD and testing guidelines for TypeScript/Node.js
    │       └── docker.md     # Container setup guide
    └── common/instructions/
        └── research.md       # Research methodology
```

## Key Principles

- **Accuracy over assumptions**: Request clarification when requirements are unclear
- **Safety first**: Minimal, necessary changes with explicit user approval
- **Project awareness**: Use existing code patterns as the source of truth
- **Test-driven development**: Write tests first, maintain high coverage
- **Clear documentation**: Structured, actionable guidance

## Usage

These instructions guide AI agents to:

- Read and understand codebases correctly
- Make safe, precise modifications
- Follow established patterns and conventions
- Ask questions when requirements are ambiguous
- Generate well-tested, production-ready code
- Research technologies systematically
- Document decisions and rationale

## Application

Instructions are layered and context-specific:

1. **Agent control file** (.agents/typescript-node/CLAUDE.md): Primary instructions for TypeScript/Node.js projects
2. **Category-specific** (.agents/common): Apply when relevant (research, testing)
3. **Stack-specific** (.agents/typescript-node/instructions): Apply for TypeScript/Node.js projects
4. **User directives**: Override everything when specified

The repository is technology-agnostic at its core, with extensible stack-specific modules.
