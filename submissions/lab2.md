# Lab2

## Task 1: Baseline Threat Model



### Risk count by severity

| Severity | Count |

|----------|------:|

| Critical | 0 |

| High | 0 |

| Elevated | 4 |

| Medium | 14 |

| Low | 5 |

| \*\*Total\*\* | 23 |



### Top 5 risks (paste from PowerShell output)

1. \*\*unnecessary-data-transfer\*\* — Unnecessary Data Transfer of Tokens \& Sessions data at User Browser; severity low; affecting User Browser

2. \*\*missing-hardening\*\* — Missing Hardening risk at Juice Shop Application; severity medium; affecting Juice Shop Application

3. \*\*unnecessary-data-transfer\*\* — Unnecessary Data Transfer of Tokens \& Sessions data at User Browser; severity low; affecting User Browser

4. \*\*unencrypted-asset\*\* — Unencrypted Technical Asset named Juice Shop Application; severity medium; affecting Juice Shop Application

5. \*\*unencrypted-asset\*\* — Unencrypted Technical Asset named Persistent Storage; severity medium; affecting Persistent Storage



## Task 2: Secure Variant \& Diff



### Risk count comparison

| Severity | Baseline | Secure | Δ |

|----------|---------:|-------:|--:|

| Critical | 7 | 7 | 0 |

| Elevated | 4 | 3 | -1 |

| Low | 5 | 5 | 0 |

| Medium | 14 | 13 | -1 |

| \*\*Total\*\* | \*\*30\*\* | \*\*28\*\* | \*\*-2\*\* |



### Which rules are GONE in the secure variant?

No rule IDs completely disappeared from the report. The diff shows 0 eliminated rules. The changes made (enforcing HTTPS for direct app access and enabling storage encryption) did not remove the rules entirely, but rather downgraded the severity of specific risks (Elevated and Medium counts each dropped by 1).



### Which rules are STILL THERE in the secure variant?

1. \*\*unnecessary-data-transfer\*\* — Still fires because the application architecture still technically transfers session tokens to the client. Encrypting the transport layer (HTTPS) does not change the fact that the data is being sent unnecessarily according to Threagile's heuristics.

2. \*\*missing-hardening\*\* — Still fires because changing a communication link to HTTPS does not fix underlying application-level hardening issues (such as missing security headers, lack of rate limiting, or exposed debug info) on the Juice Shop Application asset itself.



### Honesty check

Did the total drop more than 50%? If yes, what does that say about the cost-benefit of these particular hardening changes vs. the work you'd need to fully eliminate the rest?

The total did not drop more than 50% (it decreased by only 2 risks). This indicates that the baseline model was already relatively well-configured for basic transport security. Simple configuration tweaks like forcing HTTPS on a direct link only mitigate specific eavesdropping risks, downgrading their severity rather than eliminating the underlying architectural patterns. Fully eliminating the remaining 28 risks would require deep code-level changes, implementing strict WAF rules, and comprehensive application hardening, which requires significantly more effort than declarative model adjustments.



## Bonus Task: Auth Flow Threat Model



### Risk count

| Severity | Count |

|----------|------:|

| Critical | 7 |

| Elevated | 6 |

| Low | 22 |

| Medium | 13 |

| \*\*Total\*\* | \*\*48\*\* |



### Three auth-specific risks (NOT in the baseline model's top 5)

1. \*\*missing-authorization\*\* — STRIDE: E — Mitigation: Implement strict server-side role-based access control (RBAC) to verify the 'admin' claim inside the JWT before allowing access to the Admin Endpoint.

2. \*\*missing-authentication\*\* — STRIDE: S — Mitigation: Enforce JWT validation middleware on all protected API routes to reject requests that do not contain a valid, signed token in the Authorization header.

3. \*\*cross-site-scripting\*\* — STRIDE: T — Mitigation: Implement strict Content Security Policy (CSP) headers and sanitize user inputs to prevent execution of malicious scripts in the user's browser session during the login flow.



### Reflection (2-3 sentences)

Building the focused model surfaced feature-level vulnerabilities like missing JWT validation and missing RBAC checks, which the baseline architecture-level model completely missed. The baseline only flagged generic infrastructure issues like unencrypted assets, while the auth model highlighted exactly how an attacker could spoof an admin request or elevate privileges by manipulating the token flow.

