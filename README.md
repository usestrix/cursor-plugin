# Strix for Cursor

Strix runs AI penetration tests and code security reviews. With this plugin, the Cursor agent can scan the code that you write, show the vulnerabilities that Strix finds, fix them in your workspace, and ask Strix to retest the fix.

## What the plugin contains

- **MCP server.** The plugin connects Cursor to the hosted Strix MCP server at `https://app.strix.ai/mcp`. The server gives the agent tools for scans, findings, retests, PR reviews, assets, and schedules.
- **Command.** `/strix-scan` scans the open workspace, or a domain or repository that you name.
- **Skills.** The skills tell the agent how to use the Strix tools:
  - `run-scan` starts a scan of the workspace, a connected repository, or a verified domain.
  - `triage-findings` groups the open findings and proposes which to fix first.
  - `fix-finding` fixes one finding in your code and asks Strix to retest it.
  - `pr-review-status` shows Strix security reviews of your pull requests.
  - `security-report` writes a security report for a period.

## Requirements

- A Strix account. To make an account, go to [app.strix.ai](https://app.strix.ai).
- Strix credits or a Strix plan. Each scan spends credits.

## Sign in

1. Install the plugin from the Cursor Marketplace.
2. Open **Cursor Settings → MCP**.
3. Find the `strix` server and click **Connect**.
4. Sign in to Strix in the browser window that opens. Cursor stores the OAuth token.

The plugin does not store a secret. Cursor gets the token from the Strix OAuth flow.

## Scan your workspace

Type `/strix-scan` in the agent chat, or ask the agent to scan the project with Strix.

1. The agent makes a list of the source files in the workspace. It leaves out Git-ignored files, hidden files, dependency and build directories, secret-like files such as `.env`, and nested archives.
2. The agent shows you the file summary and the scan configuration. It does not upload or start a scan until you confirm.
3. After you confirm, the agent uploads a ZIP file of the source to Strix and starts a code review.
4. When the scan finishes, the agent lists the findings by severity. Ask the agent to fix a finding, and then to retest it.

The workspace does not need to be on GitHub. Uploads have these limits: 20,000 files, 25 MiB for one file, 250 MiB in total, and 50 MiB for the ZIP file.

## Scan a deployed app

Ask the agent to scan the domain of your app, for example `app.example.com`. Strix must verify that you own the domain before it can test the domain. The agent can register the domain and show you the verification steps.

Strix runs in the cloud. It cannot reach an app that runs only on `localhost`. Deploy the app to a domain that you own, and then scan that domain.

## Safety

- Scan only the targets that you own or have permission to test.
- A scan spends Strix credits and sends traffic to the targets.
- The agent asks for your confirmation before each scan, upload, retest, or status change.

## Test the plugin locally

1. Copy or link this repository to `~/.cursor/plugins/local/strix`.
2. Restart Cursor.
3. Open **Cursor Settings → Plugins** and make sure that Strix shows.

## Support

- Strix documentation: [docs.app.strix.ai](https://docs.app.strix.ai)
- Strix website: [strix.ai](https://strix.ai)

## License

[Apache-2.0](LICENSE)
