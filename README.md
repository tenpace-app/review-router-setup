# Review Router Setup

[![Test](https://github.com/tenpace-app/review-router-setup/actions/workflows/test.yml/badge.svg)](https://github.com/tenpace-app/review-router-setup/actions/workflows/test.yml)

Create a GitHub-to-Slack Review Router connection from a workflow or AI coding session. The Action produces one short-lived browser link. Open it and sign in to Tenpace to claim and lock the exact repository, Slack destination, routing preset, and reviewer mention policy.

The Action never receives GitHub App credentials, Slack OAuth tokens, provider authorization codes, or the human's Tenpace session.

## Routing plans

Review Router Team includes MCP setup: **30 days free with a card**, then **$5 per developer/month** or **$50 per developer/year**. The subscription renews automatically; cancel in Billing before the trial ends to avoid a charge. There is no permanent free routing plan.

The **public review-health audit remains free** and does not require a card. [Review current plan details](https://review-router.tenpace.com/#plans).

If setup returns `awaiting_billing`, an owner or admin opens the returned billing link and completes secure checkout in the browser. Never give card details to a coding assistant. The Action and MCP server cannot authorize charges.

## Set up from your coding assistant (MCP)

[![Tenpace Review Router MCP server](https://mcprush.com/badge/tenpace-app-tenpace-review-router-mcp/card.svg)](https://mcprush.com/to/tenpace-app-tenpace-review-router-mcp?ref=badge)

Connect the hosted server, then ask it to propose the GitHub-to-Slack route:

```sh
claude mcp add --transport http review-router https://api.tenpace.com/v1/review-router/mcp
```

> Set up Review Router for OWNER/REPO in #code-reviews, with reviewer mentions.

Open the returned browser link, confirm the exact plan, and complete GitHub and Slack authorization. Verify the test message, then check that a real matching PR arrives. This MCP configures routing; it does **not** review code or grant unrestricted provider access.

**[MCP quickstart: Claude Code, Cursor, troubleshooting](https://review-router.tenpace.com/docs/mcp)** · [Illustrated workflow demo](https://review-router.tenpace.com/demo) · [Free review-health check](https://review-router.tenpace.com/tools/pr-review-health)

The canonical registry name is `io.github.tenpace-app/review-router`. Use the full `/v1/review-router/mcp` URL above. The shorter `/mcp` URL is reserved for a separate gateway, not direct clients.

## Reproduce a public review-health audit

This repository includes a [read-only example workflow](.github/workflows/review-health.yml). Open [its runs](https://github.com/tenpace-app/review-router-setup/actions/workflows/review-health.yml) to see the job summary and downloadable JSON. Maintainers can run it on demand; other teams can copy the workflow into their own repository. It does not create a route, post a comment, or send private repository data to Tenpace.

Small demo repositories may not have enough review history for every metric. Missing samples are not proof of a healthy or unhealthy team. Add the generated badge only if you want to publish your repository's live report.

## Use the Action

Create `.github/workflows/review-router-setup.yml`:

```yaml
name: Set up Review Router

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: tenpace-app/review-router-setup@v1
        with:
          slack-channel: pull-requests
          preset: repo-watch
          reviewer-mentions: all
```

Run the workflow. The same GitHub user who started it opens the **Continue Review Router setup** link in the job summary, signs in to Tenpace, completes the GitHub and Slack steps, and lets the page reconcile and verify the route with a real Slack delivery.

## Config as code

Instead of inputs, commit `.tenpace/review-router.yml`:

```yaml
version: 1
route:
  slack-channel: pull-requests
  preset: repo-watch
  reviewer-mentions: all
```

You can also set `slack-workspace` when the Tenpace team has more than one connected workspace. Explicit Action inputs override the file.

## Inputs

- `slack-channel`: Slack channel name, with or without `#`. Required unless present in the config file.
- `slack-workspace`: Optional exact workspace name, Slack team ID, or Tenpace workspace ID.
- `preset`: `repo-watch`, `full-coverage`, or `team-activity`. Default: `repo-watch`.
- `reviewer-mentions`: `all`, `connected-only`, or `none`. Default: `all`.
- `config`: Config path. Default: `.tenpace/review-router.yml`.

## Outputs

- `setup-id`: Public setup-session ID.
- `setup-url`: Short-lived human authorization URL.
- `status`: Initial status, normally `awaiting_human`.

The machine setup token is masked and deliberately not exposed as an Action output. The displayed URL contains no claim secret: it is bound to the GitHub actor in the signed OIDC token, and that same GitHub user must open it. The browser flow can finish setup without putting a secret into another workflow step.

## Security model

GitHub Actions OIDC proves the repository and actor for the workflow run. It does not grant provider authority. Tenpace verifies the OIDC issuer, signature, audience, time window, repository and owner IDs, actor ID, subject, and repository binding. Action URLs contain no claim secret and can only be claimed by that GitHub actor.

The server creates an inert intent. A Tenpace user signs in and claims the setup URL, which atomically locks that exact intent version. The machine token cannot edit after claim; the person who claimed the setup can edit it, and saving creates and locks the new version in the same action. Provider OAuth stays in the browser and encrypted credentials stay on Tenpace.

Setup creation and application are idempotent. The final check sends one message through the same Slack delivery path used for pull request notifications.

See [SECURITY.md](SECURITY.md) for reporting and operational details.

## For coding agents

Read [AGENTS.md](AGENTS.md) before automating setup. Canonical machine-readable surfaces:

- [OpenAPI 3.1](https://api.tenpace.com/openapi.json)
- [Remote MCP](https://api.tenpace.com/v1/review-router/mcp)
- [MCP Registry metadata](server.json)
- [Agent descriptor](https://api.tenpace.com/.well-known/tenpace-agent.json)
- [Raw Markdown guide](https://review-router.tenpace.com/docs/agent-setup.md)
- [llms.txt](https://review-router.tenpace.com/llms.txt)

No AI-discovery convention is universal, so the same workflow is described in the Action metadata, README, AGENTS.md, linked HTML/Markdown docs, OpenAPI, MCP, and well-known descriptor.

The remote MCP server is also published to the official MCP Registry as

`io.github.tenpace-app/review-router`

## License

MIT
