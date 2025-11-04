# Example Pull Request: JSON Streaming Output

This is a complete example of a well-crafted PR description using our enhanced
template. Use this as a reference when writing your own PRs.

---

## Summary

Add JSON streaming output format (`--output-format stream-json`) for real-time
monitoring of long-running operations. This enables users to track progress in
scripts and automation workflows without waiting for complete operation to
finish.

## Specialty & Purpose

**Problem Space:** This PR addresses the observability gap in headless/scripting
use cases, specifically for long-running automated workflows.

**Unique Value:** Unlike our existing JSON output format which returns a single
response after completion, this adds streaming support for real-time event
monitoring. This is distinct from:

- Recent CLI UX improvements (focused on interactive use)
- JSON output PR (focused on final structured output)
- WebSocket server PR (focused on IDE integration)

**Project Mission Alignment:** This advances our goal of making Gemini CLI the
best choice for developers by ensuring it works seamlessly in automated
workflows, not just interactive sessions. It fills a critical gap for CI/CD
integrations where real-time feedback is essential for debugging and monitoring.

## Logical Reasoning (Logos)

**Technical Constraints:**

- Must work with standard Unix tools (grep, jq, etc.)
- Cannot break existing JSON output format
- Need to handle network interruptions gracefully
- Must support both success and error scenarios

**Alternatives Considered:**

1. **Server-Sent Events (SSE)**
   - ❌ Rejected: Too complex for CLI tools, requires HTTP server
   - Would need additional infrastructure dependencies

2. **Newline-delimited JSON (NDJSON)** ✅ Selected
   - ✅ Zero dependencies, works with standard tools
   - ✅ Proven format used by Docker, Kubernetes
   - ✅ Each line is independently parsable
   - ✅ Simple implementation

3. **Custom binary protocol**
   - ❌ Rejected: Over-engineered for this use case
   - Would require custom parsers in every language

**Performance Impact:**

- Benchmark: 50K events streamed in 2.3s vs 2.8s for buffered (18% faster)
- Memory: Constant memory usage vs O(n) for buffered output
- Network: Events available immediately vs batch at end

**Validation:**

- 15 new integration tests covering streaming scenarios
- Manual testing with GitHub Actions, GitLab CI
- Error handling tested with network interruptions
- Tested with `jq`, `grep`, `awk` for Unix tool compatibility

## Impact & Value (Pathos)

**Who Benefits:**

- **DevOps Engineers:** Running automated tests need real-time feedback to debug
  failures immediately rather than waiting 10+ minutes for completion
- **CI/CD Authors:** Want granular monitoring to optimize pipeline performance
  and catch issues early
- **Script Authors:** Debugging long-running operations no longer requires
  guessing if the script has hung or is still processing

**Pain Points Addressed:**

_Before:_

```bash
$ gemini -p "Run full test suite" --output-format json
# User waits 15 minutes with no feedback...
# If script fails at 14 minutes, all context is lost
```

_After:_

```bash
$ gemini -p "Run full test suite" --output-format stream-json
{"type":"start","timestamp":"2025-11-04T09:00:00Z"}
{"type":"progress","message":"Building project...","percent":10}
{"type":"progress","message":"Running unit tests...","percent":25}
{"type":"error","message":"Test failed: auth.test.ts","timestamp":"2025-11-04T09:02:30Z"}
# User can immediately see where failure occurred and investigate
```

**Vision Alignment:**

Our mission is to be the most developer-friendly AI CLI. This change ensures
we're not just great for interactive coding, but also for the automation
workflows that developers rely on daily. It demonstrates our commitment to
production-grade tooling.

**Future Opportunities:**

- Progress bars for long operations (building on streaming data)
- Parallel operation monitoring (multiple streams)
- Real-time log streaming from sandboxed commands
- Dashboard integrations (stream to monitoring tools)

## Quality & Credibility (Ethos)

**Standards & Best Practices:**

- Follows NDJSON specification (jsonlines.org)
- Conforms to existing CLI output format patterns
- Backward compatible with existing `--output-format json`
- Implements graceful degradation on errors

**Testing Rigor:**

