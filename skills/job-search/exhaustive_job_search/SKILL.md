---
name: exhaustive-job-search
description: Orchestrates an exhaustive job search using multiple sub-agents for Remote and Seattle-based roles across Engineering Management, various Software Engineering levels, and Solutions Architecture, cross-referencing jobs against the user's resume for a fit score.
---

# Exhaustive Job Search Skill

When you invoke this skill, your task is to launch a coordinated, exhaustive job search using multiple sub-agents. 

## Instructions

1. **Read the User's Resume First:**
   - Before launching the sub-agents, use your tools (like `view_file` or a PDF parsing tool if necessary) to read the user's latest resume located at: `C:\Users\wongp\OneDrive\Documents\Resume\2026\Universal\Patrick_Wong_Resume_Software_Engineer_Manager.pdf`.
   - You MUST fetch it fresh every time this skill is run.

2. **Launch Sub-Agents:**
   - Use the `invoke_subagent` tool to launch six `research` sub-agents simultaneously. 
   - Pass the text of the user's resume to each sub-agent in their prompt.
   - Instruct the sub-agents to prioritize searching on **LinkedIn, BuiltInSeattle, Levels.fyi jobs, and direct careers pages of Big Tech companies**.
   - Assign each sub-agent one of the following specific search tracks:
     - **Agent 1 (EM Searcher):** Search for Remote or Seattle-based "Engineering Manager" job listings.
     - **Agent 2 (Senior SE Searcher):** Search for Remote or Seattle-based "Senior Software Engineer" job listings.
     - **Agent 3 (Intermediate Big Tech Searcher):** Search for Remote or Seattle-based "Intermediate Software Engineer" job listings specifically at Big Tech companies (e.g., FAANG, Microsoft, Amazon).
     - **Agent 4 (Staff & Tech Lead Searcher):** Search for Remote or Seattle-based "Staff Software Engineer" and "Tech Lead" job listings.
     - **Agent 5 (Solutions & FDE Searcher):** Search for Remote or Seattle-based "Solutions Architect" and "Forward Deployed Engineer" job listings.
     - **Agent 6 (Senior Full Stack Searcher):** Search for Remote or Seattle-based "Senior Full Stack Engineer" job listings (prioritizing Golang, React, Angular stacks).

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
   - After launching the sub-agents, wait for all six to report back. Do not prematurely conclude the task. If a sub-agent takes too long, use `send_message` to check on its progress.
   - Once all sub-agents have reported their findings, consolidate the results into a single artifact named `job_search_results.md`.
   - Organize the artifact with sections for each of the 6 roles.
   - Use a detailed Markdown table for each section with the exact following columns: `Company | Title | Location | Match Score 1-10 | Brief Reason | Application Link`.
   - **CRITICAL**: The `Application Link` column MUST contain a **direct ATS URL** (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, etc.) or a direct job board URL (LinkedIn `/jobs/view/JOBID`, Indeed, etc.) — NOT a company careers page wrapper. See Section 4 above for the conversion rules. This enables auto-fill via the user's Simplify extension.
   - **During aggregation, the orchestrator MUST validate every link** returned by sub-agents. If any link is a company wrapper URL (contains `gh_jid=` param, or points to a generic `/careers` page), convert it to the direct ATS URL before including it in the final report.
