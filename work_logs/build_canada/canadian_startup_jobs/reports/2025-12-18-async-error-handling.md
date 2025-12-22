# Async Error Handling Refactor

(This report was AI Generated from a coding agent)


**Date:** December 18, 2025
**Type:** Refactor / Tech Debt
**Status:** Complete

## Situation
The application's error handling utility (`logErrorWithContext` in `apps/server/src/lib/errors/handler.ts`) relied on synchronous Node.js file system operations (`readFileSync`, `existsSync`). In a production environment (especially with the single-threaded nature of JavaScript runtimes), synchronous I/O blocks the event loop, potentially causing performance degradation and request queuing during error logging. Additionally, the function suffered from deep nesting ("tower of hell"), making it difficult to read and maintain.

## Task
Refactor the error handling logic to:
1.  Eliminate blocking synchronous I/O operations.
2.  Leverage the native Bun file API for better performance.
3.  Improve code readability by flattening nested conditionals and extracting atomic helper functions.

## Action
We rewrote `handler.ts` with the following changes:
1.  **Async/Await Migration**: Replaced `node:fs` imports with `Bun.file()` async methods (`.exists()` and `.text()`).
2.  **Atomic Extraction**:
    *   Extracted `printErrorHeader` to handle standard error logging.
    *   Extracted `getFileLocationFromLine` to encapsulate stack trace parsing and file verification.
3.  **Control Flow Optimization**: Implemented guard clauses to remove deep `if/else` nesting.
4.  **Logic Fix**: Corrected the loop logic to ensure the source context is printed exactly once for the first valid user-land file found in the stack trace.

## Result
The error handler is now fully asynchronous and non-blocking. The code structure is modular, with distinct responsibilities separated into helper functions. This ensures that even when errors occur, the server's main thread remains responsive.

## Reflection
While this was a small, isolated change, it sets a standard for production-readiness. Blocking operations should be strictly avoided in the hot path (or error path) of the server. The move to atomic functions also makes the error handler easier to extend if we later decide to add external logging services (like Sentry or Datadog) without cluttering the main logic flow.

[Github Commit Link](https://github.com/mackenziebowes/CanadianStartupJobs/commit/052279efb63ac616bcabb8be136fd3937655a6c6)
