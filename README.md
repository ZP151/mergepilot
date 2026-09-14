<p align="center">
  <img src="apps/desktop/src/assets/mergepilot-icon.png" width="96" alt="MergePilot logo" />
</p>

<h1 align="center">MergePilot</h1>

<p align="center">
  <strong>From local changes to verified Azure DevOps delivery.</strong><br />
  A local-first desktop copilot for work items, pull requests, CI investigation, and approved delivery actions.
</p>

<p align="center">
  <a href="https://github.com/ZP151/mergepilot/releases/latest">Download for Windows</a> ·
  <a href="#quick-start">Quick start</a> ·
  <a href="#production-readiness">Production readiness</a> ·
  <a href="docs/product/README.md">Product docs</a> ·
  <a href="CONTRIBUTING.md">Contributing</a>
</p>

<p align="center">
  <a href="https://github.com/ZP151/mergepilot/releases"><img src="https://img.shields.io/github/v/release/ZP151/mergepilot?display_name=tag&sort=semver" alt="Latest release" /></a>
  <a href="https://github.com/ZP151/mergepilot/actions/workflows/ci.yml"><img src="https://github.com/ZP151/mergepilot/actions/workflows/ci.yml/badge.svg?branch=main" alt="CI" /></a>
</p>

MergePilot helps Azure DevOps developers and tech leads connect local code to
the work item, review discussion, build, and test evidence behind a delivery
decision. Inspect the evidence, review a proposed action, approve its scope,
and check the result against Git or Azure DevOps.

