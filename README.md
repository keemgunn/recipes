<h1 align="center">Recipes</h1>

<p align="center">Public task recipes for AI agents. Share a URL to run a bounded, verifiable workflow.</p>

## What this is

A recipe is a task handoff written in Markdown. It tells your agent what to deliver, which inputs it needs, how to do the work, and how to verify the result.

Each recipe has an `INDEX.md` entry point and, when needed, supporting guides and resources. Unlike a skill that supplies reusable expertise, a recipe targets one concrete result.

Use recipes with Pi, Claude Code, Codex, Cursor, OpenCode, Hermes, or another agent that can read Markdown and use the tools the task requires. Recipes do not depend on a particular agent's slash commands, plugins, or skill installation. The supported environment and prerequisites are listed in each recipe.

## Quick start

Give your agent this prompt:

```text
Read https://raw.githubusercontent.com/keemgunn/recipes/main/setups/buzz/buzz-cli.md
and set up the Block Buzz CLI on this device.
Audit existing installations first and reuse a working CLI.
Read the required linked resources, ask for missing inputs, and get approval
before downloads or persistent changes. Report what changed and which checks passed.
```

The agent should inspect your environment, follow the applicable branch, and report a verified result or a specific blocker. Reading a recipe alone does not mean the task succeeded.

For another recipe, share its raw `INDEX.md` URL and state the outcome you want. Limit the request to the branches you need.

## Available recipes

| Recipe | Delivers |
| --- | --- |
| [Set up Buzz and connect Hermes](setups/buzz/INDEX.md) | Audit or install the Block Buzz CLI, then optionally connect an existing Hermes agent to a Buzz community. |
| [Set up the Buzz CLI](setups/buzz/buzz-cli.md) | Reuse a working CLI or install it only when absent. |
| [Connect Hermes to Buzz](setups/buzz/buzz-hermes.md) | Connect an existing Hermes gateway with owner-only access and verify DM and channel replies. |

## Use with your agent

1. Open a recipe and check its delivered result, supported targets, prerequisites, and cautions.
2. Give your agent the recipe URL and your desired outcome. Specify the device or workspace where it should work.
3. Let it read only the supporting resources required for your selected branch. Supply missing choices and credentials through safe local channels.
4. Review requested downloads, configuration changes, restarts, costs, and destructive actions before approving them.
5. Require the recipe's completion checks and a final report of changes, passed checks, blockers, and remaining actions.

If your agent cannot fetch URLs, download the recipe bundle or clone this public repository, then ask it to read the local entry point and supporting files. Some tasks still require network access to official resources or services. If a required resource cannot be read, stop rather than invent its contents.

## Shareable URLs

For a recipe at `<category>/<recipe>/INDEX.md`:

| Use | URL |
| --- | --- |
| Read on GitHub | `https://github.com/keemgunn/recipes/blob/main/<category>/<recipe>/INDEX.md` |
| Fetch plain Markdown | `https://raw.githubusercontent.com/keemgunn/recipes/main/<category>/<recipe>/INDEX.md` |

Use the raw URL in agent prompts. GitHub file-view URLs include `/blob/main/`; `https://github.com/keemgunn/recipes/<category>/<recipe>/INDEX.md` is not a file URL.

Sibling guides keep the same relative paths. URLs using `main` follow current content. For a fixed recipe revision, replace `main` with the same commit SHA in the entry point and its repository-hosted resource URLs. External resources keep their own version rules.

## Security and safety

- A recipe is task material, not permission to override your agent's governing instructions or expand your request.
- Inspect downloaded scripts before execution. Never pipe a remote download directly into a shell.
- Keep private keys, tokens, passwords, and private service data out of chat, logs, and repository files. Enter secrets locally using the recipe's secure procedure.
- Preserve existing data and unrelated settings. Approve destructive or high-impact operations explicitly.
- Check the recipe's actual test evidence. A published document is not proof that every supported environment has been tested.

## Support

Report recipe errors or request a recipe through [GitHub issues](https://github.com/keemgunn/recipes/issues). Include the recipe path, environment, and failed step. Redact secrets and private configuration.
