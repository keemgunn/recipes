---
id: "setups/buzz/buzz-hermes"
title: "Connect Hermes to Buzz"
summary: "Connect an existing Hermes installation to a Buzz community through its native messaging gateway."
category: "setups"
updated: "2026-10-03"
targets:
  - "Linux with Hermes and optional systemd persistence"
  - "macOS with Hermes and optional launchd persistence"
  - "WSL2 with Hermes using the Linux procedure"
  - "Windows native with a working Hermes installation"
requires:
  - "Hermes with the native Buzz platform and configured model provider"
  - "Buzz community access and a dedicated agent identity"
---

# Connect Hermes to Buzz

## Delivers

- A dedicated Buzz agent connected to the existing Hermes gateway, with owner-only access.
- A verified reply in a Buzz DM and a selected channel; approved gateway persistence when requested.
- Out of scope: installing Hermes itself, hosting a relay, installing Buzz Desktop, enabling community-wide tool access, or Buzz-managed ACP runtimes.

Buzz Desktop/web and the Hermes host connect independently to the same relay. The user's client does not need to SSH into Hermes for messaging; the native gateway can keep running while Buzz Desktop is offline.

## Before starting

1. Detect OS, architecture, shell, service manager, active Hermes profile, and its actual `HERMES_HOME`. Check `hermes status` and `hermes gateway status`; confirm Hermes already responds using its configured provider and supports the Buzz platform. Stop if the base installation is missing or incompatible; do not invent a minimum Hermes version.
2. Confirm the native-gateway topology. If the user wants Buzz to launch/manage Hermes through ACP, stop this branch and consult the mapped architecture/ACP docs instead.
3. Ask for non-secret inputs: community HTTP(S) base URL, intended agent name, owner's public `npub` or hex key, selected channel UUIDs, optional notification/home channel, and requested persistence.
4. Inspect existing Buzz configuration and credential presence without displaying values. If this connection already works with the requested policy, reuse it and go to Verify. Preserve unrelated messaging platforms and services.
5. Before writes, create restricted local backups of the active profile's configuration/secret files and record the gateway/service state. Before replacing or force-reinstalling persistence, also back up the affected supervisor definitions, overrides, service-specific environment settings, and any existing scheduled task. Keep potentially secret backups out of public resources, logs, shared scratch, and source control.

## Resources

| Resource | URL | Fetch when / purpose |
| --- | --- | --- |
| Our Buzz CLI recipe | https://raw.githubusercontent.com/keemgunn/recipes/main/setups/buzz/buzz-cli.md | Required: audit/reuse or install the correct CLI. Owns all CLI installation steps. |
| Hermes Buzz integration modes | https://hermes-agent.nousresearch.com/docs/integrations/buzz | Required: confirm native gateway versus ACP topology. |
| Hermes Buzz configuration | https://hermes-agent.nousresearch.com/docs/user-guide/messaging/buzz | Required: check live configuration, transport, and access semantics. |
| Upstream recipe source | https://raw.githubusercontent.com/tonbistudio/buzz-skills/1eeed8cfe81702f0e7d7f257b44103ef49f004a3/hermes-in-buzz/SKILL.md | Provenance and connection details; its CLI installation section is replaced by our recipe. |
| Credential updater | https://raw.githubusercontent.com/tonbistudio/buzz-skills/1eeed8cfe81702f0e7d7f257b44103ef49f004a3/hermes-in-buzz/scripts/update_buzz_credentials.py | When credentials/admission need setup: hidden-input storage and authenticated channel probe. |
| Public-identity / NIP-OA helper | https://raw.githubusercontent.com/tonbistudio/buzz-skills/1eeed8cfe81702f0e7d7f257b44103ef49f004a3/hermes-in-buzz/scripts/hermes_buzz_credential_helper.rs | When needed: derive public identity and create owner attestation without private keys in argv. |
| Block Buzz stable release | https://api.github.com/repos/block/buzz/releases/latest | Only if building the credential helper requires a new pinned Buzz checkout. |
| Hermes ACP reference | https://hermes-agent.nousresearch.com/docs/user-guide/features/acp | Only if the requested topology is not the native gateway; separate workflow. |

