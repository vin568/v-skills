# Maintainer Skills

Collection repository for Codex maintainer workflow skills.

## Layout

Each skill lives under `skills/<skill-name>/` and remains independently installable.

```text
skills/
  maintainer-review/
    SKILL.md
    agents/openai.yaml
```

## Install With `npx skills`

The Skills CLI requires Node.js `>=22.20.0`.

List available skills after this repository is pushed to GitHub:

```bash
npx skills add <owner>/maintainer-skills --list
```

List available skills from a local checkout:

```bash
npx skills add /path/to/maintainer-skills --list
```

If a local checkout unexpectedly reports no skills, confirm the exact skill path:

```bash
npx skills add /path/to/maintainer-skills/skills/maintainer-review --list
```

Install the `maintainer-review` skill globally:

```bash
npx skills add <owner>/maintainer-skills --skill maintainer-review -g -y
```

Install every skill in the repository globally:

```bash
npx skills add <owner>/maintainer-skills --all -g
```

Replace `<owner>` with the GitHub account or organization that owns the repository.
