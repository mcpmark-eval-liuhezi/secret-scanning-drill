# Secret Scanning — Training Notes

> Training material for a security-awareness session on **leaked credentials**.
> Written from the current official GitHub documentation — links at the bottom.
> Nothing in this repository is a real secret.

---

## 1. What secret scanning does

GitHub secret scanning continuously scans a repository for exposed credentials so that a leaked API key, password, or token is flagged before it can be exploited.

Key points:

- It searches for **known, supported secret types**, and for provider-specific secrets it also notifies the provider (partner) that issued them.
- It scans **more than code**: issue titles, descriptions, and comments (open and closed historical), pull requests, GitHub Discussions, wikis, and secret gists, because that is where plaintext credentials often get pasted.
- Detection spans the **entire Git history on all branches**, not just the current default branch — secrets committed long ago and later "deleted" still surface.
- It runs **automatically and free of charge on public repositories**, which is why this demo repo can rely on it; organization-owned private/internal repos require Secret Protection enabled on Team or Enterprise Cloud, and user-owned repos require Enterprise-level Secret Protection.
- Detection is **periodic**: GitHub periodically rescans so newly supported secret types are picked up.

## 2. How alerts are surfaced and resolved

### What the alerts look like

Three kinds of alerts exist:

| Alert type | Where it shows up | When |
| --- | --- | --- |
| **User alerts** | Repository's **Security and quality** tab | A supported secret is detected in the repository |
| **Push protection alerts** | Repository's **Security and quality** tab | A contributor bypasses push protection |
| **Partner alerts** | Sent **directly to the secret provider** — not shown in your repo's alerts | A provider-owned secret is detected |

### How detection families differ

Patterns fall into three categories: **generic** (regex-based, provider-independent, e.g. private keys and connection strings), **AI-detected** (passwords and other unstructured secrets found with a model, e.g. `password`), and **provider** (regex bound to a specific service such as AWS or Stripe). Capabilities vary by family: partner notifications exist only for provider secrets; push protection covers generic patterns and most provider patterns; validity checks, extended metadata, and Base64 support apply to some provider and generic patterns.

### How alerts are resolved

1. **Rotate the affected credential first** — treat it as compromised; this is the real fix.
2. **Remove the secret from the code** — but deleting a file does **not** remove it from Git history. Rewriting history is possible but often unnecessary once the credential has been revoked.
3. **Use validity checks** to determine whether a detected secret is still live, so you can triage active credentials before stale ones.
4. **Tune detection as needed**: enable generic and AI-detected patterns, add custom regex patterns for organization-specific formats not covered by defaults, turn on validity and metadata checks, and optionally let AI help generate the regular expressions for custom patterns.

## 3. Example secret types it detects

The reference list of supported secret types (Security and code quality → Supported secret scanning patterns) includes hundreds of patterns; this is a representative sample:

### Generic patterns (provider-independent)

| secret_type | Description |
| --- | --- |
| `rsa_private_key` | RSA private keys |
| `openssh_private_key` | OpenSSH-format private keys used for SSH authentication |
| `pgp_private_key` | PGP private keys used for encryption and signing |
| `mongodb_connection_string` | MongoDB connection strings containing credentials |
| `mysql_connection_url` | MySQL connection strings containing credentials |
| `postgres_connection_string` | PostgreSQL connection strings containing credentials |
| `http_basic_authentication_header` | HTTP Basic Authentication credentials in headers |
| `http_wearer_authentication_header` | HTTP Bearer tokens used for API authentication |

*Validity checks are **not** supported for generic patterns.*

### AI-detected patterns

| secret_type | Description |
| --- | --- |
| `password` | Passwords and other unstructured secrets, detected using AI |

*Push protection and validity checks are **not** supported for the `password` pattern.*

### Provider patterns (sample)

- **Amazon AWS** — Access Key ID, API Key, Session Token
- **Adobe** — Client Secret, Device Token, PAC Token, Refresh Token, Service Token, Short-Lived Access Token
- **Anthropic** — Admin API Key, API Key, Service Account API Key, Session ID, User API Key
- **Stripe**, plus dozens more

The full reference table lists patterns across providers with per-pattern flags for partner alerts, user alerts, push protection, validity checks, metadata checks, and Base64 support.

## 4. Documentation references

- Overview of how secret scanning works: https://docs.github.com/en/code-security/secret-scanning/searching-for-and-prioritizing-secret-scanning-alerts
- Reference list of supported secret scanning patterns: https://docs.github.com/en/code-security/secret-scanning/supported-secret-scanning-patterns
- Enabling secret scanning for your repository: https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-for-your-repository

## 5. Recap for the walkthrough

1. A committed secret is a compromised secret — rotate first, delete second.
2. Git history keeps secrets forever; revocation, not deletion, is the fix.
3. Leakage is multi-channel: code, issues, PRs, Discussions, wikis, and gists are all scanned.
4. Push protection is the prevention layer; secret scanning is the detection layer.
5. Keep detectivity high: enable generic, AI-detected, custom, and validity-checked patterns.
