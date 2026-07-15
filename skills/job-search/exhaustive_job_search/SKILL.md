---
name: exhaustive-job-search
description: Orchestrates a coordinated, exhaustive job search using multiple sub-agents to find relevant jobs, matching them against the user's resume, and extracting direct ATS links for auto-fill automation.
---

# Exhaustive Job Search Skill

When you invoke this skill, your task is to launch a coordinated, exhaustive job search using multiple sub-agents. 

## Instructions

1. **Load/Get Search Configuration:**
   - Look for the saved configuration file at: `C:\Users\wongp\.gemini\antigravity\job_search_config.json`.
   - If the file exists, parse it to retrieve the default `resume_path`, `target_roles`, and `target_locations`.
   - Check the user's initial prompt or context. If the user provided custom inputs (resume path, target roles, or locations), use those to override the default values loaded from the configuration file.
   - If any values are missing from both the user prompt and the configuration file, explicitly ask the user to provide them before starting.
   - If the user provides new inputs, ask them if they want to save these settings as defaults. If they agree (or if they explicitly requested to save/remember them), write the new parameters to `C:\Users\wongp\.gemini\antigravity\job_search_config.json` in the following format:
     ```json
     {
       "resume_path": "C:\\path\\to\\resume.pdf",
       "target_roles": ["Engineering Manager", "Senior Software Engineer"],
       "target_locations": ["Remote", "Seattle"]
     }
     ```
   - Once the final resume path is resolved, read the resume contents using your tools (e.g., `view_file` or another file reading tool) to get the fresh content.

2. **Launch Sub-Agents:**
   - Determine the search tracks by partitioning the target roles/locations specified by the user. Assign one track to each sub-agent.
   - Use the `invoke_subagent` tool to launch the `research` sub-agents simultaneously (one for each track).
   - Pass the text of the user's resume and the specific search track (roles, locations, focus sites/companies) to each sub-agent in their prompt.
   - Instruct the sub-agents to prioritize searching on relevant job boards (e.g., LinkedIn, Indeed, local/regional job sites, or company careers pages).

3. **Sub-Agent Execution Rules:**
   - Exhaust all options! If they find 20 or 100 listings, they must process them all.
   - For every job found, the sub-agent must read the job description and **score it (1-10)** based on how well it matches the user's resume, providing a **brief 1-sentence reason** for the score.
   - Sub-agents should report back via the `send_message` tool when they have completely exhausted their search options.

4. **CRITICAL — Direct ATS Link Extraction (for Simplify Automation):**

   Most companies host their job listings on their own careers page but embed the actual application form inside an **iframe** powered by an Applicant Tracking System (ATS) like Greenhouse, Lever, Ashby, Workday, or SmartRecruiters. The user needs the **direct ATS URL** (the iframe `src`), NOT the company's wrapper page URL, so that the Simplify browser extension can auto-fill the application.

   **How to get the direct ATS URL:**
   - When you land on a company careers page, look for the actual job application page. If the page URL is the company's own domain but the application form is embedded, you MUST extract the iframe source URL.
   - Use `read_url_content` or browser tools to load the page and search for `<iframe` tags. The `src` attribute of the iframe is the direct ATS URL you need.
   - If you can identify the ATS from the URL pattern (see table below), you can often construct the direct URL without loading the page.

   **URL Pattern Conversion Table:**

   | ATS | Company Wrapper URL Pattern (❌ DO NOT USE) | Direct ATS URL Pattern (✅ USE THIS) |
   |---|---|---|
   | **Greenhouse** | `company.com/careers/jobs?gh_jid=JOBID` or `company.com/careers/jobs/apply/?gh_jid=JOBID` | `https://boards.greenhouse.io/{company_slug}/jobs/{JOBID}` or `https://job-boards.greenhouse.io/{company_slug}/jobs/{JOBID}` |
   | **Lever** | `company.com/careers/JOBID` (with Lever iframe) | `https://jobs.lever.co/{company_slug}/{JOBID}` |
   | **Ashby** | `company.com/careers/JOBID` (with Ashby iframe) | `https://jobs.ashbyhq.com/{company_slug}/{JOBID}` |
   | **Workday** | `company.com/careers/...` (with Workday iframe) | `https://{company}.wd{N}.myworkdayjobs.com/en-US/{site}/job/{job-path}/{JOBID}` |
   | **SmartRecruiters** | `company.com/careers/...` (with SmartRecruiters iframe) | `https://jobs.smartrecruiters.com/{Company}/{JOBID}` |

   **Concrete Examples:**

   | ❌ BAD (company wrapper) | ✅ GOOD (direct ATS link) |
   |---|---|
   | `https://www.stitchfix.com/careers/jobs?gh_jid=7966446` | `https://boards.greenhouse.io/stitchfix/jobs/7966446` |
   | `https://fingerprint.com/careers/jobs/apply/?gh_jid=6006388004` | `https://boards.greenhouse.io/fingerprint/jobs/6006388004` |
   | `https://openai.com/careers` | Navigate to the specific job, find the Greenhouse/Ashby/Lever URL |
   | `https://www.pinterestcareers.com/` | Navigate to the specific job, extract the ATS URL |
   | `https://boards.greenhouse.io/coinbase` | Find the specific job ID: `https://boards.greenhouse.io/coinbase/jobs/{JOBID}` |
   | `https://crescendo.ai/careers` | Navigate to the specific job, extract the ATS URL |

   **Rules:**
   - If the URL contains `gh_jid=` as a query parameter on a company domain, it is ALWAYS a Greenhouse wrapper. Convert it.
   - If you find a Greenhouse/Lever/Ashby/Workday/SmartRecruiters URL already in direct form, use it as-is.
   - **NEVER** link to a generic careers page (e.g., `company.com/careers`). You MUST find the specific job listing URL.
   - If you cannot determine the direct ATS URL, load the page with your tools, find the iframe `src`, and use that.

5. **Aggregation:**
   - After launching the sub-agents, wait for all of them to report back. Do not prematurely conclude the task. If a sub-agent takes too long, use `send_message` to check on its progress.
   - Once all sub-agents have reported their findings, consolidate the results into a single artifact named `job_search_results.md`.
   - Organize the artifact with sections for each of the searched roles/tracks.
   - Use a detailed Markdown table for each section with the exact following columns: `Company | Title | Location | Match Score 1-10 | Brief Reason | Application Link`.
   - **CRITICAL**: The `Application Link` column MUST contain a **direct ATS URL** (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, etc.) or a direct job board URL (LinkedIn `/jobs/view/JOBID`, Indeed, etc.) — NOT a company careers page wrapper. See Section 4 above for the conversion rules. This enables auto-fill via the user's Simplify extension.
   - **During aggregation, the orchestrator MUST validate every link** returned by sub-agents. If any link is a company wrapper URL (contains `gh_jid=` param, or points to a generic `/careers` page), convert it to the direct ATS URL before including it in the final report.
