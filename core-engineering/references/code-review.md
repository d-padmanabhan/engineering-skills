# Code Review Patterns

## What to Evaluate

### Security
- Input validation, injection risks, secure defaults, principle of least privilege
- Check for: hardcoded secrets, secret exposure in logs, dependency CVEs
- OWASP Top 10: Injection, Authentication, Sensitive Data Exposure, SSRF, XXE, Deserialization

### Error Handling
- Graceful failures, retry mechanisms, meaningful error messages

### Testing
- Testability design, coverage gaps, edge cases

### Observability
- Logging, metrics, debugging support
- **Logging Policy:** Default to `INFO` in prod; enable `DEBUG` behind a flag/env
- **Never** log secrets; mask keys containing `token`, `secret`, `password`, `key`, `authorization`

### Accessibility (a11y)
- Semantic HTML (`<button>` for buttons)
- ARIA attributes for dynamic components
- Keyboard navigability
- Sufficient color contrast

### Resource Management
- Memory leaks, connection cleanup, file handles

### Concurrency
- Thread safety, race conditions, async patterns

### Performance
- Algorithmic efficiency, database queries, caching opportunities

## PR Review Formats

### Standard PR Review
```
Provide minimal, actionable feedback suitable for PR comments.
Focus on critical issues first, frame suggestions constructively,
group related issues, and explain the "why" behind each suggestion.
When possible, include file/line hints or code hunks suitable for
direct PR comments. Provide minimal diffs instead of paraphrase.
```

### Security-Focused Review
```
Prioritize security vulnerabilities by severity: injection risks,
improper data handling, auth/authz issues, common exploits,
hardcoded secrets, logging leaks, dependency CVEs, OWASP Top 10.
```

### Performance Review
```
Analyze production bottlenecks: inefficient algorithms, unnecessary
queries, memory leaks, scalability concerns. Tie each suggestion to
an expected metric (e.g., p95 latency, CPU %, memory, I/O) and
provide a simple measurement plan.
```

### Structural Review
```
Focus on code structure, readability, and maintainability.
Prioritize high-impact changes that don't require major refactoring.
```

## Response Format for Code Reviews

```markdown
## Summary
[Brief overview of findings]

## Critical Issues
[List with code examples, fixes, and file/line references when possible]

## Recommended Improvements
[List with code examples, explanations, and expected impact]

## Optional Enhancements
[List with brief suggestions]
```

Provide minimal diffs:
```diff
- old_code()
+ new_code()
```

## Reviewer Checklist

- [ ] Correct type(scope) in title
- [ ] Summary ≤72 chars, imperative mood
- [ ] Bulleted change list included
- [ ] Rationale paragraph provided if non-obvious change
- [ ] Security reviewed; no secrets exposed
- [ ] Tests updated/added

## Performance Suggestions Format

When suggesting performance improvements:
- **State the expected impact:** e.g., "Reduce p95 latency from X→Y", "Cut memory ~Z%"
- **Provide a measurement plan:** benchmark method/tool; production metric to watch
- **Justify the change:** tie to workload characteristics (hot paths, call frequency, I/O)

Example:
```
Cache database query results
Expected impact: Reduce average response from 200ms to 50ms
Measurement: Compare p95/p99 latency pre/post using app metrics
Justification: Query runs ~1000/min against mostly-static data
```

