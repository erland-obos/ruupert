# Research Instructions

This document defines how **Claude Code** should conduct research during the early and ongoing phases of software development projects. It covers how to gather information, evaluate technologies, investigate APIs, validate findings, document results, and communicate with stakeholders.

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should conduct research when assisting with any project. They are not designed or supported for use with other AI agents or models.

---

## Research Philosophy

### Research-Driven Development

Every research task should follow this workflow:

1. **Define the research question**  
   Clearly state what you need to find out.

2. **Gather information**  
   Use multiple authoritative sources.

3. **Document findings**  
   Capture sources, URLs, quotes, examples, diagrams, and reasoning.

4. **Verify & validate**  
   Cross-check information from at least two independent sources.

5. **Synthesize**  
   Combine validated findings into clear conclusions.

6. **Recommend**  
   Present actionable options or decisions with rationale.

---

## Research Principles

- **Thoroughness**: Research deeply, not superficially.
- **Verification**: Never rely on a single unverified source.
- **Iteration**: Expect multiple research cycles.
- **Documentation**: Record what was found, where, and why.
- **No assumptions**: Clarify with stakeholders when uncertain.
- **No hallucinations**: Only state verified facts.

---

## Web Search and Information Gathering

### When to Perform External Research

Use web search when:

- Exploring unfamiliar technologies, patterns, APIs, or libraries
- Investigating system design approaches or industry best practices
- Checking security requirements, standards, or compliance
- Understanding performance characteristics
- Locating official documentation, RFCs, or specifications
- Validating integration patterns or protocols
- Checking changelogs, version compatibility, or deprecated features

### Search Strategy

**Recommended:**

- Search official documentation first
- Use technical and specific terms
- Include version numbers and release years
- Look for real-world examples and reference implementations
- Verify findings across multiple reputable sources

**Avoid:**

- Using outdated blog posts without verification
- Trusting unverified Q&A, tutorials, or examples
- Relying on a single community answer
- Summaries without checking primary sources

### Source Priority

1. **Tier 1: Official Sources**
    - Official docs and specifications
    - Standards (RFCs, ISO, W3C)
    - Maintainer-written guides

2. **Tier 2: Verified Reputable Sources**
    - Accepted answers on Stack Overflow
    - Maintainer-confirmed GitHub issues/discussions
    - Well-known technical authors

3. **Tier 3: General Community Content**
    - Blog posts and tutorials
    - Forum threads
    - Videos, social media, talks

Always verify Tier 2 and Tier 3 using Tier 1.

---

## API Research

### API Discovery Checklist

When researching an API, determine:

1. **Documentation & API reference**
2. **Authentication requirements**
3. **Endpoints and route structure**
4. **Request/response schemas**
5. **Pagination style**
6. **Rate limits and quotas**
7. **Error formats and retry expectations**
8. **Sandbox/testing environment availability**
9. **Known issues or limitations**

### API Research Template

```markdown
# [API Name] Research

## Official Resources

- Docs:
- API Reference:
- Developer Portal:
- GitHub:
- Status Page:

## Authentication

- Method:
- How to obtain credentials:
- Security notes:

## Base URLs

- Production:
- Sandbox:

## Rate Limits

- Limit:
- Headers:
- Backoff strategy:

## Endpoints

### [GET /path]

- Purpose:
- Params:
- Auth:
- Pagination:
- Response structure:
- Notes:
```

## Data Schema

```typescript
interface Example {
}
```

### Error Handling

- Format:
- Frequent error codes:
- Recovery strategies:

### Considerations

- Edge cases:
- Limitations:
- Integration implications:

### Sources

1.
2.
3.

---

## Technology Research

### Evaluation Criteria

Evaluate a technology according to:

- **Technical capabilities**
- **Compatibility with chosen stack**
- **Performance characteristics**
- **Security posture**
- **Community support and maintenance**
- **Licensing**
- **Integration effort**
- **Operational overhead**
- **Long-term viability**

### Technology Research Template

```markdown
# [Technology Name] Research

## Overview

- Purpose:
- Primary use case:
- Docs:
- Repository:

## Version Information

- Current version:
- Release date:
- Maintenance activity:

## Technical Details

- Compatibility:
- Dependencies:
- Type system support:
- Performance considerations:

## Pros
-

## Cons
-

## Alternatives

- [Alternative A]
- [Alternative B]

## Security

- Known issues:
- Audit results:
- Policy link:

## Community

- Stars:
- Contributors:
- Recent activity:
```

