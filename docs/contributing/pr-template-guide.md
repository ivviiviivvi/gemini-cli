# Pull Request Template Guide

This guide helps you effectively use our enhanced PR template, which
incorporates classical rhetorical elements to create more compelling and
well-reasoned pull requests.

## Overview

Our PR template is designed to help you articulate not just _what_ you changed,
but _why_ it matters and _how_ it demonstrates quality. By incorporating logos
(logic), pathos (emotion/values), and ethos (credibility), you create a complete
picture that helps reviewers understand and appreciate your contribution.

## Section-by-Section Guide

### Summary

Keep this brief and direct. Answer: What changed and why does it matter?

**Example:**

> Add JSON streaming output format for real-time monitoring. This enables users
> to track long-running operations in scripts and automation workflows.

### Specialty & Purpose

This section helps reviewers understand your PR's unique contribution within the
broader context of all project PRs.

**Key Questions:**

- What niche or problem space does this address?
- How is this different from other recent or related PRs?
- What specific gap in functionality or quality does this fill?

**Example:**

> This PR specializes in improving observability for headless/scripting use
> cases. Unlike our existing JSON output format which returns a single response,
> this adds streaming support for real-time event monitoring. This fills a
> critical gap for CI/CD integrations and long-running automated workflows.

### Logical Reasoning (Logos)

Provide the technical foundation and evidence that supports your approach.

**Key Elements:**

- Technical constraints or requirements that shaped the solution
- Data, benchmarks, or metrics that informed decisions
- Alternative approaches considered and why they were rejected
- Testing methodology and validation results

**Example:**

> We evaluated three approaches:
>
> 1. Server-Sent Events (SSE) - Rejected due to complexity for CLI tools
> 2. Newline-delimited JSON (NDJSON) - **Selected** for simplicity and ecosystem
>    support
> 3. Custom binary protocol - Rejected as over-engineered
>
> NDJSON was chosen because:
>
> - Zero dependencies, works with standard Unix tools (grep, jq)
> - Proven format used by Docker, Kubernetes logs
> - Each line is independently parsable for streaming
> - Tested with 50+ integration test scenarios including error conditions

### Impact & Value (Pathos)

Connect with reviewers on a human level by showing who benefits and how.

**Key Elements:**

- User personas who will benefit
- Specific pain points being addressed
- How this aligns with project values and vision
- Future possibilities this enables

**Example:**

> **Who Benefits:**
>
> - DevOps engineers running automated tests who need real-time feedback
> - CI/CD pipeline authors who want granular monitoring
> - Script authors debugging long-running operations
>
> **Pain Points Addressed:**
>
> - Currently users must wait for entire operation to complete before seeing any
>   output
> - No way to know if long-running script has hung vs. still processing
> - Missing structured output makes automation brittle
>
> **Vision Alignment:** This advances our mission of making Gemini CLI the best
> choice for developers by ensuring it works seamlessly in automated workflows,
> not just interactive use.
>
> **Future Opportunities:**
>
> - Progress bars for long operations
> - Parallel operation monitoring
> - Real-time log streaming from sandboxed commands

### Quality & Credibility (Ethos)

Demonstrate that your work meets high standards and can be trusted.

**Key Elements:**

- Standards and best practices followed
- Testing rigor and coverage
- Expert consultation or code review
- Documentation and safeguards
- Accessibility considerations

**Example:**

> **Standards:**
>
> - Follows NDJSON specification (jsonlines.org)
> - Conforms to existing CLI output format patterns
> - Backward compatible with existing JSON output
>
> **Testing:**
>
> - 15 new integration tests covering streaming scenarios
> - Manual testing with real CI/CD pipelines (GitHub Actions, GitLab CI)
> - Error handling tested with network interruptions and malformed responses
> - Cross-platform validation on macOS, Linux, Windows
>
> **Review:**
>
> - Design reviewed with CLI team leads
> - Implementation pair-programmed with senior engineer
> - Documentation reviewed by technical writer
>
> **Safeguards:**
>
> - Graceful fallback to standard JSON on streaming errors
> - Clear error messages for parsing failures
> - Comprehensive JSDoc and usage examples

### Self-Critique & Expansion Opportunities

Show intellectual honesty by acknowledging limitations and inviting feedback.

**Key Elements:**

- Known limitations or trade-offs
- Edge cases not yet handled
- Future enhancements that could build on this
- Specific feedback you're seeking

**Example:**

