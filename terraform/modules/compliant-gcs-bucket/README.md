controls the module enforces: SC-12, SC-13, SC-28, AU-11, CM-6, AC-3.

SC-12 — Cryptographic Key Establishment and Management
Requires the organization to establish and manage cryptographic keys when cryptography is employed — covering key generation, distribution, storage, access, and destruction. The goal is ensuring keys are handled through their full lifecycle in a way that protects confidentiality and integrity of the systems relying on them. This directly maps to your key ring/crypto key work in the Terraform module — the key ring and rotation period (rotation_period = "7776000s", i.e. 90 days) you just deployed are concrete implementations of this control.

SC-13 — Cryptographic Protection
Requires implementing cryptographic mechanisms that meet applicable federal standards (typically FIPS-validated modules) to protect the confidentiality and/or integrity of information, based on the security category and applicable laws/policies. This is the "what algorithm/standard" control, whereas SC-12 governs "how you manage the keys" — they're usually implemented together.

SC-28 — Protection of Information at Rest
Requires protecting the confidentiality and integrity of information while it is stored (not in transit, not in use) — typically satisfied via encryption at rest, cryptographic hashes for integrity checking, or similar mechanisms. Your GCS bucket's default_kms_key_name encryption configuration is a direct implementation of SC-28.

AU-11 — Audit Record Retention
Requires retaining audit records for a defined time period to support after-the-fact investigation of incidents and to meet regulatory/organizational record-retention requirements. The bucket's retention_policy (30-day/2,592,000-second retention in your plan) is relevant here if that bucket stores audit/log data, though AU-11's retention period is usually driven by organizational policy rather than a fixed technical default.

CM-6 — Configuration Settings
Requires establishing and documenting mandatory configuration settings for information technology products (using organization-defined security checklists or benchmarks), implementing those settings, and monitoring/controlling changes to them. This is the broad "harden and standardize your configs" control — things like your uniform_bucket_level_access = true and public_access_prevention = "enforced" settings are CM-6 baseline configuration items.

AC-3 — Access Enforcement
Requires the system to enforce approved authorizations for logical access to information and system resources, per applicable access control policy (e.g., RBAC, ABAC). IAM bindings like the google_kms_crypto_key_iam_member resource in your plan — which grants the GCS service account the cloudkms.cryptoKeyEncrypterDecrypter role, and nothing broader — are a direct enforcement mechanism under AC-3.


How they cluster: SC-12/SC-13/SC-28 form the encryption trio (key management → crypto standard → data-at-rest outcome); CM-6 and AC-3 are baseline hardening and access-control controls that apply to nearly every resource you provision; AU-11 is retention-specific and only fully satisfied if the bucket in question is actually serving as your audit-log store with policy-driven retention (not just a default value).