## Minimal Example

```ts
// ...
```

### Recommendation

- Use / Avoid / Needs more research

### Sources

1.
2.

---

## Database Research

When researching database options or schema approaches:

### Key Questions

- What does the data look like?
- What relationships exist?
- What read/write patterns are expected?
- What consistency model is required?
- What performance constraints exist?
- What are the operational constraints (cost, availability, complexity)?

### Schema Research Checklist

- Identify all entities and attributes
- Map relationships
- Understand domain invariants
- Research indexing strategies
- Review normalization vs. denormalization trade-offs
- Investigate migration tooling and strategies

---

## When to Ask for User Input

Always request clarification when:

- Requirements are ambiguous
- Security decisions depend on organizational policy
- Credentials or access details are needed
- Multiple paths exist with distinct trade-offs
- Research exposes conflicting information
- Unknown business rules influence decisions

### Good Question Examples

**Good:**
> “I found two valid pagination strategies: cursor-based and offset-based. Do you have a preference based on system
> requirements?”

**Good:**
> “The documentation shows multiple authentication flows. Which one aligns with available credentials?”

---

## Iterative Research

Research should evolve as new knowledge emerges.

### Iterate When

- New sources reveal contradictions
- Requirements shift
- Implementation reveals hidden complexity
- Documentation is unclear
- Additional questions arise

### Iteration Process

1. Summarize current understanding
2. Identify gaps
3. Refine the research question
4. Conduct targeted investigation
5. Update documentation
6. Validate findings

---

## Verification & Reasoning

Before drawing conclusions:

- Validate against at least two sources
- Confirm compatibility with known constraints
- Test with small proofs-of-concept where possible
- Check for version-specific differences
- Identify performance or security caveats
- Ask whether conclusions still hold in edge cases

Checklist:

- [ ] Confirmed sources
- [ ] Version-specific accuracy
- [ ] No assumptions
- [ ] No hallucinated features
- [ ] Clear rationale
- [ ] All sources cited

---

## Documentation Standards

### Research Deliverables

Every research task must include:

1. **Full Research Document**
    - Findings
    - Sources with URLs
    - Examples, diagrams, schemas
    - Trade-offs
    - Risks

2. **Short Summary for Stakeholders**
    - Key findings
    - Recommendations
    - Any open questions

3. **Implementation Notes (if relevant)**
    - Code stubs
    - Config requirements
    - Dependencies
    - Testing considerations

### Source Citation Format

## Sources

1. **[Title]**
    - URL:
    - Date Accessed:
    - Key Insight:

⸻

Research Workflows

Workflow: New API Integration

1. Discover and evaluate docs
2. Understand auth, endpoints, schemas
3. Test with sample requests
4. Identify edge cases
5. Document response structures and pitfalls
6. Provide integration recommendations
7. Outline implementation steps

Workflow: Technology Evaluation

1. Define requirements
2. Research primary candidate
3. Research alternatives
4. Evaluate trade-offs
5. Create minimal proof-of-concept if needed
6. Recommend approach

Workflow: Architecture Exploration

1. Clarify the architectural problem
2. Research relevant patterns
3. Identify trade-offs and constraints
4. Present 2–4 viable approaches
5. Recommend and justify
6. Capture decision rationale

---

## Anti-Patterns to Avoid

- Do not assume requirements
- Do not state unverified claims
- Do not rely on outdated or single-sourced information
- Do not ignore security considerations
- Do not skip documentation
- Do not hide uncertainties

---

## Research Quality Checklist

Before completing a research task:

### Completeness

- All required areas investigated
- Alternatives considered
- Edge cases identified

### Accuracy

- Facts verified
- Versions current
- No assumptions

### Clarity

- Structured, readable output
- Clear recommendations

### Documentation

- All sources included
- Dates recorded

### Communication

- Open questions listed
- Next steps defined

---

## Summary

These general research instructions provide a standardised process for Claude Code when:

- deciding what to research
- conducting research thoroughly
- evaluating APIs and technologies
- documenting findings clearly
- communicating effectively
- avoiding assumptions and inaccuracies

This file serves as one of the core instruction documents for Claude Code on any project, referenced from CLAUDE.md alongside
architecture, coding, testing, and configuration guidelines.
