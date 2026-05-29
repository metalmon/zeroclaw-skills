# zeroclaw-skills

A collection of [ZeroClaw](https://github.com/zeroclaw-labs/zeroclaw) skills.

Skills are Markdown bundles that teach an agent how to use a capability
reliably. They follow the open-skills layout: each skill lives in its own
directory under `skills/<name>/SKILL.md` and is loaded on demand via the
`read_skill` tool.

## Layout

```
skills/
  <skill-name>/
    SKILL.md
```

## Skills

| Skill | Purpose | Requires |
|-------|---------|----------|
| [`office-documents`](skills/office-documents/SKILL.md) | Extract clean text / Markdown from Microsoft Office files (`.docx`, `.xlsx`, `.pptx`, `.doc`, `.xls`, `.ppt`) without flooding the model's context with raw base64. | The `office-tools` WASM plugin (provides the `office_read` tool) and the native `file_read` / `execute_pipeline` tools. |

### office-documents — companion skill for the `office-tools` plugin

This skill does **not** parse documents itself. It is the instruction layer for
the [`office-tools`](https://github.com/metalmon/zeroclaw/tree/main/plugins/office-tools)
WASM plugin, which exposes the `office_read` tool. The skill teaches the agent
to chain `file_read` (base64) → `office_read` through `execute_pipeline` with
`"result": "last"`, so binary bytes are passed machine-to-machine and only the
extracted text returns to the model. Without the plugin installed, the
`office_read` tool referenced in the recipe will not exist.

## Using this collection

Point a ZeroClaw install at this repository as its open-skills source, or copy
individual skill directories into your install's `shared/skills/`.

To use it as the open-skills repo, clone it into the configured open-skills
directory and enable open skills in `config.toml`:

```toml
[skills]
open_skills_enabled = true
open_skills_dir = "/path/to/this/clone"
```

ZeroClaw discovers skills under `skills/<name>/SKILL.md` and keeps the clone
up to date with `git pull --ff-only`.
