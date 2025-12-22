# Tools Architecture & DB Tools Vision - Dec 17, 2025

(This report was AI Generated from a coding agent)


## Situation
As we refactor the "Agentic Exploration" phase, we need a robust way to give the LLM capabilities that extend beyond just "looking at the web." The previous approach locked tools inside specific functions, making them hard to reuse or test. Furthermore, the problem of **Tagging** (mapping messy real-world data to strict DB enums) is difficult to solve with a single static prompt because the list of tags (Industries, Provinces) is dynamic and potentially large.

## Solution: Modular Tool Suites (`lib/ai/tools/*`)
We broke tools out into a dedicated directory, grouped by domain. This follows the **Interface Segregation Principle**—agents only import the tools they need.

### 1. `firecrawl.ts` (The Eyes)
*   **Purpose:** Web navigation and extraction.
*   **Tools:** `readPage`, `searchSite`.
*   **Reasoning:** Decouples the "how" of scraping (Firecrawl API) from the "what" (Agent intent). If we switch scraping providers, the agent logic remains unchanged.

### 2. `db.ts` (The Memory) - *The Vision*
*   **Purpose:** Allow the agent to interact with the database metadata *dynamically* during its reasoning loop.
*   **Tools:** `listTags`, `createTag`.
*   **The Vision:** instead of stuffing 500+ Industry strings into the system prompt (expensive, wasteful context window), we give the agent a **Tool** to query the database.
    *   *Agent Thought:* "This company looks like FinTech. Do I have a 'FinTech' tag?"
    *   *Action:* `listTags({ tagName: "Industry", skip: 0, take: 100 })`
    *   *Result:* "Found: 'Financial Technology', 'Banking', 'Crypto'"
    *   *Agent Decision:* "I'll map this to 'Financial Technology'."
    *   *Alternative:* "No match found. I will create a new tag 'FinTech' using `createTag`."

## Why this works
1.  **Scalability:** We can have 10,000 tags in the DB. The agent only retrieves what it needs.
2.  **Self-Correction:** If the agent guesses a tag that doesn't exist, it can verify first.
3.  **Active Learning:** The `createTag` tool allows the system to grow its ontology organically based on new data it encounters, rather than relying on a stale hardcoded list.

## Current Status
*   `firecrawl.ts`: Implemented.
*   `db.ts`: Skeleton implemented. Currently supports `Team Size` CRUD. Needs expansion to `Industry`, `Province`, and `Raising Stage`.

[Github Commit Link](https://github.com/mackenziebowes/CanadianStartupJobs/commit/b2ac7eb37fb94333334904b3867c7f354580555d)