Read this guide and our CLI recipe, then fetch only resources needed for the selected branch using permitted web tools. Resolve relative links against this guide's public URL; save required helpers to known absolute paths and read them before execution. Stop on missing resources or material source conflicts; current official docs govern supported behavior, while our CLI recipe owns CLI installation.

## Cautions

- Use a dedicated **agent** private key, never the human owner's key, for `BUZZ_PRIVATE_KEY`.
- Never ask for a private key, auth tag, or token in chat, an agent question tool, argv, logs, or shell history. The user enters secrets in a real trusted local terminal with hidden input. Stop if input would echo or `getpass` warns that hidden input is unavailable.
- Store credentials only in the active Hermes profile's secret environment file. Verify Unix mode `0600` or equivalent Windows ACLs; the upstream updater's permission changes are best-effort and do not prove Windows protection.
- The updater replaces `.env` atomically and creates no backup. Inspect symlinks and file ownership first; do not replace a symlinked/shared credential file without an approved storage-preserving method.
- Never shell-source `.env`: JSON auth tags can lose their quotes. Read credential presence redacted; use the updater's parser/probe, not a printed environment dump. Its parser supports simple one-line `KEY=value` entries, not full dotenv expansion/export/multiline syntax; stop and resolve incompatible formatting before using it.
- Keep `allow_all_users=false`, an explicit owner allowlist, and mention gating. Do not loosen these to troubleshoot delivery.
- Ask before downloads/build tools, credential replacement, service changes, restart downtime, or persistent startup changes. Run exactly one gateway per profile; preserve unrelated platform configuration.
- ACP is not a substitute for this recipe: Buzz may auto-approve ACP tool execution. ACP agents need owner-only access and a separate security review.

## Cook

### 1. Audit or install the CLI

Follow [Set up the Buzz CLI](buzz-cli.md). If the correct CLI is already present and working, **skip installation** and use its audited absolute path. Do not execute the upstream skill's CLI installation section as an additional step.

Pass: the selected absolute `buzz` path executes `--help` and provides `channels list` and message operations.

### 2. Prepare credential tools when needed

1. Download and inspect the pinned updater and Rust helper from Resources. The updater uses Python 3.10+ standard library only; no package install is required. Use the existing Hermes interpreter through `uv run --no-project --python "<absolute-Hermes-python>" "<absolute-updater-path>" ...`; if `uv` is unavailable, ask before adding it or agree on the existing-interpreter alternative. Run secret-entry commands in the user's terminal, not a captured agent tool session.
2. Reuse an already audited compatible credential-helper executable if available. Otherwise obtain a pinned stable `block/buzz` checkout, preferably the retained checkout from the CLI recipe. If the CLI was reused, this helper build is separate and must not reinstall or upgrade the CLI.
3. Copy the inspected Rust source into that checkout at `crates/buzz-sdk/examples/hermes_buzz_credential_helper.rs`, only if that destination is absent or already identical. Build it with:

```text
cargo build --locked --release -p buzz-sdk --example hermes_buzz_credential_helper --manifest-path "<absolute-build-dir>/Cargo.toml" --target-dir "<absolute-build-dir>/target"
```

4. Install the verified helper to an approved user path as `hermes-buzz-credential-helper` (`.exe` on Windows), preserving any existing file. Run its absolute path with `self-test`; it must print `ok`. This test uses disposable generated keys, no real credentials, and no network. If it cannot compile against the selected SDK, stop and resolve compatibility; do not substitute unreviewed code.

In the commands below, `<updater-command>` means the resolved interpreter invocation plus the updater's absolute path. Fill every placeholder before running; private values are never placeholders to put on the command line.

Pass: the updater's `--help` works and the helper's `self-test` passes.

### 3. Store and verify the dedicated agent identity

1. Reuse a matching existing agent identity when available. Otherwise the user creates the intended agent in Buzz Desktop and retains its one-time agent `nsec` locally. Creating an agent or adding it to a channel does not necessarily grant relay admission.
2. In the user's trusted terminal, run:

```text
<updater-command> set-agent-key --hermes-home "<resolved-Hermes-home>"
```

3. Derive only public values:

```text
<updater-command> public --hermes-home "<resolved-Hermes-home>" --helper "<absolute-helper-path>"
```

