---
name: lessons-writer
description: Documents lessons learned, patterns discovered, and mistakes to avoid in PROJECT_CONTEXT.md (section 10).
---
## Lessons Writer Skill

Capture and organize lessons learned throughout the development process to prevent repeated mistakes and share knowledge.

### When to Use
- After ANY correction from user or reviewer
- When discovering a non-obvious solution
- When encountering a gotcha or edge case
- After resolving a difficult bug
- When a pattern emerges that should be documented
- At the end of each issue/feature completion

### File Location
`PROJECT_CONTEXT.md` (section `## 10. Lessons Learned`)

### Document Structure
```markdown
# Lessons Learned

> This document captures patterns, gotchas, and learnings from development.
> Review this file at the start of each session.

## Table of Contents
- [Architecture Patterns](#architecture-patterns)
- [Common Pitfalls](#common-pitfalls)
- [Testing Insights](#testing-insights)
- [Performance Notes](#performance-notes)
- [Security Considerations](#security-considerations)
- [Tool-Specific Tips](#tool-specific-tips)

---

## Architecture Patterns

### [Pattern Name]
**Context:** <when this pattern applies>
**Solution:** <the pattern/approach>
**Example:**
```code
<code example>
```
**Learned from:** Issue #<num>

---

## Common Pitfalls

### [Pitfall Title]
**Symptom:** <what goes wrong>
**Cause:** <why it happens>
**Prevention:** <how to avoid it>
**Learned from:** Issue #<num>

---

## Testing Insights

### [Insight Title]
**Discovery:** <what was learned>
**Application:** <how to apply it>
**Learned from:** Issue #<num>

---

## Performance Notes

### [Note Title]
**Issue:** <performance problem>
**Solution:** <optimization applied>
**Impact:** <measurable improvement>
**Learned from:** Issue #<num>

---

## Security Considerations

### [Security Note]
**Vulnerability:** <type of security issue>
**Mitigation:** <how it was addressed>
**Prevention:** <how to prevent in future>
**Learned from:** Issue #<num>

---

## Tool-Specific Tips

### [Tool/Library Name]
**Tip:** <the insight>
**Context:** <when it applies>
**Learned from:** Issue #<num>

---

*Last updated: <timestamp>*
```

### Adding a Lesson

#### From User Correction
When the user corrects an approach:
1. Acknowledge the correction
2. Understand the root cause
3. Write a lesson entry:
```markdown
### Avoid <What Was Wrong>
**Symptom:** <what I did wrong>
**Cause:** <why I did it that way>
**Prevention:** <the correct approach going forward>
**Learned from:** User correction on Issue #<num>
```

#### From Bug Discovery
When finding a non-obvious bug:
```markdown
### <Bug Type> in <Area>
**Symptom:** <how it manifested>
**Cause:** <root cause>
**Prevention:** <how to catch it earlier>
**Learned from:** Issue #<num>
```

#### From Successful Pattern
When something works particularly well:
```markdown
### <Pattern Name>
**Context:** <when to use this>
**Solution:** <the approach>
**Example:**
```code
<minimal working example>
```
**Learned from:** Issue #<num>
```

### Categorization Rules
- **Architecture Patterns**: Structural decisions, design patterns
- **Common Pitfalls**: Mistakes to avoid, gotchas
- **Testing Insights**: Testing strategies, mock approaches
- **Performance Notes**: Optimizations, profiling discoveries
- **Security Considerations**: Security fixes, vulnerability patterns
- **Tool-Specific Tips**: Library quirks, API behaviors

### Writing Guidelines
1. **Be specific**: Include code examples when possible
2. **Be actionable**: Focus on prevention/application
3. **Be concise**: One paragraph max per section
4. **Reference source**: Always include issue number
5. **Date entries**: Add timestamp for context

### Review Protocol
At session start, agents should:
1. Read `PROJECT_CONTEXT.md (section ## 10. Lessons Learned)`
2. Note any lessons relevant to current work
3. Apply documented patterns proactively

### Output Format
After adding a lesson:
```
## Lesson Recorded

**Category:** <category>
**Title:** <lesson title>
**Added to:** PROJECT_CONTEXT.md (section ## 10. Lessons Learned)

Summary: <one-line summary of what was learned>
```

### Integration with Other Skills
- `code-reviewer`: Triggers lessons-writer for review findings
- `tester`: Triggers lessons-writer for test failures
- `senior-engineer-executor`: Triggers lessons-writer for corrections
- `security-checker`: Triggers lessons-writer for security issues
