\# Lab 6 — Submission



\## Task 1: Checkov on Terraform + Pulumi



\### Terraform scan

\- Total checks: 78

\- Passed: 0

\- Failed: 78



| Severity | Count |

|----------|------:|

| Critical | 0 |

| High | 0 |

| Medium | 0 |

| Low | 0 |

| Unspecified (null) | 78 |



\### Top 5 rule IDs (by frequency)

| Rule ID | Count | What it checks |

|---------|------:|----------------|

| CKV\_AWS\_289 | 4 | Ensure IAM policies does not allow data exfiltration |

| CKV\_AWS\_355 | 4 | Ensure no IAM policies allow `\*` resource |

| CKV\_AWS\_23 | 3 | Ensure every security groups rule has a description |

| CKV\_AWS\_288 | 3 | Ensure IAM policies does not allow write access without constraints |

| CKV\_AWS\_290 | 3 | Ensure IAM policies does not allow privilege escalation |



\### Pulumi scan

Scanned via KICS in Task 2 (Checkov does not natively scan Pulumi Python source without pre-rendered state).



\### Module-leverage analysis (Lecture 6 slide 17)

Looking at your top-5 Terraform rules, which ONE fix would eliminate the most findings if applied

at the module level? 

If the IAM module enforced least-privilege by default and rejected any policy containing `Action: "\*"` or `Resource: "\*"`, it would immediately resolve the findings for CKV\_AWS\_355, CKV\_AWS\_289, and CKV\_AWS\_288 (totaling 10 findings). This demonstrates that fixing the root cause at the module level closes multiple downstream findings simultaneously.



\---



\## Task 2: KICS on Ansible + Pulumi



\### Severity breakdown (Ansible)

| Severity | Count |

|----------|------:|

| HIGH | 3 |

| MEDIUM | 0 |

| LOW | 1 |

| INFO | 0 |



\### Top 5 KICS queries (by frequency)

| Query | Severity | Files |

|-------|----------|------:|

| Passwords And Secrets - Generic Password | HIGH | 6 |

| Passwords And Secrets - Password in URL | HIGH | 2 |

| Passwords And Secrets - Generic Secret | HIGH | 1 |

| Unpinned Package Version | LOW | 1 |



\### Checkov vs KICS — when to use which? (Lecture 6 slide 10)

\- One thing Checkov did \*\*better\*\* for the Terraform sample: Checkov has deep, graph-based understanding of Terraform state and cloud provider specifics, giving more precise AWS-focused rules (e.g., detecting IAM privilege escalation paths).

\- One thing KICS did \*\*better\*\* for the Ansible sample: KICS natively supports Ansible playbooks and inventory files out-of-the-box, seamlessly catching hardcoded secrets and misconfigurations, whereas Checkov is primarily focused on cloud IaC like Terraform.

\- (Optional) An example of a finding only ONE of them caught for the same resource type: KICS caught hardcoded secrets in the Ansible inventory file (`ansible\_password`), which Checkov would ignore as it doesn't scan Ansible inventory files by default.



\---



\## Bonus: Custom Checkov Policy



\### Policy file (paste full contents of labs/lab6/policies/my-custom-policy.yaml)

```yaml

metadata:

&#x20; id: CKV\_CUSTOM\_1

&#x20; name: "Ensure S3 bucket has object lock enabled"

&#x20; category: GENERAL\_SECURITY

&#x20; severity: MEDIUM

definition:

&#x20; and:

&#x20;   - cond\_type: attribute

&#x20;     resource\_types:

&#x20;       - aws\_s3\_bucket

&#x20;     attribute: object\_lock\_enabled

&#x20;     operator: equals

&#x20;     value: true

```

