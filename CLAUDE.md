# Thalian Docs — Claude Code Instructions

This repo powers [docs.thalian.ai](https://docs.thalian.ai), built with **MkDocs Material** and deployed via Cloudflare Pages on `git push`.

---

## Page Structure

Every documentation page follows this exact structure:

```markdown
# Page Title

One-sentence or short paragraph describing what this page covers.

---

## First Section

Content...

## Second Section

Content...

---

*Footer line with cross-link to related page.*
```

### Rules:
- **One `#` heading** per page (the page title) — never more than one
- **`---` horizontal rule** immediately after the intro paragraph
- **`---` horizontal rule** before the footer line
- **Footer line** in italics with a cross-link to the most relevant related page, e.g.: `*For information on X, see [Page Name](./page-name.md).*`
- Sections use `##`. Subsections use `###`. Never go deeper than `###`
- No table of contents in the page body — MkDocs generates it automatically

---

## Typography & Formatting

### Text
- **Bold** for UI element names, feature names, and emphasis: `**Findings**`, `**Connect**`, `**MFA enforcement**`
- *Italics* for example finding sentences and the footer cross-link
- `inline code` for technical values: field names, API methods, config values, paths, scores, formulas
- No ALL CAPS except in actual UI labels that are capitalized

### Lists
- Use `-` for unordered lists (not `*`)
- Use `1.` numbered lists for sequential steps only
- Bold the lead term in definition-style lists: `- **Term:** Description`

### Tables
- Use `|---|---|` alignment (no colons for centering)
- Bold the first column when it's a label/name: `| **Okta** | API token | Users, groups... |`
- Keep tables compact — no extra whitespace padding
- Use tables for structured comparisons (plans, permissions, platforms). Use lists for everything else

### Links
- Internal cross-links use relative paths: `[Page Name](./page-name.md)`
- External links use full URLs: `[app.thalian.ai/signup](https://app.thalian.ai/signup)`
- Bold links in the index page: `- **[Page Name](page-name.md)** — Short description`

### Code blocks
- Use triple backticks with no language tag for formulas/pseudocode
- Use language-tagged blocks only for actual code snippets (rare in these docs)

### Admonitions (MkDocs Material)
- Available but used sparingly. Format:
  ```
  !!! warning "Title"
      Content indented 4 spaces.
  ```
- Only use for genuinely important callouts (deprecated features, security warnings)

---

## Content Conventions

### Voice & Tone
- Second person ("you", "your") — addressing IT admins directly
- Present tense, active voice
- Concise and direct — no filler, no marketing language
- Technical but not jargon-heavy — explain acronyms on first use

### Findings
- Always express example findings as italicized sentences in quotes:
  `*"Sarah Chen has admin access to Salesforce but hasn't logged into Okta in 45 days."*`
- Never show raw data table rows as finding examples

### UI References
- Reference pages by their sidebar name in bold: **Integrations**, **Findings**, **Reports**, **Settings**
- Include the route path in backticks on first mention: `` The Findings page (`/findings`) ``
- Describe navigation as: `Go to **Integrations** → **Browse**`

### Feature Descriptions
- Lead with what it does, not how it works internally
- Include plan availability where relevant (Free / Pro / Enterprise)
- Link to related pages rather than repeating content

---

## File Naming

- All lowercase, words separated by hyphens: `getting-started.md`, `findings-and-remediation.md`
- No underscores, no camelCase
- Name should match the nav label in `mkdocs.yml` (roughly)

---

## mkdocs.yml

When adding a new page:
1. Create the `.md` file in `docs/`
2. Add it to the `nav:` list in `mkdocs.yml`
3. Add a link to it from `index.md` under the appropriate section
4. Cross-link from the most related existing page's footer

Enabled extensions: `tables`, `admonition`, `pymdownx.details`, `toc` with permalinks.

---

## Screenshots

Screenshots are the only visual content in the docs — no diagrams, illustrations, or stock images. Every screenshot is a cropped capture of the actual Thalian UI from `demo.thalian.ai`.

### When to use
- To orient users on a page or feature they haven't seen before
- To show the result of a multi-step action (e.g., after connecting an integration)
- To clarify a UI element that's hard to describe in text alone

### When NOT to use
- For simple actions (clicking a button, toggling a filter)
- When text already explains the UI clearly
- For every section — screenshots should supplement, not replace, written instructions

### File conventions
- **Format:** PNG (sharp text on dark UI)
- **Max width:** 1200px
- **Location:** `docs/assets/screenshots/` — organized by section (`integrations/`, `findings/`, `settings/`, etc.)
- **Naming:** lowercase, hyphenated, descriptive: `findings-page-severity-filter.png`, `integrations-okta-connected.png`
- **Content:** Always use demo workspace data — never real customer data

### Markdown syntax
```markdown
![Short description of what the screenshot shows](./assets/screenshots/section/filename.png)
```

---

## What NOT to Do

- Don't use emoji in page content
- Don't nest sections deeper than `###`
- Don't use HTML in markdown files
- Don't repeat content across pages — cross-link instead
- Don't include internal implementation details (Supabase project IDs, API internals, worker names)
- Don't use placeholder or example data that implies fake workspace state
- **Don't use em dashes (`—`) or AI-sounding language** (`84c1188` swept these out — keep prose plain and direct)
- **Don't add "Oneleet" or any specific GRC vendor name** to public docs (kept vendor-agnostic in `dd7f995` on landing; same convention applies here)

---

## Sync With Other Repos

These docs aggregate state that ships in other repos. When any of the following changes upstream, update the corresponding doc here in the same release:

| Upstream change | Pages to update |
|------------------|-----------------|
| New rule or rule-count change in `thalian-prod` | `findings-and-remediation.md`, `index.md` (if hero count is shown), `changelog.md` |
| New integration | `integrations/<platform>.md`, `integrations/index.md` table, `integrations-guide.md`, `changelog.md` |
| Plan limits / data retention change | `getting-started.md`, `index.md`, relevant policy doc (privacy, DPA) |
| AI model change | `ai-transparency.md`, `data-processing-agreement.md` |
| New API endpoint | `api-reference.md` (full request/response schemas) |
| Compliance score formula change | `reports-and-audit.md`, `compliance.md` |

### Recent doc work (since last CLAUDE.md sync, 2026-04-05)

- **Homepage redesign** (`dbc8d3e`, `9686683`, `f76d958`) — card-grid layout on `index.md`; sidebars hidden on home page for full-width grid; CDN cache-busted via `custom.css` query string after deploy.
- **API reference expansion** (`245f368`) — full request/response schemas, plus accuracy fixes (Workspace ONE vendor name corrected in `a43e438`).
- **New integration guides:** GitLab (`12ecfba`), PingOne (`130266c`), Datadog (`4ff3157`).
- **Fleet integration guide expansion** (`d08b308`) — network requirements + remediation actions.
- **Changelog: rule counts** — bumped 316→320→341→390→400+ across April 2026 entries (`4fabf99`, `f0abb43`, `e2562c5`, `a0a984b`); accuracy audit caught Kandji→Iru rename.
- **April 2026 release post** (`ecbf400`) + multiple changelog entries: bulk app policy actions (`c73dace`), workspace risk score rebuild (`8472513`), Google-only SSO ratio fix (`1caa315`), webhook destination picker + event improvements (`02a824d`), MFA enforcement fix (`cff0446`), Compliance Trend multi-metric improvement (`9db6062`), posture timeline + coverage widget (`eee4b59`), attack surface map + grouped findings + geolocation + Slack App Directory (`b953c25`), bulk app policy actions (`6c8e12f`), Compliance Trend SOC 2 / ISO 27001 score updates (`9c1ca2d`), April 25 release post (`5675a4d`), Okta AI agent identity detection (`11f0707`), AI chat grouped finding context + `remove_admin_role` (`a01be76`).
- **Pro retention bump** (`f821017`) — 90 days → 1 year (mirrors product change).
- **Anthropic retention clarification + Enterprise model bump to Claude Opus 4.7** (`48ab9d6`) — keep `ai-transparency.md` synced with `thalian-prod` model pins.
- **Sub-processor registry + AI tool detection docs + developers page** (`a4a6d6a`).
- **Getting Help support hub** (`20efb...` / `20fffd8`) — new `support.md` page.
- **Em-dash and AI-language sweep** (`84c1188`) — style baseline.
- **MCP server v1.1.0 release post + changelog** (`4517e8e`) — documents 12-tool surface (6 query + 6 action), write-scope keys, OAuth `client_credentials` for Claude.ai + `authorization_code` with PKCE for human flows.
- **NHI / AI agent governance docs** (`ddb4c70`) — `changelog.md`, `compliance.md`, `okta.md`, `findings-and-remediation.md` updated for Okta AI Agents sync, `possible_ai_agent_unclassified` + `drift::ai_agent_growth` rules, NIST CSF 2.0 mapping. Follow-up (`ef24516`): NHI access review `scope_filter` (`nhi_only` / `ai_agents_only`), AI chat grouped findings context, `remove_admin_role` chat tool.
- **Remediation actions for 5 integrations** (`0e4243d`) — per-platform docs now describe the first write-capable remediation actions: PingOne (`suspend_user`/`unsuspend_user` via Workforce API), GitLab (`remove_from_group` via Group Access Token), Datadog (`suspend_user`/`unsuspend_user`), Outlook + SharePoint (`revoke_sessions` only — account-wide lifecycle blocked at executor; use Entra ID).
- **Compliance PDF export fix changelog** (`03759ac`) — May 2026 Fixes section captures the full controls table in the PDF export (was clipped before).
- **ISO 42001 framework + Compliance Trend multi-framework** (`1646921`) — `compliance.md` expanded from 2-framework to 4-framework coverage (SOC 2, ISO 27001, NIST CSF 2.0, ISO 42001). New "ISO 42001 coverage" section documents the seven Annex A controls (A.4.2, A.6.2.2, A.6.2.6, A.6.2.8, A.7.3, A.9.2, A.10.3) grouped by category plus practitioner guidance. Changelog adds ISO 42001 entry under New Features and Compliance Trend multi-framework plot under Improvements.
