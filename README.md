# Personal Claude setup for dev containers

Keep your custom skills, hooks, status line, and settings in this folder, then
use them while working inside your existing dev container. No plugin installation
or launcher scripts needed.

**The flow:** edit here → start Claude in the container with your local setup →
test → edit again. The container keeps its normal setup, including its MCPs.

## Where things live

| Location | Purpose |
| --- | --- |
| `~/personal/projects/claude-plugin` | This checkout on the host: all your customizations |
| `/home/vscode/claude-plugin` | The same files inside the container, mounted read/write |
| `~/personal/projects/dev-container` | The project whose dev container you use for testing |

```text
plugin/
  .claude-plugin/plugin.json
  skills/hello/SKILL.md
  hooks/hooks.json           # add when needed
config/
  settings.json             # personal settings, including the status-line command
  statusline.sh
```

## Start Claude with your setup

Run on the host:

```bash
cd ~/personal/projects/dev-container
devcontainer up --workspace-folder .
devcontainer exec --workspace-folder . \
  claude \
  --add-dir /home/vscode/claude-plugin \
  --plugin-dir /home/vscode/claude-plugin/plugin \
  --settings /home/vscode/claude-plugin/config/settings.json
```

What happens:

1. `devcontainer up` starts or reuses the project's container. Its configuration
   mounts this checkout read/write at `/home/vscode/claude-plugin`.
2. `devcontainer exec` starts a new Claude session in that container's workspace.
3. `--plugin-dir` loads your skills and any plugin hooks for this session.
4. `--settings` layers your personal settings onto Claude's normal settings.
   Values you specify override the corresponding normal settings for this
   session, subject to organization-managed policy; other settings stay in place.
5. `--add-dir` lets Claude access this checkout too, so you can ask it to edit
   your customizations while working in the project. Normal approvals still apply.

The command does **not** install your plugin globally, copy your settings into
shared configuration, or disable the container's MCPs. Plain `claude` starts
without explicitly loading this local setup.

Already inside the container? Run the `claude` command above without the
`devcontainer exec --workspace-folder .` prefix.

## Make changes and test

Edit on the host or inside the container: both paths refer to the same files,
and edits persist on the host.

- **Skills:** add or edit `plugin/skills/<name>/SKILL.md`. Invoke `/danny:<name>`.
- **Hooks:** add `plugin/hooks/hooks.json` with a top-level `"hooks"` object.
  Keep scripts beside it and use `${CLAUDE_PLUGIN_ROOT}` in hook commands, for
  example `bash "${CLAUDE_PLUGIN_ROOT}/hooks/check.sh"`.
- **Status line:** edit `config/statusline.sh`. Its command is in
  `config/settings.json`. Install `jq` in the container for the detailed display;
  otherwise the script prints `danny | Claude`.
- **Settings:** edit `config/settings.json` to try personal preferences.

Start a fresh Claude session with the same command to reliably reload changes.
Status-line script edits appear on its next run. No container rebuild is needed
for ordinary edits.

To check the setup, invoke `/danny:hello` and expect `personal plugin loaded`.
After a response, the status line should begin with `danny |`. Make a small edit
inside `/home/vscode/claude-plugin` and confirm it appears in this host checkout.

## One-time mount setup

The test project's `.devcontainer/devcontainer.json` already declares this mount:

```json
"source=${localEnv:HOME}/personal/projects/claude-plugin,target=/home/vscode/claude-plugin,type=bind"
```

If an existing container still has the previous read-only/nested mounts, it needs
recreating once. **Save anything stored only in the container and exit its
sessions first:**

```bash
cd ~/personal/projects/dev-container
# Removes the old container; bind mounts/volumes retain their data.
devcontainer up --workspace-folder . --remove-existing-container
```

Do not use the removal flag for daily starts. On Linux, the container user also
needs write permission on the host files; the mount does not change ownership.

## What stays shared

This is a personal configuration layer, **not a separate sandbox**. The test
container still shares host Claude credentials, history, conversations, plans,
todos, and `.claude.json`. Hooks and tools can modify accessible files, and
interactive commands such as `/config` may save settings outside this checkout.
Prefer editing the files here for experiments.

Promoting your changes into the shared setup is a separate step when you're
happy with them. The checked-in status-line command uses the fixed container
path above.
