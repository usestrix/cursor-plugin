---
name: strix-scan
description: Scan the code in this workspace, or a deployed app, for security vulnerabilities with Strix
---

# Scan with Strix

Run a Strix security scan with the run-scan skill.

- If the user named no target, scan the source code of the open workspace.
- If the user named a domain or a connected repository, scan that target.
- Show the files to upload and the scan configuration, then wait for the user to confirm before you upload or start a scan.
- When the scan finishes, list the findings by severity and offer to fix the most severe one with the fix-finding skill.
