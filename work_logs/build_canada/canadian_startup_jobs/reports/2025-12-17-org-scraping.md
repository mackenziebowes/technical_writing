# STAIR Report - Dec 17, 2025

(This report was AI Generated from a coding agent)


## Situation
We are building a Canadian Startup Job Board that aggregates jobs from company career pages. The core challenge is populating the database with accurate Organization data (companies) from a simple URL. Many sites hide key details (HQ location, team size) on secondary pages like "About Us" or "Contact," making simple scraping insufficient.

## Task
Implement an automated `createNewOrganizationFromURL` function that:
1.  Takes a URL.
2.  Accurately extracts core details (Name, City, Province, Description).
3.  Handles cases where data is missing from the home page.
4.  Standardizes the output into our DB schema.

## Action
We engineered a **Two-Pass Agentic Workflow**:
1.  **Discovery Phase (Agentic):** We implemented a `generateText` call equipped with custom tools (`readPage`, `searchSite`) backed by Firecrawl. This allows the LLM to autonomously navigate the site map, identifying and reading relevant pages (e.g., searching for "Contact" to find a city) if the home page is insufficient.
2.  **Extraction Phase (Structured):** We piped the unstructured findings from the Discovery Phase into a `generateObject` call constrained by a Zod schema matching our Database.
3.  **Integration:** We utilized Google's Gemini 2.0 Flash (via Vercel AI SDK) for cost-effective, high-speed reasoning and extraction.

## Insight
Simple "fetch and scrape" pipelines fail on real-world websites because information architecture varies wildly. By giving the LLM "eyes" (tools) and a goal ("find the city"), we shifted from *guessing* where data is to *finding* it. The two-pass approach decouples "hunting" (high token usage, reasoning) from "formatting" (strict schema enforcement), resulting in higher accuracy and cleaner data.

## Results
*   **Capabilities:** The system can now autonomously find organization details even if buried deep in a footer link or separate "About" page.
*   **Architecture:** Created a reusable pattern for "Agentic Scraping" that can be applied to Job parsing next.
*   **Reliability:** Added robust error handling (custom `AppError` codes) and fallback logic to ensure the pipeline fails gracefully.

[Github Commit Link](https://github.com/mackenziebowes/CanadianStartupJobs/commit/0b58b649f7f3640ba7245a3ae7d84747c5a027c0)
