# Agentic AI Custom Skills Registry

A centralized registry for tracking, using, and sharing agentic AI skills (compatible with coding assistants like Gemini and Antigravity). While the initial set of skills is focused on job search automation and resume optimization, this repository is designed to host and organize custom skills across any domain.

---

## 📂 Repository Structure

Skills are grouped into domain-specific subdirectories under the `skills/` directory.

```
├── LICENSE
├── README.md
└── skills
    └── <domain>/                         # e.g., job-search, coding, writing, utilities
        └── <skill-name>/
            ├── SKILL.md                 # Core instructions and metadata for the skill
            └── scripts/                 # Optional auxiliary scripts or executables
```

### Current Registry Layout
```
└── skills
    └── job-search
        ├── exhaustive_job_search
        │   └── SKILL.md                 # Configures multi-agent job searching & ATS URL parsing
        └── resume-ats-analyzer
            ├── SKILL.md                 # Simulates recruiter screening on resumes against JDs
            └── scripts
                └── extract_text.py      # Extract text from PDF, DOCX, TXT, and MD formats
```

---

## 🛠️ Included Skills

### Domain: `job-search`

#### 1. Exhaustive Job Search
Located at: [exhaustive_job_search/SKILL.md](file:///c:/Projects/skills/skills/job-search/exhaustive_job_search/SKILL.md)

Orchestrates a parallelized, multi-agent search for open roles matching a user's target profile and qualifications.

- **Automated Resume Parsing**: Reads a specified resume PDF path to guide the search.
- **Coordinated Sub-Agent Execution**: Spawns 6 specialized `research` sub-agents simultaneously targeting different career tracks (EM, Senior SWE, Mid SWE, Staff, Solutions Architect, Full Stack).
- **Direct ATS Link Extraction**: Automatically converts company-specific wrapper URLs into clean, direct Applicant Tracking System (ATS) iframe source URLs (Greenhouse, Lever, Ashby, Workday, SmartRecruiters) to support auto-fill extensions like **Simplify**.
- **Consolidated Scoring**: Ranks match suitability on a scale of 1-10 with accompanying rationale.

#### 2. Resume ATS Analyzer
Located at: [resume-ats-analyzer/SKILL.md](file:///c:/Projects/skills/skills/job-search/resume-ats-analyzer/SKILL.md)

Simulates an ATS parser and professional recruiter to review a resume against a target Job Description (JD).

- **Format Interoperability**: Parses documents in `.pdf`, `.docx`, `.txt`, and `.md` formats using a helper script.
- **Evaluation Pillars**: Evaluates ATS parsing accuracy, core keyword matches, screening metrics (tenure, years of experience), recruiter red/green flags, and likely search queries.

---

## ➕ Creating & Sharing Skills

This repository is built to be easily extensible. To add and share a new skill:

1. **Choose or Create a Domain**: Determine which subdirectory under `skills/` your skill belongs to (e.g., `skills/coding`, `skills/utilities`).
2. **Create a Skill Folder**: Create a directory for your skill containing at least a `SKILL.md` file.
3. **Write the `SKILL.md` File**: Define the instructions for the agent. The file must start with YAML frontmatter containing the `name` and `description` of the skill:
   ```markdown
   ---
   name: your-skill-name
   description: A short description of what this skill does and when the agent should use it.
   ---

   # Your Skill Title
   Detailed step-by-step instructions for the agent...
   ```
4. **(Optional) Add Scripts**: Put auxiliary scripts (Python, Bash, Node, etc.) in a `scripts/` directory inside your skill folder.

---

## 🚀 Loading Skills

To load and use these skills in your own agent environment:
- **Workspace-level**: Copy or symlink the domain directory (or specific skill folder) to your workspace's `.agents/skills/` directory.
- **Global-level**: Copy or symlink the skill folder into your agent's global customization directory (typically `~/.gemini/config/skills/`).

The agent automatically discovers and registers these skills on startup.

---

## 📄 License

This repository is distributed under the MIT License. See the [LICENSE](file:///c:/Projects/skills/LICENSE) file for details.
