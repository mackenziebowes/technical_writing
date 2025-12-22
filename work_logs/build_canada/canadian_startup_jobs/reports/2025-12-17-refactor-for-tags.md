# STAIR Report - Dec 17, 2025: Refactoring for Tagging & Modularity

(This report was AI Generated from a coding agent)


## Situation
The `createNewOrganizationFromURL` function successfully implemented the "Agentic Discovery" flow, but as we prepared to add **Tagging** (Industry, Province, Team Size, Stage), we realized the function was violating the **Single Responsibility Principle** and **Open/Closed Principle**. It was handling Firecrawl interactions, AI Discovery, AI Extraction, Error Handling, and DB Insertion all in one place. Adding tags would have made this function monolithic and brittle.

## Task
Refactor the organization creation pipeline to support:
1.  **Modularity:** Separate "Tools" (Search/Read) from "Extraction" logic.
2.  **Extensibility:** Allow adding new extractors (like Tags) without rewriting the core discovery logic.
3.  **Clean Architecture:** Move towards a "Service/Pipeline" pattern.

## Action
1.  **Extracted Tools:** Moved the Firecrawl-backed AI tools (`readPage`, `searchSite`) into their own directory (`apps/server/src/lib/ai/tools/`).
2.  **Refactored Firecrawl Utils:** Updated `scrape.ts` and `firecrawl/index.ts` to expose cleaner interfaces for the tools to use.
~~3.  **Prepared for Tagging:** Created a new prompt `getOrganizationTags.ts` and prepared the logic to fetch existing tags (like Team Sizes) to populate the Zod schema dynamically.~~
4.  **Updated Prompts:** Refined `getNewOrganization.ts` to be more focused on core data, delegating tagging to the new upcoming flow.

## Insight
Early refactoring prevents "God Functions." By splitting the "Agentic Explorer" (which builds context) from the "Extractors" (which structure data), we can reuse the expensive exploration step for multiple purposes (e.g., extracting Org data *and* Tag data *and* Social links) without re-running the agent or cluttering a single function.

## Results
*   **Codebase State:** The `lib/ai/tools` directory now houses the reusable agent capabilities.
*   **Readiness:** The system is now architecturally ready to accept the `TagExtractor` service without breaking the `OrganizationExtractor`.
*   **Cleanliness:** `createNewOrganizationFromURL` is becoming an orchestrator rather than a worker.

[Github Commit Link](https://github.com/mackenziebowes/CanadianStartupJobs/commit/0b58b649f7f3640ba7245a3ae7d84747c5a027c0)
