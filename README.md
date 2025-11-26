# AI Agent Instruction Repository

## Purpose

This repository provides standardized instructions, best practices, and configuration templates for AI agents. It
enables consistent, high-quality AI-assisted development workflows.

## How to use

### Option 1: Add to existing project

Copy the instruction files to your project root.

```bash
# Copy the agent control files
cp -r .agents /path/to/your/project/

# Optional: Copy CLAUDE.md to project root for easier access
cp .agents/typescript-node/CLAUDE.md /path/to/your/project/CLAUDE.md
```

### Option 2: Start new project

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

Or, if copied to project root:

```text
Read CLAUDE.md to understand the project context and instructions.
```

A slightly fuller, more detailed example (will use more of your agent's context memory):

```text
Load CLAUDE.md as the primary instruction file, and also load any referenced instruction files.
Follow CLAUDE.md unless conflicts arise, then ask for clarification.
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
    ├── typescript-node/
    │   ├── CLAUDE.md                       # Primary agent control file
    │   └── instructions/
    │       ├── coding.md                   # TypeScript/Node.js best practices
    │       ├── testing.md                  # TDD and testing guidelines
    │       └── docker.md                   # Container setup guide
    └── common/instructions/
        └── research.md                     # Research methodology
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

1. **Agent control file** (.agents/typescript-node/CLAUDE.md): Primary instructions for TypeScript/Node.js projects
2. **Category-specific** (.agents/common): Apply when relevant (research, testing)
3. **Stack-specific** (.agents/typescript-node/instructions): Apply for TypeScript/Node.js projects
4. **User directives**: Override everything when specified

The repository is technology-agnostic at its core, with extensible stack-specific modules.
