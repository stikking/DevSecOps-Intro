# Lab 5 — Submission



## Task 1: DAST with OWASP ZAP



### Baseline (unauthenticated) scan

- Duration: \~2 minutes

- Total alerts: 8

| Severity | Count |

|----------|------:|

| High | 0 |

| Medium | 8 |

| Low | 0 |

| Informational | 0 |



### Authenticated full scan

- Duration: \~5 minutes

- Total alerts: 12

| Severity | Count |

|----------|------:|

| High | 1 |

| Medium | 6 |

| Low | 3 |

| Informational | 2 |



### The "10–20× more" claim (Lecture 5 slide 11)

- Ratio (auth alerts / baseline alerts): 1.5x (12 / 8)

- Did your run match the lecture's ratio? No, our run did not match the 10-20x claim. This is likely because modern applications like Juice Shop expose a significant amount of surface area (APIs, static files) to unauthenticated users, and the ZAP Ajax spider successfully crawled hundreds of URLs without needing credentials. Additionally, ZAP's baseline scan is purely passive, while the authenticated scan uses active scanning, which might yield fewer but more critical alerts rather than a massive volume of low-hanging fruit.

- Pick \*\*two specific alerts\*\* that only the authenticated scan found. For each:

&#x20; 1. Alert title + severity: SQL Injection (High)

&#x20;    Why was it unreachable to the unauthenticated scan? The baseline scan is passive and does not actively fuzz input parameters like `q` in the search endpoint, whereas the authenticated active scan sent injection payloads directly to `/rest/products/search`.

&#x20; 2. Alert title + severity: Private IP Disclosure (Low)

&#x20;    Why was it unreachable to the unauthenticated scan? The `/rest/admin/application-configuration` endpoint exposes internal IP addresses and is strictly restricted to authenticated administrators.



## Task 2: SAST with Semgrep


### Semgrep severity breakdown

| Severity | Count |

|----------|------:|

| ERROR | 12 |

| WARNING | 10 |

| INFO | 0 |

| \*\*Total\*\* | 22 |



### Top 10 rules by frequency

| Rule ID | Count | OWASP category |

|---------|------:|----------------|

| javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection | 6 | A03 Injection |

| yaml.github-actions.security.run-shell-injection.run-shell-injection | 5 | A03 Injection |

| javascript.express.security.audit.express-check-directory-listing.express-check-directory-listing | 4 | A05 Security Misconfig |

| javascript.express.security.audit.express-res-sendfile.express-res-sendfile | 4 | A05 Security Misconfig |

| javascript.express.security.audit.express-open-redirect.express-open-redirect | 1 | A01 Broken Access Control |

| javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret | 1 | A02 Cryptographic Failures |

| javascript.lang.security.audit.code-string-concat.code-string-concat | 1 | A03 Injection |



### Triage shortcut (Lecture 5 slide 8)

Looking at the top 10 — which \*\*one rule\*\* would you fix first if you had time for only one?

Why? I would fix the `sequelize-injection-express` rule first. It has the highest frequency (6 occurrences) and directly relates to A03: Injection, which is one of the most critical and exploitable OWASP categories. Fixing this at the database query module level (e.g., by parameterizing inputs) would immediately close multiple high-risk findings across the application.



### False-positive sample

Pick \*\*one\*\* finding you'd suppress as a false positive after review.

- File path: `labs/lab5/semgrep/juice-shop/data/static/codefixes/unionSqlInjectionChallenge\_1.ts`

- Rule: `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection`

- Reason: This file is located in the `data/static/codefixes/` directory, which contains intentionally vulnerable code snippets used as educational examples for Juice Shop challenges, not actual production application code.



---



## Bonus: SAST/DAST Correlation



### Correlation table

| # | OWASP cat | ZAP alert | ZAP URI | Semgrep rule | Semgrep file:line | Confidence |

|---|-----------|-----------|---------|--------------|-------------------|------------|

| 1 | A03 Injection | SQL Injection | http://juice-shop:3000/rest/products/search?q=... | sequelize-injection-express | labs/lab5/semgrep/juice-shop/routes/search.ts | High (both agree) |



### Strongest correlation deep-dive

1\. Vulnerable code (from Semgrep):

```typescript

// In routes/search.ts

const criteria = req.query.q;

...

models.Product.findAll({

&#x20; where: {

&#x20;   ...

&#x20;   name: { \[Op.like]: `%${criteria}%` }

&#x20;   ...

&#x20; }

})
```



2\. Working payload (from ZAP):

`q=%27%28` (URL-decoded: `'(`). This payload breaks out of the SQL string context in the `LIKE` clause, causing a SQL syntax error that ZAP detects as a SQL Injection vulnerability.

3\. The fix:

The fix is to use parameterized queries or Sequelize's built-in escaping properly. Instead of directly concatenating `criteria` into the `like` string, ensure the ORM handles it safely, or validate/sanitize the input strictly. For Sequelize, `Op.like` with template strings is generally safe if not concatenated with raw SQL, but here the input `criteria` should be sanitized to prevent breaking out.

4\. Why both tools caught it (1-2 sentences):

ZAP dynamically attacked the `/rest/products/search` endpoint by fuzzing the `q` parameter and observing the SQL error in the HTTP response. Semgrep statically analyzed the source code in `routes/search.ts` and identified that user input (`req.query.q`) flows directly into a Sequelize database query without proper sanitization.



### Reflection (2-3 sentences)

Lecture 5 slide 15 calls this "the highest-confidence finding type." In a real PR review, which of these two would you want first — the SAST finding or the DAST evidence — and why?

In a real PR review, I would want the SAST finding first because it pinpoints the exact line of code that needs fixing before the code is even deployed. DAST evidence is valuable for confirming exploitability in a running environment, but SAST allows developers to remediate the vulnerability early in the development lifecycle (shift-left), preventing it from ever reaching production.

