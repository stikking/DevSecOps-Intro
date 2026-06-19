\# Lab 4 — Submission



\## Task 1: Syft + Grype on Juice Shop



\### SBOM stats

\- `juice-shop.cdx.json` component count: 1846

\- `juice-shop.cdx.json` size: 1504963 bytes (\~1.43 MB)

\- `juice-shop.spdx.json` component count: 911



\### Grype severity breakdown

| Severity | Count |

|----------|------:|

| Critical | 3 |

| High | 42 |

| Medium | 39 |

| Low | 22 |

| Negligible | 0 |

| \*\*Total\*\* | 106 |



\### Top 10 CVEs

| CVE | Severity | Package | Installed | Fix |

|-----|----------|---------|-----------|-----|

| CVE-2023-46233 | CRITICAL | crypto-js | 3.3.0 | 4.2.0 |

| CVE-2019-10744 | CRITICAL | lodash | 2.4.2 | 4.17.12 |

| GHSA-5mrr-rgp6-x4gr | CRITICAL | marsdb | 0.6.11 | null |

| CVE-2026-45447 | HIGH | libssl3t64 | 3.5.5-1\~deb13u2 | 3.5.6-1\~deb13u2 |

| NSWG-ECO-428 | HIGH | base64url | 0.0.6 | >=3.0.0 |

| CVE-2020-15084 | HIGH | express-jwt | 0.1.3 | 6.0.0 |

| CVE-2022-23539 | HIGH | jsonwebtoken | 0.1.0 | 9.0.0 |

| NSWG-ECO-17 | HIGH | jsonwebtoken | 0.1.0 | >=4.2.2 |

| CVE-2022-23539 | HIGH | jsonwebtoken | 0.4.0 | 9.0.0 |

| NSWG-ECO-17 | HIGH | jsonwebtoken | 0.4.0 | >=4.2.2 |



\### Fix-available rate

Out of the top 10 CVEs, 9 have fixes available. This means that prioritizing patches by fix-availability and severity >= HIGH is crucial to quickly eliminate the most exploitable risks with existing updates.



\---



\## Task 2: Trivy Comparison



\### Side-by-side counts

| Severity | Grype | Trivy | Δ |

|----------|------:|------:|--:|

| Critical | 3 | 5 | 2 |

| High | 42 | 43 | 1 |

| Medium | 39 | 39 | 0 |

| Low | 22 | 22 | 0 |

| \*\*Total\*\* | 106 | 109 | 3 |



\### Why the difference?

Pick \*\*two specific CVEs\*\* that ONE tool found and the other didn't. For each:

1\. CVE-2015-9235 (jsonwebtoken) - Found by Trivy, missed by Grype. Reason: Different CVE database refresh cadence and matching rules.

2\. CVE-2022-25881 (http-cache-semantics) - Found by Trivy, missed by Grype. Reason: Different fix-version awareness in the vulnerability databases.



\### When would you pick each?

\- When does Syft+Grype's \*\*decoupled\*\* model win? It wins when you need to generate an SBOM once and scan it multiple times over time as new CVEs are published, or when attaching the SBOM as an attestation (Lab 8) for compliance without needing the original image.

\- When does Trivy's \*\*all-in-one\*\* win? It wins in a standard CI pipeline where you need a fast, comprehensive scan of the image directly, including IaC, secrets, and misconfigurations in a single step.



\---



\## Bonus: Sign-Ready SBOM for Lab 8



\### CycloneDX schema version

\- `specVersion`: 1.6

\- `bomFormat`: CycloneDX



\### Image digest captured

\- `docker inspect ... RepoDigests`: bkimminich/juice-shop@sha256:fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0



\### Attestation predicate (paste first 30 lines of juice-shop-attestation.json)

{

&#x20; "\_type": "https://in-toto.io/Statement/v1",

&#x20; "subject": \[

&#x20;   {

&#x20;     "name": "bkimminich/juice-shop:v20.0.0",

&#x20;     "digest": {

&#x20;       "sha256": "fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0"

&#x20;     }

&#x20;   }

&#x20; ],

&#x20; "predicateType": "https://cyclonedx.org/bom/v1.5",

&#x20; "predicate": {

&#x20;   "$schema": "http://cyclonedx.org/schema/bom-1.6.schema.json",

&#x20;   "bomFormat": "CycloneDX",

&#x20;   "specVersion": "1.6",

&#x20;   "serialNumber": "urn:uuid:3029aa91-17f9-4f6f-a046-28dd913de777",

&#x20;   "version": 1,

&#x20;   "metadata": {

&#x20;     "timestamp": "2026-06-19T20:12:27+03:00",

&#x20;     "tools": {

&#x20;       "components": \[

&#x20;         {

&#x20;           "type": "application",

&#x20;           "author": "anchore",

&#x20;           "name": "syft",

&#x20;           "version": "1.45.1"

&#x20;         }

&#x20;       ]

&#x20;     },

&#x20;     "component": {



\### What this enables in Lab 8

Cosign signs the binding between the image (its sha256 digest) and the SBOM. This proves that the SBOM was generated specifically for this version of the image and has not been tampered with or swapped, providing a verifiable provenance attestation.

