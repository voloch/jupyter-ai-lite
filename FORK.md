# voloch fork of jupyter-ai-lite

Branch `feat/manual-trigger-only`, based on upstream tag `v0.21.0`.

## What differs from upstream

A new setting, `completionManualTriggerOnly` (default `true`, also a switch in the
Jupyternaut settings panel). When on, `jupyternaut-lite` only calls the model for
inline completions invoked explicitly (`Alt+\`, command `inline-completer:invoke`).
Automatic as-you-type triggers return no items without a network request.

**Why:** in a notebook every completion request sends all code cells above and below
the cursor plus the system prompt, and JupyterLab fires an automatic request on every
pause (`debouncerDelay` defaults to 0). With a reasoning model this cost about
$0.03/min. Upstream has no manual-only option: jupyterlab/jupyterlab#16754 is open,
jupyterlab/jupyter-ai#968 was closed without one.

Set `completionManualTriggerOnly` to `false` to get upstream behaviour back.

The completion cleanup also strips only Markdown fence lines (```` ```python ````, ```` ``` ````)
instead of whole fenced blocks. Upstream's `/```[\s\S]*?```/` deleted the code when a model
(e.g. `qwen/qwen3-coder-next`) wrapped its completion in a fence, so nothing was shown.

**Shortcut:** on the Linux/Chrome setup this was built for, JupyterLab never matched the
default `Alt \` binding (nor `Alt /`), while `Alt W` worked. The working binding is a user
shortcut in `~/.jupyter/lab/user-settings/@jupyterlab/shortcuts-extension/shortcuts.jupyterlab-settings`:

```json
{ "shortcuts": [ { "command": "inline-completer:invoke", "keys": ["Alt I"],
                   "selector": ".jp-mod-completer-enabled", "preventDefault": true } ] }
```

**Secrets manager:** turn off "Use secrets manager" in the Jupyternaut settings.
`jupyter-secrets-manager` defaults to an in-memory store, so the key vanishes on page
reload and the completer silently has no model.

## Build and install

```bash
export PATH=<venv>/bin:$PATH          # needs jlpm and jupyter-builder from a JupyterLab venv
jlpm install --immutable
jlpm build:prod                        # builds both packages and the labextension
cd python/jupyternaut-lite
uv build --wheel -o ../../dist .       # hatch skips the JS build because the labextension exists
uv pip install --python <venv>/bin/python --reinstall --no-deps \
  ../../dist/jupyternaut_lite-0.21.0+manual-py3-none-any.whl
```

The `+manual` local version label keeps the `jupyter-ai-lite` pin `jupyternaut-lite==0.21.0`
satisfied while making the fork visible in `pip list`. Restart JupyterLab and hard-refresh
the browser after installing.

## Rebasing on a new upstream release

Re-apply the five-file change (schema, `tokens.ts`, `settings-model.ts`,
`completion-provider.ts`, `ai-settings.tsx`) on the new tag, bump the `+manual` base
version in `python/jupyternaut-lite/jupyternaut_lite/_version.py`, and rebuild.
