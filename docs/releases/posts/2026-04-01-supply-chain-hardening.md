---
date: 2026-04-01
slug: supply-chain-hardening
---

# npm supply chain hardening

In response to the March 30 Axios npm supply chain attack (CVE pending, attributed to North Korean threat actor UNC1069), we audited all dependencies and confirmed Thalian is not affected — axios is not in our dependency tree.

<!-- more -->

We've additionally hardened our build pipeline:

- npm audit now blocks deployments on high-severity findings
- postinstall scripts from transitive dependencies are disabled by default
- All dependency versions are pinned exactly
- Lockfile integrity validation has been added to CI

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-04-01)
