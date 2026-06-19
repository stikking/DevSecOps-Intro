\# Lab 3 — Submission

\## Task 1: SSH Commit Signing

\### Local configuration

\- `git config --global gpg.format → ssh` 

\- `git config --global user.signingkey → C:/DevSecOps/git\_key.txt.pub`

\- `git config --global commit.gpgsign → true`

\### Local verification

Output of `git log --show-signature -1`: (commit ddb8972eba552c8d164e854d136b63a771650b13 (HEAD -> feature/lab3, origin/feature/lab3

Good "git" signature for a.salakhutdinov@innopolis.university with ED25519 key SHA256:iKF6rdu2MNKQc2SCSOA+FNe3wqEyv6rM71vxyvdL+ck

Author: Stikking <stikking11@gmail.com>

Date:   Fri Jun 19 18:07:56 2026 +0300

test: first signed commit)



\### GitHub verification

Direct link to your most recent commit on GitHub: (https://github.com/stikking/DevSecOps-Intro/commit/ddb8972eba552c8d164e854d136b63a771650b13)

\- Screenshot of the Verified badge: ("DevSecOps-Intro\\submissions\\verification.png")

\### One-paragraph reflection (2-3 sentences)

In a STRIDE-R scenario, an attacker could forge a commit by changing local Git settings to inject malicious code under a colleague's identity, allowing the real developer to plausibly deny responsibility. The green "Verified" badge makes this attack visible by cryptographically guaranteeing the commit was authored by the actual key owner. Consequently, any unsigned or forged commit lacks this badge, immediately alerting the team to suspicious activity and preventing the attacker from covering their tracks through impersonation.

