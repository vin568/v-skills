# Maintainer Skills

Collection repository for Codex maintainer workflow skills.

## Layout

Each skill lives under `skills/<skill-name>/` and remains independently installable.

```text
skills/
  code-review/
    SKILL.md
    agents/openai.yaml
```

## Install

Install a skill from this repository by pointing the installer at its skill path:

```bash
install-skill-from-github.py --repo <owner>/<repo> --path skills/code-review
```

Multiple skills can be installed from the same repository by passing multiple `--path` values.
