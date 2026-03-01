# SECURITY.md — Automated Security Scanning Guide

This document is designed to be read by an LLM agent. Follow the steps below in order:
1. Check whether each tool is installed; if not, install it using the instructions for the detected OS.
2. Run all three security scans against the current repository/project directory.
3. Report findings grouped by tool and severity.

---

## Supported Platforms

- **Ubuntu / Debian**
- **macOS**

> Windows is not supported. If a Windows environment is detected, inform the user and stop.

---

## Step 1 — Detect Operating System

Before installing or running any tool, detect the OS:

```bash
uname -s
```

- Output `Linux` → use **Ubuntu/Debian** instructions
- Output `Darwin` → use **macOS** instructions

---

## Step 2 — Check & Install Tools

For each tool below, first check if it is already installed. If the check command succeeds, skip the install. If it fails, run the install instructions for the detected OS.

---

### 2.1 Semgrep

**Check:**
```bash
semgrep --version
```

**Install — Ubuntu:**
```bash
# install through pip
python3 -m pip install semgrep

# If you receive "error: externally-managed-environment", use pipx instead:
# sudo apt install pipx
# pipx install semgrep

# Confirm installation
semgrep --version
```

**Install — macOS:**
```bash
# Option 1: Homebrew (recommended)
brew install semgrep

# Option 2: pip
python3 -m pip install semgrep

# Confirm installation
semgrep --version
```

> Note for Homebrew users: ensure Homebrew is added to your PATH.

---

### 2.2 Trivy

**Check:**
```bash
trivy --version
```

**Install — Ubuntu:**
```bash
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**Install — macOS:**
```bash
brew install trivy
```

**Confirm installation:**
```bash
trivy --version
```

---

### 2.3 Gitleaks

**Check:**
```bash
gitleaks version
```

**Install — Ubuntu:**
```bash
GITLEAKS_VERSION=$(curl -s "https://api.github.com/repos/gitleaks/gitleaks/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
wget -qO gitleaks.tar.gz https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_${GITLEAKS_VERSION}_linux_x64.tar.gz
sudo tar xf gitleaks.tar.gz -C /usr/local/bin gitleaks
rm -rf gitleaks.tar.gz

# Confirm installation
gitleaks version
```

**Install — macOS:**
```bash
brew install gitleaks

# Confirm installation
gitleaks version
```

---

## Step 3 — Run Security Scans

Once all three tools are confirmed installed, run the following scans from the **root of the repository/project directory**.

> Set the target path variable before running:
> ```bash
> TARGET="."   # or replace with the path to the project directory
> ```

---

### 3.1 Semgrep — OWASP Top 10 & General Security

Semgrep is used to detect code-level vulnerabilities including those in the OWASP Top 10 (injection, broken auth, sensitive data exposure, XXE, broken access control, security misconfiguration, XSS, insecure deserialization, known vulnerable components, insufficient logging).

```bash
semgrep scan \
  --config "p/owasp-top-ten" \
  --config "p/security-audit" \
  --config "p/secrets" \
  --severity ERROR \
  --severity WARNING \
  --json \
  --output semgrep-results.json \
  $TARGET
```

**Human-readable output (optional):**
```bash
semgrep scan \
  --config "p/owasp-top-ten" \
  --config "p/security-audit" \
  --config "p/secrets" \
  --severity ERROR \
  --severity WARNING \
  $TARGET
```

**Key flags explained:**
- `p/owasp-top-ten` — rules mapped to OWASP Top 10 categories
- `p/security-audit` — broad security audit ruleset
- `p/secrets` — detects hardcoded secrets and credentials in source code
- `--severity ERROR --severity WARNING` — captures high and medium severity findings
- `--json --output semgrep-results.json` — saves findings for review

---

### 3.2 Trivy — Vulnerability Scanning (Dependencies, IaC, Container Images)

Trivy scans for known CVEs in dependencies, misconfigurations in infrastructure-as-code, and exposed secrets.

**Filesystem scan (dependencies + IaC + secrets):**
```bash
trivy fs \
  --scanners vuln,misconfig,secret \
  --severity CRITICAL,HIGH,MEDIUM \
  --format json \
  --output trivy-results.json \
  $TARGET
```

**Human-readable output (optional):**
```bash
trivy fs \
  --scanners vuln,misconfig,secret \
  --severity CRITICAL,HIGH,MEDIUM \
  $TARGET
```

**Key flags explained:**
- `--scanners vuln` — scans for known CVEs in package dependencies (npm, pip, go, etc.)
- `--scanners misconfig` — checks IaC files (Dockerfile, Kubernetes, Terraform, etc.) for security misconfigurations
- `--scanners secret` — detects hardcoded secrets and tokens
- `--severity CRITICAL,HIGH,MEDIUM` — filters to actionable findings; add `LOW` if comprehensive output is needed
- `--format json --output trivy-results.json` — saves findings for review

---

### 3.3 Gitleaks — Secrets & Credentials in Git History

Gitleaks scans the full Git history and working directory for hardcoded secrets, API keys, tokens, and credentials.

**Scan full Git history:**
```bash
gitleaks detect \
  --source $TARGET \
  --report-format json \
  --report-path gitleaks-results.json \
  --verbose
```

**Scan staged/uncommitted changes only:**
```bash
gitleaks protect \
  --source $TARGET \
  --report-format json \
  --report-path gitleaks-staged-results.json \
  --staged \
  --verbose
```

**Key flags explained:**
- `detect` — scans the full Git log and working tree
- `protect` — scans only staged changes (useful as a pre-commit check)
- `--report-format json` — structured output for review or CI integration
- `--verbose` — prints each finding as it is discovered

---

## Step 4 — Review & Report Findings

After all scans complete, summarise findings as follows:

1. **Semgrep** (`semgrep-results.json`) — list findings by OWASP category, file, line number, and severity.
2. **Trivy** (`trivy-results.json`) — list CVEs by package, version, fixed version (if available), and severity; list any misconfigurations and secrets found.
3. **Gitleaks** (`gitleaks-results.json`) — list any detected secrets by file, line, rule ID, and commit hash.

For each finding provide:
- Tool name
- Severity (CRITICAL / HIGH / MEDIUM / LOW)
- File path and line number
- Description of the issue
- Recommended remediation

If no issues are found by a tool, explicitly state: `[TOOL NAME]: No issues detected.`

---

## Output Files

| File | Tool | Contents |
|---|---|---|
| `semgrep-results.json` | Semgrep | OWASP Top 10 & security audit findings |
| `trivy-results.json` | Trivy | CVEs, misconfigurations, secrets |
| `gitleaks-results.json` | Gitleaks | Secrets in full Git history |
| `gitleaks-staged-results.json` | Gitleaks | Secrets in staged changes |

---

## Notes

- Always run scans from the root of the repository unless a specific subdirectory is intended.
- Gitleaks requires the target directory to be a Git repository (i.e. `.git` folder must be present). If it is not, skip the Gitleaks scan and inform the user.
- Trivy will automatically download and update its vulnerability database on first run; an internet connection is required.
- Semgrep rules are fetched from the Semgrep registry; an internet connection is required.
- These scans are non-destructive and read-only; they will not modify any project files.