| Test Category         | Coverage |
| --------------------- | -------- |
| Unit tests            | 100%     |
| Integration tests     | 15 new   |
| Cross-platform manual | ✓        |
| Error scenarios       | 8 cases  |

**Platforms Validated:**

- ✅ macOS 13+ (Apple Silicon & Intel)
- ✅ Ubuntu 20.04, 22.04
- ✅ Windows 10, 11
- ✅ Docker (alpine, debian bases)

**Expert Review:**

- Design reviewed with @cli-team-lead on 2025-10-28
- Implementation pair-programmed with @senior-engineer
- Documentation reviewed by @tech-writer
- Security implications reviewed (no sensitive data in streams)

**Code Quality:**

- ESLint: 0 warnings
- TypeScript: Strict mode, no `any` types
- Test coverage: 94% overall, 100% for new code
- Documentation: JSDoc on all public APIs

**Safeguards:**

```typescript
// Graceful fallback on streaming errors
try {
  await streamJsonOutput(results);
} catch (error) {
  console.warn('Streaming failed, falling back to JSON');
  console.log(JSON.stringify(results));
}
```

## Self-Critique & Expansion Opportunities

**Known Limitations:**

1. **Binary Data:** No support for binary data in streams (would require base64
   encoding, increasing size by 33%)
2. **Line Length:** No maximum line length enforced (extremely large events
   could cause issues with some parsers)
3. **Retry Logic:** No built-in retry for streaming failures (user must handle)
4. **Compression:** No compression support (could reduce bandwidth for
   high-volume streams by ~60%)

**Unaddressed Edge Cases:**

- Behavior when stdout/stderr are the same stream (edge case in some shell
  redirections)
- Locale-specific decimal separators in progress percentages (may affect parsing
  in non-US locales)
- Very high frequency events (>1000/sec) not tested (may cause buffering issues)

**Future Enhancements:**

1. **Performance Tuning:**
   - Add `--stream-buffer-size` flag for tuning
   - Support for batch mode (group multiple events per line)

2. **UX Improvements:**
   - Progress bars in streaming mode
   - Color-coded event types
   - Event filtering by type

3. **Advanced Features:**
   - Compression support for high-volume streams (gzip, zstd)
   - WebSocket option for browser-based monitoring
   - Multiplexed streams for parallel operations

**Feedback Welcome:**

- Is the event schema intuitive for automation authors?
- Should we include more contextual metadata in each event (e.g., operation ID)?
- Are there other streaming use cases in CI/CD we should consider?
- Would a `--stream-filter` option to filter events client-side be valuable?

## Related Issues

Closes #12345

Related to #11223 (JSON output format)

## How to Validate

### Basic Streaming Test

```bash
# Install the PR branch
npm install -g @google/gemini-cli@pr-12345

# Test basic streaming
gemini -p "List all files in current directory" --output-format stream-json

# Expected: See newline-delimited JSON events, one per line
# {"type":"start",...}
# {"type":"progress",...}
# {"type":"complete",...}
```

### Unix Tool Integration

```bash
# Test with jq
gemini -p "Analyze this codebase" --output-format stream-json | jq -r '.message'

# Test with grep
gemini -p "Run tests" --output-format stream-json | grep '"type":"error"'
```

### Error Handling

```bash
# Test network interruption (kill network during operation)
gemini -p "Long running task" --output-format stream-json

# Expected: Graceful fallback to standard JSON output with warning
```

### Cross-Platform

Test on:

- [ ] macOS (both Intel and Apple Silicon if available)
- [ ] Linux (Ubuntu 22.04)
- [ ] Windows 11

## Pre-Merge Checklist

- [x] Updated relevant documentation and README (added section to
      docs/cli/output-formats.md)
- [x] Added/updated tests (15 new integration tests)
- [ ] Noted breaking changes (none - backward compatible)
- [x] Validated on required platforms/methods:
  - [x] MacOS
    - [x] npm run
    - [x] npx
    - [x] Docker
    - [x] Podman
    - [ ] Seatbelt (not applicable for output format)
  - [ ] Windows (awaiting Windows CI runner)
    - [x] npm run
    - [x] npx
    - [x] Docker
  - [x] Linux
    - [x] npm run
    - [x] npx
    - [x] Docker
