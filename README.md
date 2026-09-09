# Portable GitHub Copilot Research Team

A reusable six-role agent team for long-running engineering and research work
with GitHub Copilot CLI. The repository contains only agent definitions and
shared operating instructions. It does not contain project code, project
memory, credentials, machine-specific paths, or Copilot session data.

The workflow is execution-first: the lead delegates implementation and
experiments, accepts engineering results from real tests and runtime evidence,
and requests adversarial review only when it is useful.

## Team design

| Role | Responsibility | Default model | Effort |
|---|---|---|---|
| A0 Lead | Objectives, prioritization, delegation, integration, and delivery | GPT-6 Astra | High |
| A1 Theory | Research, mechanisms, competing hypotheses, and difficult diagnosis | GPT-5.6 Sol | XHigh |
| A2 Implementation | Bounded code changes, repairs, and focused tests | GPT-6 Astra | Medium; High for difficult work |
| A3 Experiment | Runs, integration debugging, experiments, and direct evidence | GPT-5.6 Terra | Medium; High for difficult diagnosis |
| A4 Gemini Reviewer | Default independent review for major milestones or persistent failures | Gemini 3.8 Flash | High |
| A4 Grok Reviewer | Optional independent second perspective | Grok 4.6 | High |
| A5 Knowledge | Requested handoffs and concise project documentation | GPT-5.6 Luna | Medium |

Model availability depends on your Copilot plan and organization policy. If a
declared agent model is unavailable, Copilot CLI may fall back to the active
session model. Use `/model` inside the CLI to see the models available to your
account, and edit the agent profiles if you prefer a different model mix.

## How work flows

```text
User
  └─ A0 Lead
      ├─ A1 Theory       research and diagnosis when needed
      ├─ A2 Implementation
      ├─ A3 Experiment
      ├─ A4 Reviewers    only at major closeout or persistent failure
      └─ A5 Knowledge    only for requested durable documentation
```

A0 normally follows this cycle:

1. Inspect only enough context to identify the smallest executable work unit.
2. Send implementation to A2 or execution and evidence collection to A3.
3. Evaluate the actual diff, relevant tests, integration checks, and runtime
   results.
4. Invoke one A4 reviewer only for a major milestone or a blocking error,
   repeated failure, or evidence contradiction that remains after normal
   debugging.
5. Use independent Gemini and Grok first passes together only for unusually
   consequential or conflicting cases.
6. Ask A5 for a handoff or durable project note only when one is wanted.

A0 may authorize experiments within the current objective, allowed paths, and
available resource envelope. User approval is still required when execution
changes scope, creates new external spending, exceeds an explicit resource or
time limit, or is irreversible, destructive, or externally consequential.

## Requirements

- A GitHub account with access to GitHub Copilot CLI.
- Git.
- GitHub Copilot CLI installed and authenticated.
- `tmux` is optional but useful on remote machines.

GitHub's current npm installation command is:

```bash
npm install -g @github/copilot
```

If you do not want a global install or do not have administrator access, install
the CLI inside this repository instead:

```bash
npm install --prefix .tools/copilot-cli @github/copilot
.tools/copilot-cli/node_modules/.bin/copilot
```

Start the CLI with `copilot`, enter `/login`, and complete authentication. See
the official [Copilot CLI quickstart](https://docs.github.com/en/copilot/get-started/cli-quickstart)
for other installation methods.

## Quick start

Clone this configuration repository:

```bash
git clone https://github.com/wyaaaattwho/copilot-research-team-config.git
cd copilot-research-team-config
```

Start A0 and grant access only to the project you want the team to work on:

```bash
copilot \
  --model gpt-6-astra \
  --effort high \
  --agent a0-lead \
  --add-dir /absolute/path/to/your-project
```

Then give A0 an outcome-oriented request, for example:

```text
Inspect the target repository and its local instructions. Implement the
smallest useful work package for the current objective, verify it with relevant
tests or real execution, and report the result and remaining risks.
```

The configuration repository remains the CLI working directory, while
`--add-dir` grants access to each target repository. Repeat `--add-dir` when a
task spans multiple repositories:

```bash
copilot \
  --model gpt-6-astra \
  --effort high \
  --agent a0-lead \
  --add-dir /absolute/path/to/application \
  --add-dir /absolute/path/to/benchmark
```

Use absolute paths, especially over SSH, to avoid ambiguity about which project
the agents may access.

## Running in tmux

Create a persistent terminal session:

```bash
tmux new-session -s copilot-team
cd /path/to/copilot-research-team-config
copilot \
  --model gpt-6-astra \
  --effort high \
  --agent a0-lead \
  --add-dir /absolute/path/to/your-project
```

Detach with `Ctrl+B`, then `D`. Reattach later with:

```bash
tmux attach-session -t copilot-team
```

## Autopilot and permissions

Autopilot can continue a multi-step objective without waiting after every model
turn. Tool, path, and URL permissions are separate controls.

A bounded example is:

```bash
copilot \
  --model gpt-6-astra \
  --effort high \
  --agent a0-lead \
  --add-dir /absolute/path/to/your-project \
  --autopilot \
  --max-autopilot-continues 10 \
  --allow-all-tools \
  --allow-all-urls
```

`--allow-all-tools` removes per-tool confirmation and can permit consequential
commands. Use it only in a controlled environment, keep path access narrow, and
review working-tree changes before accepting them. For stricter setups, omit
the allow-all flags and approve individual tools or URLs as needed. See
[Configuring GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/configure-copilot-cli)
for allowlists and denylists.

## Selecting agents directly

Normally A0 delegates to the other roles. You can also select a profile with
`/agent` in an interactive session or start one directly:

```bash
copilot --agent a1-theory --add-dir /absolute/path/to/your-project
copilot --agent a3-experiment --add-dir /absolute/path/to/your-project
copilot --agent a4-reviewer --add-dir /absolute/path/to/your-project
copilot --agent a4-reviewer-grok --add-dir /absolute/path/to/your-project
```

Restart Copilot CLI after changing an agent file so the new definition is
loaded.

## Repository layout

```text
.github/
├── agents/
│   ├── a0-lead.agent.md
│   ├── a1-theory.agent.md
│   ├── a2-implementation.agent.md
│   ├── a3-experiment.agent.md
│   ├── a4-reviewer.agent.md
│   ├── a4-reviewer-grok.agent.md
│   └── a5-knowledge.agent.md
├── copilot/
│   └── settings.json
└── copilot-instructions.md
```

- Agent files define role boundaries, tools, models, and effort levels.
- `copilot-instructions.md` contains rules shared by the whole team.
- `settings.json` records the preferred outer-session model and effort for this
  configuration repository. Explicit CLI flags remain the clearest way to
  select a session model.

## Adapting the team

- Change `model` and `reasoningEffort` in an agent profile to match models
  available to your account.
- A0 and A2 intentionally inherit the outer Astra session; the A0 instructions
  request Astra Medium when dispatching A2.
- Adjust the `tools` list to narrow what a role can do.
- Edit A0's `agents` list if you add or remove specialist profiles.
- Keep project-specific build commands, paths, safety constraints, and project
  memory in the target repository rather than in this portable configuration.

Custom agent files use GitHub's documented `.agent.md` format. See the official
[custom agents reference](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
and [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference).

## Data and security

Do not commit any of the following:

- `~/.copilot` or copied Copilot session state;
- authentication tokens, cookies, or credentials;
- installed CLI dependencies;
- project datasets, experiment artifacts, or private project memory;
- machine-specific paths or secrets embedded in prompts.

Copilot sessions remain local to each machine. Git synchronizes these agent
definitions, not active sessions or conversation history.