4. Compare the resulting `hex`/`npub` with the intended agent in Buzz. If they differ, stop and resolve the identity; do not regenerate keys repeatedly. Independently verify secret-file permissions.

Pass: the stored key belongs to the intended dedicated agent and no secret appeared in chat, argv, or logs.

### 4. Prove relay admission

Use the native gateway/CLI community HTTP(S) base URL, preferably HTTPS, from the community settings. Do not blindly use an ACP `wss://` transport URL as the native CLI base URL.

```text
<updater-command> probe --hermes-home "<resolved-Hermes-home>" --buzz "<absolute-buzz-path>" --relay "<community-base-URL>"
```

The probe loads the profile's `BUZZ_*` values into the CLI subprocess environment and runs `channels list`. Inspect results locally; avoid publishing private channel names or raw relay responses.

- Successful admission: confirm intended channel membership and UUIDs. An empty channel list is not a successful usable connection.
- `relay_membership_required` / 403 with a matching public identity: prefer administrator admission for the agent, or use owner attestation below if the community requires it.
- Other failure: resolve the specific binary, URL, credentials, or relay error. Do not reinstall a working CLI as an authentication fix.

#### Owner attestation — only if required

Explain the delegation before signing: the helper defaults to empty, unrestricted conditions, so attestation is not necessarily limited to admission or messaging. Obtain approval for that scope, or use reviewed relay-supported restrictions through `--conditions`; do not invent a restriction format. Prefer administrator admission when it avoids bringing the human key onto the Hermes host. If attestation is approved, the user retrieves the human/owner private key from Buzz Identity settings and enters it only in the trusted local terminal on the Hermes host:

```text
<updater-command> set-auth-tag --hermes-home "<resolved-Hermes-home>" --helper "<absolute-helper-path>" --agent-pubkey "<agent-public-64-hex>"
```

The updater passes the owner key over stdin, validates a four-string `auth` tag, and stores only `BUZZ_AUTH_TAG`. It does not store the owner key; memory clearing is best-effort, not guaranteed erasure. Do not print the raw helper `auth-tag` output or retain the owner key in a file. Repeat the probe after attestation.

Pass: authenticated channel discovery succeeds for the exact agent and includes the intended channel.

### 5. Configure the active Hermes profile

Inspect the installed `hermes config set`/`get` help. Use those commands in the selected profile; do not hand-edit `config.yaml`. Set and read back each value, using supported JSON-array syntax for lists:

| Key | Value |
| --- | --- |
| `gateway.platforms.buzz.extra.relay_url` | The tested community HTTP(S) base URL. |
| `gateway.platforms.buzz.extra.cli_path` | Audited absolute CLI path. |
| `gateway.platforms.buzz.extra.channels` | Explicit selected channel UUID list; `[]` only if watching all joined channels is approved. |
| `gateway.platforms.buzz.extra.home_channel` | Selected UUID only when notifications/cron delivery are requested. |
| `gateway.platforms.buzz.extra.poll_interval` | `4` seconds. |
| `gateway.platforms.buzz.extra.require_mention` | `true`. |
| `gateway.platforms.buzz.extra.allow_all_users` | `false`. |
| `gateway.platforms.buzz.extra.allowed_users` | Nonempty list containing the owner's public `npub` or hex key. |
| `display.platforms.buzz.interim_assistant_messages` | `false`. |
| `display.platforms.buzz.tool_progress` | `off`. |
| `gateway.platforms.buzz.enabled` | `true`, after credentials and access policy are ready. |

Existing environment/service overrides can defeat these settings. Inspect names and redacted set/unset state for `BUZZ_RELAY_URL`, `BUZZ_CHANNELS`, `BUZZ_HOME_CHANNEL`, `BUZZ_ALLOWED_USERS`, `BUZZ_ALLOW_ALL_USERS`, `BUZZ_POLL_INTERVAL`, `BUZZ_CLI_PATH`, `BUZZ_TRANSPORT`, and `BUZZ_CREDENTIALS_FILE`. Preserve deliberate transport/storage choices; migrate or remove conflicting non-secret overrides only after their canonical replacement is verified and the user approves. Check inherited service variables too, not just `.env`. Preserve `BUZZ_PRIVATE_KEY`, `BUZZ_AUTH_TAG`, and any required documented token.

