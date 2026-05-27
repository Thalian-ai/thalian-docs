---
date: 2026-05-27
slug: device-side-ai-detection
---

# Device-side AI agent detection: device + identity + OAuth, joined

Thalian now detects AI tools installed on managed devices, joins them with corporate OAuth grants and identity classification, and surfaces three new compound rules that no identity-only tool can fire.

<!-- more -->

## What changed

Until this release, Thalian's AI tool detection lived on the OAuth surface — it knew that twelve people in your workspace had granted Cursor read access to their Drive, but it had no idea whether Cursor was actually installed on those people's laptops. That was a real gap. An AI tool with a corporate OAuth grant is a governed surface. An AI tool installed on a device with NO matching OAuth grant is the opposite — it's the user signed in with a personal `@gmail.com` account, sending company code to a vendor cloud that your DLP doesn't see.

We can now detect both, and the joins between them are where the wedge sits.

## How it works

Thalian's sync handlers for **Fleet**, **Jamf Pro**, **Iru** (formerly Kandji), and **Microsoft Intune** now pull per-device installed software inventory and classify each app against a closed catalog of 30+ AI tools — Cursor, Claude desktop, ChatGPT desktop, Windsurf, Continue, Sourcegraph Cody, GitHub Copilot, Codeium, Perplexity, Devin, Ollama, LM Studio, and others. Classified installs land in a new `device_apps` table with `last_seen` lifecycle tracking. Apps still installed have their `last_seen` advance on every sync; apps uninstalled longer than 30 days get purged automatically.

A new base rule, **AI tool installed on managed device**, fires per install with severity tiered by the tool's risk profile (local LLMs trigger higher severity because they ingest unbounded local context).

The interesting work is the three compound rules.

## The three compound rules

### AI tool on device with no corporate OAuth grant (medium)

Fires when an AI tool is installed on a device but the device's owner holds NO matching OAuth grant in any connected SaaS. The combination of "they have it" + "they didn't auth with the corporate account" is the personal-account-on-corporate-device pattern: vendor cloud egress with no DLP, no audit log, no SSO enforcement. The rule guards itself with an IDP-connected check (without an IDP, "no OAuth" is trivially true for everyone) and excludes local LLMs (covered by a more specific rule below).

If the user is using a sanctioned AI tool, severity drops to low and the remediation prompt asks whether the workspace's sanctioned-account policy explicitly permits personal-account use.

### AI tool footprint: device + corporate OAuth (informational)

The happy-path enrichment finding. Fires when a device install, identity classification, and corporate OAuth all align — proof that an AI tool is governed across surfaces for that identity. It's informational, doesn't inflate workspace risk score, but appears in the entity detail panel so you can see "Cursor on Tori's MacBook with corporate Cursor OAuth" as a positive signal rather than absence of risk.

### Local LLM on privileged user device (high)

Fires when Ollama or LM Studio is installed on a device whose owner has admin access in any IDP. Local LLMs don't send prompts to a vendor cloud, but they ingest whatever you feed them — code, credentials, customer data — and that context can leak through prompt logs, vendor telemetry, or runaway model behavior. Combined with admin privileges, the blast radius is whatever data the privileged identity can read across your environment.

Sanctioned local LLMs downgrade to medium with a remediation prompt asking whether the workspace has a documented data-handling policy for privileged-user local-LLM use.

## What you need to do

If you have Fleet, Jamf Pro, or Iru connected, you don't need to do anything. The next sync will populate `device_apps` and the rules will fire on the following analysis run.

If you have Microsoft Intune connected, you'll see a one-time **Software inventory** scope warning on the Integrations page until you reconnect Microsoft to grant `DeviceManagementApps.Read.All`. This scope was added to the standard Microsoft OAuth scope set as a natural extension of the existing `DeviceManagementManagedDevices.Read.All` grant (you already trusted us to see your managed devices; this is what's installed on them). Reconnect once and the warning clears — the reconnect itself writes an `integration_scope_extended` audit log entry for SOC 2 CC8.1 attribution. Until you reconnect, your other Microsoft integrations (Entra, Outlook, Teams, SharePoint) continue working at their existing scope.

## Compliance coverage

Each new rule maps to:

- SOC 2 **CC6.1** (Logical and Physical Access Controls), **CC9.1** (Risk Mitigation)
- NIST CSF 2.0 **PR.AA-01** (Identities and Credentials), **PR.AA-05** (Access Permissions), **ID.AM-01** (Hardware Inventory)
- ISO 42001 **A.4.2** (Resource Documentation), **A.7.3** (Acquisition of Data), **A.9.2** (Responsible Use)

These mappings keep your SOC 2 observation evidence and AI Management System (ISO 42001) control surface tight with the new detection surface.

## Why this matters

The cross-platform compound rules are the moat. Identity-only AI agent governance tools — Astrix, Oasis, Entro — can see your OAuth grants and your NHIs, but they can't see what's installed on your devices. MDM tools — Jamf, Intune — can see what's installed but have no view of OAuth grants or identity classification. The join is where the real exposure lives: a Cursor install on a senior engineer's MacBook with no matching corporate OAuth grant is a real, actionable, single-finding story. Neither end of the stack can tell it alone.

If you're running the AI tool detection surface today and want to see how device installs change the picture, connect your MDM and run a sync.
