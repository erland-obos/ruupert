# AI Agent Instruction Repository

## Purpose

This repository provides standardized instructions, best practices, and configuration templates for AI agents. It
enables consistent, high-quality AI-assisted development workflows.

## How to use

### Option 1: Add to existing project

Copy the instruction files to your project root. You may rename `AGENT.md` to match your specific agent (e.g.,
`CLAUDE.md`, `COPILOT.md`, `CODEX.md`, `CURSOR.md`).

```bash
# Copy the main control file
cp AGENT.md /path/to/your/project/

# Copy the specific instructions
cp -r .agents /path/to/your/project/
```

### Option 2: Start new project

Clone this repository to start a fresh project with instructions pre-configured:

```bash
git clone https://github.com/your-org/ai-instructions.git my-project
cd my-project
rm -rf .git       # Remove existing git history
git init          # Initialize new git repository
```

Then instruct your AI assistant (referencing your specific control file):
> "Read AGENT.md (or `CLAUDE.md`, `COPILOT.md` etc) to understand the project context and instructions."

## What's Inside

### Core Agent Control

- **AGENT.md** (or **COPILOT.md**, etc.): Main instruction file defining agent behavior, operational rules, and
  communication protocols

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
├── AGENT.md                               # Primary agent control file
└── .agents/
    ├── common/instructions/
    │   └── research.md                     # Research methodology
    └── typescript-node/instructions/
        ├── coding.md                       # TypeScript/Node.js best practices
        ├── testing.md                      # TDD and testing guidelines
        └── docker.md                       # Container setup guide
```

## Key Principles

- **Accuracy over assumptions**: Request clarification when requirements are unclear
- **Safety first**: Minimal, necessary changes with explicit user approval
- **Project awareness**: Use existing code patterns as source of truth
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

1. **Global rules** (AGENT.md): Always apply
2. **Category-specific** (.agents/common): Apply when relevant (research, testing)
3. **Stack-specific** (.agents/typescript-node): Apply for TypeScript/Node.js projects
4. **User directives**: Override everything when specified

The repository is technology-agnostic at its core, with extensible stack-specific modules.
