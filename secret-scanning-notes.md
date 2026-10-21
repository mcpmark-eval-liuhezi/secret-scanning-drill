# Secret Scanning — Training Notes

> Training material for our security-awareness session on **leaked credentials**.
> Summary written from the current GitHub documentation (see links at the bottom).
> Nothing in this repository is a real secret.

---

## 1. What secret scanning does

Secret scanning exists to stop one very common failure mode: an API key, password, or token
that gets hardcoded into a repository and committed, turning it into a target for
unauthorized access the moment it lands in version control.

- **Detects hardcoded credentials** — API keys, passwords, tokens, and other known secret types.
- **Scans your entire Git history**, on all branches — not just the current state of the default branch.
  This means secrets committed long ago and "deleted" later are still found.
- **Periodically rescans** repositories when new secret types are added to GitHub's detection list.
- **Scans more than just code.** GitHub also automatically scans:
  - Titles, descriptions, and comments in issues (including open and closed historical issues)
  - Titles, descriptions, and comments in pull requests
  - Titles, descriptions, and comments in GitHub Discussions
  - Wikis
  - Secret gists
- The point is to surface **secret sprawl** — the uncontrolled spread of credentials across
  repositories — before it becomes an incident.

### Availability (why this repo is public)

- **Public repositories:** secret scanning runs automatically, for free. (This demo repo relies on that.)
- **Organization-owned private/internal repositories:** requires GitHub Secret Protection,
  enabled on GitHub Team or GitHub Enterprise Cloud.
- **User-owned repositories:** available on GitHub Enterprise Cloud with Enterprise Managed Users,
  or GitHub Enterprise Server when the enterprise has Secret Protection enabled.

---

## 2. How alerts are surfaced

There are **three kinds of alerts**:

| Alert type | Where it shows up | When |
| --- | --- | --- |
| **User alerts** | The repository's **Security and quality** tab | A supported secret is detected in the repository |
| **Push protection alerts** | The repository's **Security and quality** tab | A contributor bypasses push protection |
| **Partner alerts** | Sent **directly to the secret provider** — not shown in your repo's alerts | A secret belonging to a partner provider is detected |

The partner integration is worth calling out to the team: GitHub works with a large range of
service providers, and when a partner secret is detected, the provider is notified so they can
take action (e.g. revoking the credential). Those alerts bypass your dashboard entirely.

### Detection categories

Patterns fall into three categories:

| Category | Description | Approach | Example `secret_type` |
| --- | --- | --- | --- |
| **Generic** | Secrets not tied to a specific provider — private keys, database connection strings | Regex | `rsa_private_key` |
| **AI-detected** | Passwords and other unstructured secrets | AI model (Copilot) | `password` |
| **Provider** | Secrets tied to a specific service provider (AWS, Azure, Stripe, …) | Regex | `aws_access_key_id` |

Not every capability applies to every category: partner notifications only exist for provider
secrets (where the provider is a partner); push protection covers generic patterns and *most*
provider patterns; validity checks, extended metadata, and Base64 support apply to some
provider patterns and generic patterns.

---

## 3. How alerts get resolved

When secret scanning detects a leaked credential, GitHub generates an alert in the
**Security and quality** tab with details about the exposed credential.

The remediation flow:

1. **Rotate the affected credential first.** This is the real fix — treat the leaked secret as compromised.
2. Remove the secret from your code — but understand that **removing it from a file does not remove
   it from Git history**. Rewriting history is time-intensive, and it's often unnecessary once the
   credential has been revoked.
3. **Validity checks** can help you prioritize: when enabled, secret scanning may contact the
   issuing service to determine whether a detected secret is still active, so you can deal with
   live credentials before stale ones.

### Customizing detection

- **Generic patterns** — expand detection to secrets not tied to a specific provider (private keys,
  connection strings, generic API keys).
- **Custom patterns** — define your own regular expressions for organization-specific secret formats
  that default patterns don't cover.
- **Validity checks** — verify whether detected secrets are still active.
- **AI-detected secrets** — catch unstructured secrets like passwords.
- **AI-generated regular expressions** — have AI draft the regex for your custom patterns.

---

## 4. Example secret types it detects

### Generic patterns (provider-independent)

| Token | Description |
| --- | --- |
| `ec_private_key` | Elliptic Curve (EC) private keys |
| `generic_private_key` | Private keys with a `-----BEGIN PRIVATE KEY-----` header |
| `rsa_private_key` | RSA private keys |
| `openssh_private_key` | OpenSSH-format private keys used for SSH authentication |
| `pgp_private_key` | PGP private keys used for encryption and signing |
| `mongodb_connection_string` | MongoDB connection strings containing credentials |
| `mysql_connection_url` | MySQL connection strings containing credentials |
| `postgres_connection_string` | PostgreSQL connection strings containing credentials |
| `http_basic_authentication_header` | HTTP Basic Authentication credentials in headers |
| `http_bearer_authentication_header` | HTTP Bearer tokens used for API authentication |

*Note: validity checks are **not** supported for generic patterns.*

### AI-detected patterns

| Token | Description |
| --- | --- |
| `password` | Passwords and other unstructured secrets, detected using Copilot |

*Note: push protection and validity checks are **not** supported for passwords.*

### Provider patterns (531 supported at time of writing)

A small sample of the providers covered:

- **1Password** — Service Account Token
- **Adafruit** — IO Key
- **Adobe** — Client Secret, Device Token, PAC Token, Refresh Token, Service Token, Short-Lived Access Token
- **Aikido** — API Client Secret, CI Scanning Token
- **Airtable** — API Key, Personal Access Token
- **Aiven** — Auth Token, Service Password
- **Alibaba Cloud** — AccessKey ID / secret
- **Amazon AWS** — Access Key ID, API Key, Session Token
- **Anthropic** — Admin API Key, API Key, Service Account API Key, Session ID, User API Key
- **Apify** — Actor Run API Token
- …and many more. The full reference table lists **531 patterns** across providers such as
  AWS, Azure, and Stripe, with per-pattern flags for partner alerts, user alerts, push
  protection, validity checks, metadata checks, and Base64 support.

---

## 5. Talking points for the session

1. **A committed secret is a compromised secret.** Treat any credential that has been pushed to a
   repository — especially a public one — as public knowledge. Rotate first, investigate second.
2. **Deleting the file doesn't delete the secret.** Git history keeps it forever. Rotation, not
   deletion, is the fix.
3. **Secrets leak outside code too.** Issues, PRs, Discussions, wikis, and gists are all scanned
   because they're all places people paste things.
4. **Push protection is the prevention layer.** Secret scanning finds what's already there;
   push protection blocks the secret before it gets pushed.
5. **Know your plan.** Automatic free scanning applies to public repositories only; private repos
   need Secret Protection on a paid plan.

---

## Documentation sources

These notes were written from the official GitHub documentation:

- **Secret scanning (overview):** https://docs.github.com/en/code-security/concepts/secret-security/secret-scanning
- **Supported secret scanning patterns (reference):** https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns
