Security Scanner
A plug-and-play security scanning workflow powered by SECURITY.md. Point your LLM agent at this file and it will automatically install and run three industry-standard security tools against any codebase — no manual setup required.

What This Does
SECURITY.md is a structured instruction file designed to be read by an LLM (such as Claude, ChatGPT, GitHub Copilot, or any agent with tool-use capability). When prompted, the LLM will:

Detect your operating system (Ubuntu or macOS)
Check whether each security tool is installed — and install any that are missing
Run a full security scan using three tools in sequence
Summarise all findings with severity ratings, file locations, and remediation advice

Tools Used
ToolPurposeSemgrepStatic code analysis — OWASP Top 10, security audit rules, hardcoded secrets in sourceTrivyDependency CVEs, IaC misconfigurations (Dockerfile, Terraform, Kubernetes), secretsGitleaksSecrets and credentials leaked in Git history or staged changes

Supported Platforms
PlatformSupportedUbuntu / Debian✅macOS✅Windows❌

How to Use This in Your Own Repository
You can drop SECURITY.md into any project and use it immediately. There are two ways to do this:
Option A — Copy the file directly

Download or copy SECURITY.md from this repository
Place it in the root of your project:

   your-project/
   ├── SECURITY.md   ← add this
   ├── src/
   ├── package.json
   └── ...

Open your LLM of choice and use the prompt in the section below

Option B — Reference it via URL
If you don't want to copy the file, you can paste the raw URL of SECURITY.md directly into your LLM prompt and ask it to fetch and follow the instructions.

Prompting Your LLM
Once SECURITY.md is in place (or you have the URL), use a prompt like one of the following:
If the file is in your project:
Please read SECURITY.md in this repository and follow the instructions to check 
whether Semgrep, Trivy, and Gitleaks are installed. Install any that are missing, 
then run all three security scans against this project and summarise the findings.
If using an agent with file access (e.g. Claude, Copilot Workspace):
Read the SECURITY.md file and execute the steps inside it. Detect my OS, install 
any missing tools, run all scans, and give me a full findings report grouped by 
tool and severity.
If passing the raw URL:
Please fetch the contents of [URL to your raw SECURITY.md] and follow the 
instructions to run a full security scan on this codebase.

What the Scan Covers
Semgrep

OWASP Top 10 vulnerabilities (injection, XSS, broken auth, IDOR, etc.)
General security audit rules
Hardcoded secrets and credentials in source code
Severity: ERROR and WARNING

Trivy

Known CVEs in project dependencies (npm, pip, Go modules, Maven, etc.)
Misconfigurations in Dockerfiles, Kubernetes manifests, Terraform, and other IaC
Hardcoded secrets and tokens in files
Severity: CRITICAL, HIGH, and MEDIUM

Gitleaks

API keys, tokens, passwords, and credentials committed anywhere in Git history
Secrets in staged but uncommitted changes
Covers 150+ secret types out of the box


Output Files
After a scan completes, the following files will be created in your project root:
FileContentssemgrep-results.jsonOWASP Top 10 and security audit findingstrivy-results.jsonCVEs, misconfigurations, and secretsgitleaks-results.jsonSecrets detected in full Git historygitleaks-staged-results.jsonSecrets detected in staged changes
These files are machine-readable JSON. Your LLM agent will parse them and produce a plain-English summary. You can also feed them into other tools or dashboards.

Tip: Add these output files to your .gitignore to avoid committing scan results to your repository.

gitignore# Security scan output
semgrep-results.json
trivy-results.json
gitleaks-results.json
gitleaks-staged-results.json

Requirements

An LLM agent with the ability to run terminal/shell commands (e.g. Claude with computer use, GitHub Copilot Workspace, Cursor, or a locally run agent)
Internet access (Trivy and Semgrep fetch their rule databases on first run)
A Unix-based OS (Ubuntu/Debian or macOS)
The target directory must be a Git repository for Gitleaks to run; if it isn't, Gitleaks will be skipped automatically


Example Findings Report
After running the scans, your LLM should produce a summary along these lines:
SEMGREP — 3 findings
  [HIGH] sql-injection detected in src/db/query.js:42
  [WARNING] hardcoded-secret detected in config/settings.py:17
  [WARNING] xss-detected in src/views/profile.js:88

TRIVY — 5 findings
  [CRITICAL] CVE-2023-44487 in http2 v1.0.0 (fixed in v1.0.1)
  [HIGH] CVE-2024-21626 in runc v1.1.5 (fixed in v1.1.12)
  [MEDIUM] Dockerfile: privileged container enabled (Dockerfile:14)

GITLEAKS — 2 findings
  [HIGH] github-pat detected in .env:3 (commit: a3f91bc)
  [HIGH] aws-access-key-id detected in scripts/deploy.sh:11 (commit: 7c204de)

Contributing
Found an improvement to the scan configuration or want to add support for additional tools or platforms? PRs are welcome. Please test any changes against both Ubuntu and macOS before submitting.

License
Apache License — free to use, copy, and adapt in your own projects.