# Lab 3 — Submission
## Task 1: SSH Commit Signing
### Local configuration
- `git config --global gpg.format → ssh` 
- `git config --global user.signingkey → C:/DevSecOps/git_key.txt.pub`
- `git config --global commit.gpgsign → true`
### Local verification
Output of `git log --show-signature -1`: (commit ddb8972eba552c8d164e854d136b63a771650b13 (HEAD -> feature/lab3, origin/feature/lab3
Good "git" signature for a.salakhutdinov@innopolis.university with ED25519 key SHA256:iKF6rdu2MNKQc2SCSOA+FNe3wqEyv6rM71vxyvdL+ck
Author: Stikking <stikking11@gmail.com>
Date:   Fri Jun 19 18:07:56 2026 +0300
test: first signed commit)

### GitHub verification
Direct link to your most recent commit on GitHub: (https://github.com/stikking/DevSecOps-Intro/commit/ddb8972eba552c8d164e854d136b63a771650b13)
- Screenshot of the Verified badge: ("DevSecOps-Intro\submissions\verification.png")
### One-paragraph reflection (2-3 sentences)
In a STRIDE-R scenario, an attacker could forge a commit by changing local Git settings to inject malicious code under a colleague's identity, allowing the real developer to plausibly deny responsibility. The green "Verified" badge makes this attack visible by cryptographically guaranteeing the commit was authored by the actual key owner. Consequently, any unsigned or forged commit lacks this badge, immediately alerting the team to suspicious activity and preventing the attacker from covering their tracks through impersonation.

## Task 2: Pre-commit + gitleaks
.pre-commit-config.yaml (paste the full content)
repos:  - repo: https://github.com/gitleaks/gitleaks    rev: v8.18.1    hooks:      - id: gitleaks  - repo: https://github.com/pre-commit/pre-commit-hooks    rev: v4.5.0    hooks:      - id: detect-private-key      - id: check-added-large-files
pre-commit install output
pre-commit installed at .git/hooks/pre-commit
(If using python -m pre_commit install, the output is the same)

The blocked commit
Output of the git commit that gitleaks blocked (the failing hook output):

Detect hardcoded secrets.................................................Failed

hook id: gitleaks
exit code: 1
○
│╲
│ ○
○ ░
░ gitleaks

Finding: GH_PAT=REDACTED
Secret: REDACTED
RuleID: github-pat
Entropy: 4.143943
File: submissions/leak-attempt.txt
Line: 2
Fingerprint: submissions/leak-attempt.txt:github-pat:2

6:57PM INF 1 commits scanned.
6:57PM INF scan completed in 91.2ms
6:57PM WRN leaks found: 1

Tune-out exercise
### Suppose a teammate insists they need to commit AKIA* strings because they're documentation examples in docs/. Briefly describe two approaches:
Inline allowlist — [allowlist] block in .gitleaks.toml. When is this OK?
- * This is acceptable when the string is a well-known, explicitly documented fake value (such as AWS's AKIAIOSFODNN7EXAMPLE) used strictly for educational or testing purposes. By allowlisting the specific string, you ensure the scanner ignores it while still actively catching any real, dynamically generated secrets that match the rule.

- * Path exclusion — paths: [docs/] in .gitleaks.toml. When is this risky? This is risky because it creates a blind spot where any file in that directory completely bypasses scanning. If a developer accidentally pastes a real, valid secret into a documentation file, the pre-commit hook will fail to catch it, potentially leading to a leaked credential in the repository's history.

## Bonus: History Rewrite
### Before
272737a (HEAD -> master) docs: add usage notes5c3be9e feat: empty log05c18f5 feat: add config9aef3e5 init

Output of git log -p | grep -c 'ghp_': 2

### After
5016931 (HEAD -> master) docs: add usage notes0634471 feat: empty logb4c3161 feat: add config56d2607 init

Output of git log -p | grep -c 'ghp_': 0Output of git log -p | grep -c 'REDACTED': 2

### The two-step pattern in real life
git filter-repo --replace-text replacements.txt — rewrite locally
Key rotation (revoking and reissuing the compromised secret) — what's the MANDATORY second step in a real incident?(Hint: Lecture 3 slide 12 has this — it's the difference between cleanup and remediation.)
Two real-world gotchas you discovered (2 sentences each)
git filter-repo was not recognized as a git command because Python's Scripts directory wasn't in the system PATH. I had to download the git-filter-repo script manually and execute it directly via python git-filter-repo to bypass the environment issue.
PowerShell's Out-File cmdlet added a Byte Order Mark (BOM) to the replacement text file, which caused the filter-repo matching to fail silently on the first run. I had to use [System.IO.File]::WriteAllText to generate a clean UTF-8 file without a BOM for the secret replacement to work correctly.


