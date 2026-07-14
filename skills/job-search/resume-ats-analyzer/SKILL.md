---
name: resume-ats-analyzer
description: >-
  Extracts text from PDF, DOCX, TXT, or MD resumes and analyzes them against job descriptions or general ATS filters to generate a recruiter compatibility report.
---

# Resume ATS Analyzer

## Overview
The `resume-ats-analyzer` skill allows the agent to parse resumes in multiple formats (PDF, DOCX, TXT, MD) and evaluate them as a recruiter using an Applicant Tracking System (ATS) filter (such as Greenhouse, Lever, or Workday). It can evaluate resumes against a specific job description or perform a general assessment.

## Dependencies
None.

## Quick Start
To extract text from a resume:
```bash
uv run "~/.gemini/config/skills/resume-ats-analyzer/scripts/extract_text.py" "path/to/resume.pdf" --output "path/to/output.txt"
```

Then, use the extracted resume text along with any provided job description to perform the recruiter analysis.

## Utility Scripts
The skill includes a Python script located at `scripts/extract_text.py` to handle document formats that are not natively readable as plain text (such as PDF and DOCX).

**Command Line Usage**:
```bash
uv run "~/.gemini/config/skills/resume-ats-analyzer/scripts/extract_text.py" <input_path> --output <output_path>
```
* **Arguments**:
  - `input_path`: Path to the resume file (`.pdf`, `.docx`, `.txt`, `.md`).
  - `--output`: Required path where the plain text content should be written.

## Workflow

### 1. Extract Resume Text
Determine the format of the resume file. If the file is a PDF or DOCX, run the text extraction helper script to generate a plain text representation of the resume:
```bash
uv run "~/.gemini/config/skills/resume-ats-analyzer/scripts/extract_text.py" "<RESUME_PATH>" --output "<TEMP_TXT_PATH>"
```
If the file is already a `.txt` or `.md` file, you can read it directly.

### 2. Fetch Job Description (If Provided)
Identify how the job description is provided:
- **Raw Text**: Read it directly from the prompt.
- **Local File**: View/read the file directly.
- **URL**: Use the `read_url_content` or `read_browser_page` tool to fetch the text of the job description.

### 3. Conduct ATS Recruiter Simulation
Analyze the extracted resume text and the job description using LLM reasoning. Simulate the behavior of a recruiter filtering applications on an ATS (like Greenhouse/Lever) using these evaluation pillars:

* **ATS Parsing Accuracy**: How cleanly the layout, headers, and dates will map to database fields.
* **Core Keyword Match Strength**: List critical skills/technologies from the JD and evaluate if the resume explicitly matches them.
* **Key Screening Metrics**: Assess total years of relevant experience, tenure stability, and degree verification (flagging if it specifies CS-only and candidate lacks one, or if experience overrides it).
* **Recruiter Red & Yellow Flags**: Identify gaps in employment history, short job tenures, lack of direct context, or excessive buzzwords.
* **Candidate Strengths / Green Flags**: Highlight strong metrics, technical leadership, and clear project impacts.
* **Recruiter Search Terms**: List the exact query terms a recruiter is likely to search for that would surface this resume.

### 4. Write the Report
Create a beautifully formatted Markdown report containing the full analysis and save it as requested.

## Common Mistakes
* **Reading DOCX Directly**: Do not attempt to view/read `.docx` files using standard text viewing tools as they are binary zip files. Always use the `extract_text.py` script.
* **Omitting Table Data**: Resumes often put skills or contact details inside tables. Ensure the extraction script is used as it parses tables correctly.
