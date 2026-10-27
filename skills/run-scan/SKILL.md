---
name: run-scan
description: >
  Start or rerun a Strix penetration test or code security review. Use when
  the user says "scan this project with Strix", "pentest this", "scan
  example.com with Strix", "run a security scan on the repo", "rerun the
  last scan", or asks for a scan schedule. Covers the open workspace, a
  connected repository, and a verified domain.
---

# Run a Strix scan

Propose a scan configuration, get confirmation, and start the scan.

## When

- The user wants a code security review of the open workspace or of a connected repository.
- The user wants a penetration test of a deployed web app or API.
- The user wants a repeat of an earlier scan or a recurring schedule.

Do not use this skill to review a pull request (pr-review-status) or to retest one finding (fix-finding).

## Pick the target

- If the user says "this project", "my code", or names no target, the target is the source code of the open workspace. Follow "Scan the open workspace".
- If the user names a domain or a connected repository, follow "Scan a domain or a repository".
- Strix runs in the cloud and cannot reach `localhost`. If the user wants to test an app that runs only on the local machine, tell the user to deploy it and verify its domain. Offer a code review of the workspace in the meantime.

## Scan the open workspace

The workspace does not need to be on GitHub. You package the source as a ZIP file, upload it to Strix, and start a `code_review` scan on it.

1. Find the workspace root. In a Git repository, use `git rev-parse --show-toplevel`.
2. Make the file list. In a Git repository, run `git ls-files --cached --others --exclude-standard` so that ignored files stay out. Outside Git, list all regular files.
3. Remove these entries from the list:
   - hidden files and directories, for example `.git`, `.vscode`, and `.cursor`
   - dependency and build output, for example `node_modules`, `vendor`, `venv`, `.venv`, `__pycache__`, `dist`, `build`, `.next`, `target`, and `coverage`
   - secret-like files, for example `.env*`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `*.pfx`, and cloud credential files
   - nested archives, for example `*.zip`, `*.tar`, `*.gz`, and `*.jar`
4. Check the limits: at most 20,000 files, 25 MiB for one file, and 250 MiB in total. If the list is over a limit, ask the user which directories to leave out.
5. Show the user a summary: the number of files, the top-level directories, and the secret-like files that you removed. Include the scan configuration from "Confirm and start" in the same message. Wait for the user to confirm.
6. After confirmation, write the ZIP file to a temporary directory outside the workspace. Use `zip -q -X <tmp>/strix-source.zip -@` with the file list on standard input. If `zip` is not available, use Python `zipfile` or PowerShell `Compress-Archive`. If the ZIP file is larger than 50 MiB, stop and ask the user to narrow the file list.
7. Call `request_upload` with `file_name` `strix-source.zip`, `file_size` in bytes, and `category` `repository`.
8. Upload the bytes with `curl` (`curl.exe` on Windows). Do not print the upload token:
   ```bash
   curl --fail -X PUT "<signed_url>" \
     -H "Authorization: Bearer <token>" \
     -H "Content-Type: application/zip" \
     --upload-file <tmp>/strix-source.zip
   ```
9. Call `complete_upload` with the `upload_id`. Strix validates the archive in this step.
10. Call `create_scan` once with `upload_ids` set to `[upload_id]`. If the user also gave a verified domain, add its `domain_ids` and Strix runs a `live_test` with the source as white-box context.
11. Delete the local ZIP file. If `create_scan` fails with a 4xx error, call `delete_upload` with the `upload_id`.

## Scan a domain or a repository

1. Call `list_domains` and `list_repositories` to find assets that match the target. If nothing matches, show the available assets and ask which to use.
2. If the target is not registered, offer to call `create_domain` (arguments `domain` and `asset_type`) or `create_repositories`. `create_repositories` takes a `body` argument that holds one repository object, or an array of them, each with `full_name` and optional `provider` and `installation_id`. A new domain must pass `verify_domain` before a scan can run against it.
3. If a similar scan exists, call `get_scan_template` on it and reuse its configuration. For an exact repeat, prefer `rerun_scan`.
4. Follow "Confirm and start".

## Confirm and start

1. Propose the configuration: targets, engagement type, scan tier, and scope notes. Tell the user that the scan spends credits and sends traffic to the targets. Call `get_credit_balance` when the balance matters.
2. Wait for the user to confirm. Do not upload source or call `create_scan` before the user confirms.
3. After confirmation, call `create_scan` once with an `Idempotency-Key`. Report the scan id and the link to follow progress.
4. To follow progress, call `get_scan`. Call `list_scan_agents` and `get_scan_agent_trace` when the user wants to see what the agents did. Call `send_scan_message` to give a running scan extra context, for example test credentials that already exist as a test user.
5. When the scan finishes, use the triage-findings skill to present the results.

## Schedules

- For recurring scans, call `create_schedule`. Put the schedule definition in the `body` argument: every `create_scan` field plus `cron_expression` and `timezone`. Set `also_run_now` in `body` only when the user wants a scan immediately, and confirm first because that scan spends credits.
- Call `list_schedules`, `update_schedule`, and `trigger_schedule` to manage existing schedules.

## Rules

- One confirmation per scan. Never start a scan in a loop.
- Never scan a target that the user does not own or is not authorized to test.
- Never upload a file that the user did not see in the summary.
- Use `cancel_scan` only when the user asks to stop a scan.