**Release baseline: v0.5.32.** The installed developer workflow has recorded
acceptance evidence. Broader team adoption and deployment workflows remain
subject to the [production readiness conditions](#production-readiness) below.

## Why MergePilot

Preparing a PR or investigating a failed build often means reconstructing the
same context across an editor, Git, Azure Boards, Repos, and Pipelines.
MergePilot brings that investigation and the next action into one workspace.

- **Work with the whole delivery context.** Connect repository changes to work
  item intent, PR discussions, policies, builds, and tests.
- **Make decisions from inspectable evidence.** See the target and relevant
  revisions before deciding what to do next.
- **Keep control of writes.** Review explicit proposals for supported Git and
  Azure DevOps mutations before execution.
- **Verify the outcome.** Action records distinguish execution from successful
  verification after an authoritative re-read.

Azure DevOps remains the system of record. MergePilot focuses on the path from
a concrete blocker to a verified next action. Full Boards administration,
autonomous production approvals, and a user-managed connector marketplace are
outside the [product scope](PRODUCT.md).

## What you can do

| Workflow                | In the workspace                                                                                             | Outcome                                                      |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| Understand a work item  | Inspect its description, acceptance criteria, linked PRs, and build evidence before starting an action.      | Begin work with traceable requirements and context.          |
| Prepare a pull request  | Investigate local changes, source/target branch divergence, and remote policy through guided PR preparation. | Review an editable proposal before creating a PR.            |
| Review a change         | Examine diffs, discussions, policies, and pipeline evidence together.                                        | Concentrate human review on the change and its risks.        |
| Investigate CI failures | Inspect run evidence and test results; request a recovery proposal when appropriate.                         | Separate read-only diagnosis from approved execution.        |
| Close the delivery loop | Approve a supported update, execute it, and re-read Git or Azure DevOps.                                     | Check the resulting state and recorded verification outcome. |

### How it works

```text
Local repository + Azure DevOps context
                  |
           Inspect evidence
                  |
          Propose next action
                  |
      Policy check + your approval
                  |
               Execute
                  |
    Re-read Git / Azure DevOps state
                  |
        Record verified outcome
```

**Context** is the shared Project Link selector: it connects a local workspace
to an Azure DevOps organization, project, and repository. **Work**, **Changes**,
and delivery views use that context; the agent handles investigation and
approved actions. Settings holds account, model, capability, and diagnostic
controls.

## Quick start

### Install on Windows

1. Download the Windows installer from
   [GitHub Releases](https://github.com/ZP151/mergepilot/releases/latest).
   Windows x64 is the current installer target.
2. Check the release notes and publisher signature before installation. The
   release workflow can publish unsigned internal-test artifacts when signing
   secrets are absent; see [Windows code signing](docs/windows-code-signing.md).
3. Install and launch MergePilot. The packaged app includes its local daemon;
   Node.js, pnpm, and Rust are needed only for source development.
4. Configure your model and Microsoft account, then select a Project Link in
   **Context** using the steps below.

### Connect your account and model

| Requirement           | Setup                                                                                                                                                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Repository            | Select a local Git checkout and map it to the intended Azure DevOps repository in Context. Git operations require Git and access to the checkout.                                                                                 |
| Azure DevOps identity | Sign in with a Microsoft account that can access the target organization and project. Under Settings → Account → Advanced Azure auth, configure your organization's tenant and application client ID when required.               |
| Model access          | Configure a supported Azure OpenAI or OpenAI model in Settings and test the connection. Azure configuration needs the endpoint, deployment, and credential. Provider access and billing are supplied by you or your organization. |
| Secrets               | Use the configured Key Vault integration or local environment secret storage. Settings → Account → Model secrets selects the built-in model's secret source.                                                                      |
| Write permissions     | Your Azure DevOps permissions and policies still apply. Enable approved remote writes only for the intended project and workflow.                                                                                                 |

For organizational setup, see the
[Azure application registration requirements](docs/azure-app-registration-request.md)
and [cloud permission requirements](docs/azure-cloud-permissions-required.md).
Use your own tenant and resources; example deployment details in those documents
are not shared service credentials.

### Try one delivery loop

Start with a non-production project and a work item with a linked PR or build.

1. Select the repository's Project Link in **Context** and inspect the work item
   in **Work**.
2. Ask: **“Explain this work item's acceptance criteria and linked PR/build
   evidence. Keep this read-only.”**
3. For a change you want to submit, ask: **“Help me prepare a PR from my current
   branch. Inspect changes and target-branch policy before proposing creation.”**
4. Review the proposed target and payload. Approve only the intended write, or
   decline it to stop the action.
5. Check the action's verification result and the resulting Azure DevOps
   artifact. A model's completion message alone is not the success criterion.

## Data and action boundaries

**Local-first describes the runtime and default storage, not offline inference.**
The desktop communicates with a local daemon over loopback HTTP and SSE.
Task-relevant code, prompts, and delivery evidence can be sent to your configured
model provider. Connected workflows communicate with Azure DevOps.

| Boundary               | Behavior to account for                                                                                                                                                             |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Local state            | The daemon defaults to the user's `.mergepilot` directory; `MERGEPILOT_HOME` can override it. Local state can contain sensitive repository and conversation content.                |
| Optional cloud storage | Configured Azure Table Storage and Cosmos DB integrations can persist Project Links and sessions remotely. Check your deployment configuration before treating state as local-only. |
| Credentials            | Use the app's authentication and secret configuration. Keep keys, token caches, local configuration, and private logs out of commits and issue reports.                             |
| Approved writes        | The action lifecycle binds the proposal, approval, execution, and verification evidence. Stale targets and failed verification require inspection before a new proposal.            |
| Recovery               | Inspect the existing action record after a timeout or restart before requesting another write. A timeout does not prove that the remote operation failed.                           |

The [governance specification](docs/product/risk-and-governance.md) describes the
required policy, freshness, idempotency, retention, and audit contracts. Review
these requirements against your organization's data and access policies during
a pilot.

## Production readiness

### Recorded release evidence

The v0.5.32 acceptance run, `verify-msorfadi`, was recorded on **2026-08-11**
against product source **`7067240`**. It reports **14/14 required gates passed**,
including package tests and typechecks, builds, **85 mocked browser tests**,
**30 source-live tests**, installed desktop checks, and real Azure DevOps checks.
See the [version-pinned gate report](https://github.com/ZP151/mergepilot/blob/v0.5.32/docs/manual-testing/2026-08-05/verification/current-gates.md).

The [release closure record](https://github.com/ZP151/mergepilot/blob/v0.5.32/docs/product/next-iteration-known-gaps.md)
documents the source → MSI → installed daemon payload evidence and an installed
Work Item → linked PR/build → approval → write-back → authoritative re-read loop.
These are recorded results for that source and fixture, rather than evidence
that every later checkout, tenant, model, or deployment scenario has passed.

### Conditions for broader use

- **Validate your environment.** Repeat a supported loop with your tenant,
  permissions, repository policies, and model configuration. External pilot
  research and a governed non-production deployment fixture remain next steps.
- **Verify the artifact you distribute.** Check installer signing and the
  relationship between tested source, release tag, build, and installed payload.
  Further release-provenance automation remains on the roadmap.
- **Measure real responsiveness.** Separate app overhead from model and Azure
  DevOps latency; the accepted developer loop does not establish a response-time
  SLA for another deployment.
- **Exercise operational controls.** Test cancellation, stale proposals,
  restart recovery, diagnostics, and the remote-write switch before expanding
  access. Continue accessibility and narrow-window acceptance work.

Release owners retain deployment authority. The
[known-gap register](docs/product/next-iteration-known-gaps.md) and
[measurement and pilot plan](docs/product/measurement-research-and-gtm.md) define
the work needed to expand adoption responsibly.

## Roadmap

The iteration history has moved MergePilot from PR review and repository chat
toward a governed delivery workbench:

| Milestone                                         | Status and direction                                                                                                                                                                                 |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cycles 00–06                                      | Established the shared action path and Work Item → PR → CI foundation, then expanded Changes, Work, delivery evidence, and hardening. The accepted v1 developer loop is the bounded release outcome. |
| v0.5.29                                           | Consolidated Context, session continuity, guided PR preparation, Work/Changes states, and Settings.                                                                                                  |
| v0.5.32                                           | Closed Work Inspector, guided PR preparation, and installed developer-loop acceptance with recorded source and artifact evidence.                                                                    |
| Next: truthful latency and deterministic evidence | Attribute app/provider/tool delays and make verifier inputs and release identities explicit.                                                                                                         |
| Then: governed pilot expansion                    | Validate external team workflows and non-production deployment evidence.                                                                                                                             |
| Later: release confidence and adoption            | Improve release repeatability, accessibility, and measurable team value.                                                                                                                             |

The last three rows are planned work. Their
[Cycle 07–09 proposal](https://github.com/ZP151/mergepilot/tree/0480815/docs/product/cycles)
is on an unmerged planning branch at the time of this README update. The
[canonical product plan](docs/product/README.md) governs accepted scope.

The product's north-star metric is **verified delivery loops per active project
per week**: evidence recorded, mutation approved, action executed, and remote
state verified. Review and triage time, incorrect write-backs, reversals, and
stale-action prevention are the accompanying quality measures.

## Develop from source

The supported Windows development path uses Git, PowerShell 7, Rust/MSVC build
tools, and WebView2 for the Tauri shell. The repository runner expects
**Node.js 22.11.0** and **pnpm 9.0.0** under `.tools`.

```powershell
git clone https://github.com/ZP151/mergepilot.git
Set-Location mergepilot
```

`.tools` is ignored by Git and is **not included in a fresh clone**. Before
running the wrapper, extract the official Node.js Windows x64 distribution so
that `.tools\node-v22.11.0-win-x64\node.exe` exists, and place the pnpm 9.0.0
Windows x64 standalone executable at `.tools\pnpm.exe`. Obtain pnpm from its
[official release](https://github.com/pnpm/pnpm/releases/tag/v9.0.0).

From the repository root:

```powershell
.\scripts\windows\pnpm-project.ps1 install --frozen-lockfile
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/desktop tauri:dev
```

`tauri:dev` prepares the daemon sidecar and launches the desktop shell. The
desktop package's `dev` script starts **only the Vite frontend** and is intended
for browser development with a separately running daemon.

### Verify a change

```powershell
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/core typecheck
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/core test
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/daemon typecheck
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/daemon test
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/desktop typecheck
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/desktop test
.\scripts\windows\pnpm-project.ps1 --filter @mergepilot/desktop build
```

These package checks are a development baseline. Changes to live actions or
packaged behavior also need the corresponding
[business acceptance checks](docs/automated-business-test-suite-plan.md) and
[verification run tooling](scripts/verification).

## Architecture

```text
Tauri desktop (React)
        |
        | loopback HTTP + SSE
        v
Local daemon: sessions, turn orchestration, approvals, verification
        |
        v
Core: repository context, planner, policy, Git / Azure DevOps clients
        |
        +-- Local Git checkout and state
        +-- Azure DevOps APIs / managed capability transport
        +-- Configured model provider
        +-- Optional cloud persistence and review service
```

| Area                                    | Location                                         |
| --------------------------------------- | ------------------------------------------------ |
| Desktop interface and native shell      | [`apps/desktop`](apps/desktop)                   |
| Local API and action orchestration      | [`packages/daemon`](packages/daemon)             |
| Planner, policy, retrieval, and clients | [`packages/core`](packages/core)                 |
| Optional review service                 | [`packages/review-agent`](packages/review-agent) |
| Command-line interface                  | [`packages/cli`](packages/cli)                   |

See the [architecture overview](docs/architecture.md) and
[delivery graph and action runtime](docs/product/delivery-graph-and-action-runtime.md)
for component boundaries and the target contracts.

## Troubleshooting and support

| Symptom                                  | First check                                                                                                                       |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Microsoft sign-in or ADO access fails    | Check the selected account, tenant/client configuration, Project Link, and permissions on the actual target.                      |
| Model connection fails or a turn is slow | Test the configured model; check endpoint, deployment, and secret source. Use diagnostics to distinguish app and provider delays. |
| An action is stale or verification fails | Refresh the artifact and inspect its action record before proposing another write.                                                |
| Remote writes need to stop               | Turn off **Allow approved remote writes** in Settings → Capabilities.                                                             |
| Local toolchain is missing               | Provision the two `.tools` paths described above; the runner does not download them automatically.                                |

For a reproducible bug, [open an issue](https://github.com/ZP151/mergepilot/issues)
with the app version, Windows version, affected workflow, expected/actual result,
and correlation ID from Settings → Diagnostics. Include redacted evidence only.
Read [Contributing](CONTRIBUTING.md) before proposing a change.

## Related projects and design references

These projects informed the workflow positioning and documentation structure;
they are references, not required installations or claims of feature parity.

- [Microsoft Azure DevOps MCP](https://github.com/microsoft/azure-devops-mcp):
  Azure DevOps tool access, with explicit connection setup and troubleshooting.
  MergePilot adds its own desktop context and governed delivery workflow.
- [PR-Agent](https://github.com/The-PR-Agent/pr-agent): task-oriented PR review
  workflows and deployment documentation. MergePilot connects review to work
  items and subsequent delivery evidence.
- [OpenHands](https://github.com/OpenHands/OpenHands): developer-agent workflows
  with explicit installation, runtime, and architecture boundaries.

For competitive context, see the
[product landscape](docs/product/competitive-landscape.md). For actual vendored
code, provenance, and notices, see
[third-party source reuse](docs/third-party-source-reuse.md).

## License

This repository does not currently include a root license granting reuse of
MergePilot code. Clarify licensing with the maintainer before redistribution or
commercial reuse. Vendored components retain their respective licenses and
notices; those do not grant a license to the entire application.
