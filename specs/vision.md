# harden — Vision

**Systematically harden any application against its own runtime errors.**

## The Problem

Applications fail at runtime. Null references, unhandled edge cases, malformed input, unexpected state — these show up in logs as errors, stack traces, and crashes. A developer notices an error, context-switches to reproduce it, writes a fix, and hopes they got it right. Errors accumulate faster than they're addressed. Many are never seen at all.

## The Insight

Runtime logs are a goldmine of real-world failure data. Every `ERROR:` line is a test case the application failed in production. If an agent could read those logs, compare them against a record of what's already been fixed, and systematically write tests and fixes for what's new — the application would get more robust with every pass.

## The Solution

**harden** is an agent skill that reads an application's runtime logs, identifies new errors, and hardens the codebase against them.

The developer points the skill at a log file from their running application:

```
/skill:harden path/to/app.log
```

The skill:

1. Reads the log, looking for `ERROR:` lines with timestamps
2. Reads the **hardening ledger** to know what's already been addressed
3. Identifies errors that are **new** — not yet in the ledger
4. For each new error: analyzes root cause, writes a test that reproduces it, fixes the code
5. Updates the ledger with what was found and what was done

The **hardening ledger** is the memory that makes this work across time. An error from six months ago that's in the ledger won't be re-analyzed. A new error from last night will be.

The skill is agent-agnostic. It follows the [Agent Skills standard](https://agentskills.io) and uses only standard tools (read, write, edit, bash). It works with any coding agent.

## Success Criteria

- Works on any codebase that produces timestamped logs with `ERROR:` prefixed lines
- Finds errors in runtime logs that haven't been addressed before
- Produces real, running tests that reproduce each error
- Produces fixes that make those tests pass
- Maintains a ledger that accumulates project knowledge over months and years
- Never re-analyzes an error that's already been addressed
