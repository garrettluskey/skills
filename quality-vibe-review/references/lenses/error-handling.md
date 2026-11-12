# Error handling

**Question:** Does the change fail simply and visibly, with try/catch only where it belongs?

## Rules

- **Only the top level needs try/catch.** The top level is the boundary where a failure can be handled or reported: a request or command entry point, a message or event handler, a thread or task loop, a plugin or game callback. Trace callers to confirm whether a method is one. Flag try/catch in inner or helper code that could let the exception propagate to that boundary.
- **No nested try/catch.** Flag a try inside another try or catch, and a try in a method that's only ever called from inside another try.
- **More than two is too many.** Flag a try with more than two catch blocks, or a method with more than two try/catch statements. Each one adds a path the reader has to follow.

## Also look for

- **Swallowed failures.** Empty catches, catch-log-continue that hides a real failure, and returning `null` or a default on error so the caller can't tell anything went wrong.
- **Broad catches below the top level.** `catch (Exception)` or equivalent in inner code.
- **Exceptions as control flow.** Throwing and catching to steer normal logic.
- **Lost stack traces.** `throw ex;` in C#, or rethrowing a new exception without the original as its cause.
- **Pointless wrapping.** Custom exceptions that add no information.
- **Catch for cleanup.** Cleanup done in a catch instead of `using`, `finally`, or `defer`.
- **Defensive noise.** Null checks or guards for states the caller already rules out.

## Acceptable

- A catch at the top level that logs, reports, or translates the error
- A catch of one specific, expected exception that's handled meaningfully, e.g. a retry or a fallback with a real reason

## Not this category

- Guard clauses for readability go to Readability.