> **Known Limitations:**
>
> - No support for binary data in streams (would require base64 encoding)
> - Maximum line length not enforced (could cause issues with extremely large
>   events)
> - No built-in retry logic for streaming failures
>
> **Unaddressed Edge Cases:**
>
> - Behavior when stdout/stderr are the same stream needs clarification
> - No handling for locale-specific decimal separators in progress values
>
> **Future Enhancements:**
>
> - Add `--stream-buffer-size` flag for tuning performance
> - Support for progress bars in streaming mode
> - Compression support for high-volume streams
> - WebSocket option for browser-based monitoring
>
> **Feedback Welcome:**
>
> - Is the event schema intuitive for automation authors?
> - Should we include more contextual metadata in each event?
> - Any other streaming use cases we should consider?

## Tips for Success

### Be Specific

Avoid vague statements. Instead of "improves performance," say "reduces startup
time from 2.3s to 0.8s (65% improvement) as measured across 100 runs."

### Use Examples

Concrete examples make abstract concepts tangible. Show before/after code,
command outputs, or user scenarios.

### Balance Depth and Brevity

Each section should be substantive but focused. If a section grows too long,
consider if some details belong in documentation or design docs instead.

### Invite Engagement

Your PR description is the start of a conversation, not a monologue. Ask
questions and welcome feedback.

### Update as You Learn

If reviewers raise good points or if your approach evolves, update your PR
description to reflect the current understanding. This helps future maintainers.

## The Rhetorical Framework

### Logos (Logic)

Appeals to reason and logic. This is your technical foundation—the objective
facts, data, and reasoning that support your solution.

**When to emphasize:** Technical infrastructure, performance optimizations,
security fixes, architectural changes.

### Pathos (Emotion/Values)

Appeals to values, emotions, and shared goals. This connects your work to the
human impact and the project's mission.

**When to emphasize:** User-facing features, accessibility improvements, DX
(developer experience) enhancements, documentation.

### Ethos (Credibility)

Establishes trust and demonstrates quality. This shows why your work should be
believed and relied upon.

**When to emphasize:** Critical bug fixes, security patches, breaking changes,
foundational refactoring.

### Balanced Approach

The best PRs use all three elements in harmony. Technical rigor (logos)

- clear impact (pathos) + demonstrated quality (ethos) = compelling, trustworthy
  contribution.

## Examples from Other Projects

### Example 1: Security Fix

- **Logos:** CVE details, attack vector analysis, security research
- **Pathos:** User data protection, trust in the platform
- **Ethos:** Security audit, penetration testing, expert review

### Example 2: UI Feature

- **Logos:** User research data, A/B test results, accessibility guidelines
- **Pathos:** User frustration addressed, delight factor, inclusive design
- **Ethos:** Design system compliance, usability testing, cross-browser
  validation

### Example 3: Performance Optimization

- **Logos:** Profiling data, algorithmic complexity, benchmark comparisons
- **Pathos:** Reduced wait times, better experience, cost savings
- **Ethos:** Load testing, production monitoring, expert code review

## FAQ

**Q: Do I need to fill out every section for every PR?**

A: Use judgment. Small bug fixes might have brief sections, while major features
should be thorough. The key is providing enough context for reviewers to
understand and evaluate your work.

**Q: This seems like a lot of work. Why bother?**

A: Good PR descriptions save time for everyone. They reduce review cycles, help
future maintainers understand decisions, and create a valuable historical
record. Think of it as documentation written at the perfect time—when context is
fresh.

**Q: What if I don't know the answer to some questions?**

A: That's okay! Be honest about unknowns. "Not yet tested on Windows" or
"Performance impact unclear, would appreciate profiling help" are perfectly
valid. This actually strengthens trust (ethos) by showing intellectual honesty.

**Q: Can I adjust the template for my specific PR?**

A: Yes, but keep the overall structure. You might rename sections or emphasize
different aspects based on your PR's nature. The framework should help, not
hinder.

## Conclusion

This template might seem elaborate, but it reflects our commitment to
thoughtful, well-reasoned contributions. By taking time to articulate the logic,
value, and quality of your work, you help create a culture of excellence that
benefits everyone.

Remember: A great PR description is a gift to your reviewers, your future self,
and everyone who will maintain this code.

## Complete Example

For a complete, real-world example of a well-crafted PR using this template, see
[PR Example: JSON Streaming Output](/docs/contributing/pr-example.md). This
example demonstrates how to fill out each section effectively with concrete
details, evidence, and clear reasoning.