Pass: the intended profile has the tested URL/absolute CLI path, matching identity, selected channels, and effective owner-only access without contradictory overrides.

### 6. Start or reload one gateway

- Existing running gateway: use its established supervisor and documented reload/restart mechanism after approval. Do not launch a second foreground instance or reinstall an unrelated service.
- New Linux/systemd or macOS/launchd service: with approval, use `hermes gateway install --force`, then `hermes gateway start` and `hermes gateway status`, in the selected profile. Reinstall an existing service only when required by the installed Hermes version, such as a launchd environment refresh.
- Linux logout persistence: request approval before `loginctl enable-linger` for the intended account. Do not create a system-wide service as an incidental workaround.
- WSL2: use Linux service management only if systemd is already available. Otherwise verify with `hermes gateway run`; enabling systemd/restarting WSL or adding a Windows scheduled task needs separate approval.
- Windows native: verify with `hermes gateway run`. Optional Task Scheduler persistence uses the full Hermes executable path, selected profile/home, and `gateway run --external-supervisor` only if the installed version supports it. Never claim Linux/macOS service support on Windows.

Pass: exactly one gateway runs for the intended profile, using the agreed supervisor and credential environment.

## Verify

1. Gateway status/logs show Buzz connected, the correct public identity, and the intended nonzero channel set. Connection log text varies by version; upstream examples include `buzz connected` and `watching N channel(s)`.
2. Ask the owner to send one simple DM and one `@Agent hello` mention in the selected test channel. Explain any provider charge before testing; use no production data or tool-triggering request.
3. Confirm inbound Buzz handling, response generation, and successful outbound send in local logs, **and** ask the user to confirm both replies actually appear in Buzz. Process state or send logs alone are insufficient.
4. Confirm the allowlist/mention policy remains effective and unrelated platforms still work. Verify logout/reboot survival only when requested and approved; do not reboot the device merely to finish this recipe.

Pass: the user sees both replies, owner-only policy is verified, and any requested persistence has truthful test evidence. If the user cannot perform the UI checks, report the connection as awaiting end-to-end verification.

## Recovery

| Symptom | Check before changing anything |
| --- | --- |
| CLI binary missing | Absolute `cli_path`, file permissions/architecture, and service environment. |
| Admission 403 / `relay_membership_required` | Matching public identity, direct relay membership, or required owner attestation. |
| Zero channels | Agent channel attachment and configured channel filters. |
| Ignored messages | Sender allowlist, mention gating, correct test identity, and newest-message handling. |
| Inbound works, outbound fails | Installed CLI path and exact credential environment; do not shell-source `.env`. |
| Gateway appears silent | Active profile's `gateway.log` as well as supervisor logs; do not disable an unrelated platform. |

For rollback, pause only the affected profile's gateway through its existing supervisor. Restore this run's configuration/credential changes and any replaced supervisor definitions, overrides, service environment, or scheduled task from the restricted backups, preserving later edits. Reload the established supervisor as required, restore its previous enabled/running state, and verify pre-existing platforms. Remove only new helper files/startup entries this run created, with approval; leave a reused CLI and existing identities untouched. Disabling the connection does not revoke relay membership/attestation—revocation is a separate owner/admin action. Rotate keys only if exposure occurred and the user authorizes it.

## Report

Return CLI `reused`/`installed`, topology, selected profile, redacted change summary, connection/DM/mention/persistence checks, and blockers. Never include private keys, auth tags, raw environment contents, or private channel data. Distinguish configured, running, and end-to-end verified states.

## Share prompt

> Read https://raw.githubusercontent.com/keemgunn/recipes/main/setups/buzz/buzz-hermes.md and connect my existing Hermes installation to my Buzz community using the native gateway. Use its linked Buzz CLI recipe, reuse a working CLI, and keep access owner-only.

## Source

Adapted from `tonbistudio/buzz-skills`'s `hermes-in-buzz/SKILL.md` at commit `1eeed8cfe81702f0e7d7f257b44103ef49f004a3`, with current official Hermes documentation checked on 2026-10-03. CLI installation is owned by [our recipe](buzz-cli.md); credential helpers remain pinned upstream resources. No target-device installation or connection test was performed while authoring this recipe.
