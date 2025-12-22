# STAR Report - Dec 19, 2025: Tagging Agent Observability + Org Flow Hardening

(This report was AI Generated from a coding agent)


**Date:** December 19, 2025
**Type:** Feature / Developer Experience
**Status:** Complete

## Situation
We introduced an agentic tagging loop to enrich newly-created Organizations with tags (Province, Industry, Team Size, Raising Stage). However, we lacked visibility into what the agent was doing internally: which tool calls it made, what tool results it received, and what it was “thinking” between steps. This made debugging expensive and slow, and we also noticed duplicate tool calls (e.g., the tagging agent re-mapping/searching the site after the primary discovery pass).

## Task
1. Add lightweight observability to the AI agent step loop so we can inspect the agent’s behavior across test data.
2. Keep the observability implementation DRY and reusable across multiple agentic flows.
3. Harden the organization creation flow so `careersPage` is enforced as required during structured extraction.
4. Capture a reminder to build a data harness/caching layer to reduce duplicate tool calls between the discovery and tagging phases.

## Action
1. **Built an observability module** in `apps/server/src/lib/ai/observability.ts`:
   - Implemented a `observePrepareSteps(agentName)` function compatible with the Vercel AI SDK `prepareStep` hook.
   - Optimized logging to inspect only the most recent step (`steps.at(-1)`) to avoid repeatedly flattening the entire step history.
   - Logged step number, last reasoning/thought, last tool call, last tool result, and last message.
   - Added timestamps for easier timing/latency debugging.

2. **Integrated observability into org discovery + tagging**:
   - `createNewOrganizationFromURL.ts` uses `prepareStep: observePrepareSteps("Primary")` for the discovery stage.
   - `autoTagOrganization.ts` uses `Experimental_Agent` with `prepareStep: observePrepareSteps("Tagging")` to instrument the tagging loop.

3. **Schema hardening for `careersPage`**:
   - Modified `getObjectData` in `apps/server/src/lib/ai/functions/createNewOrganizationFromURL.ts` to extend the insert schema and make `careersPage` explicitly required (non-optional) at extraction time.

4. **Captured duplicate-call follow-up**:
   - Added `notes/2025-12-19-data-harness-reminder.md` documenting the need for a “data harness” (cache/hand-off of sitemap and discovery artifacts) to avoid re-running expensive web navigation during tagging.

## Result
- We can now see (with timestamps) how the tagging agent progresses step-by-step: tool calls, tool results, and reasoning.
- Debugging the org flow is dramatically faster because failures and redundant calls are visible immediately.
- `careersPage` extraction is more reliable: invalid/partial org objects fail earlier and more explicitly.
- A concrete follow-up note exists to implement a shared context/caching layer to reduce duplicate `mapSite`/search work across phases.

## Reflection
Agentic workflows are only practical in production when they’re observable. This work establishes a reusable `prepareStep` observer pattern we can attach to any future tool-calling flow. The next leverage point is to persist/hand off discovery artifacts (site map / relevant page snippets) between phases to reduce repeated exploration cost and improve determinism.

[Github Commit Link](https://github.com/mackenziebowes/CanadianStartupJobs/commit/21694095c569e545b1036cb5ff4894fc5b04baa8)
