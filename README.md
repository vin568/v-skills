# V Skills

Collection repository for agent workflow skills.

## Layout

Each skill lives under `skills/<skill-name>/` and remains independently installable.

```text
skills/
  architecture-fit-review/
    SKILL.md
    agents/openai.yaml
  test-review/
    SKILL.md
    anti-patterns.md
    agents/openai.yaml
```

## Install With `npx skills`

The Skills CLI requires Node.js `>=22.20.0`.

List available skills from GitHub:

```bash
npx skills add vin568/v-skills --list
```

List available skills from a local checkout:

```bash
npx skills add /path/to/v-skills --list
```

If a local checkout unexpectedly reports no skills, confirm the exact skill path:

```bash
npx skills add /path/to/v-skills/skills/architecture-fit-review --list
```

Install the `architecture-fit-review` skill globally:

```bash
npx skills add vin568/v-skills --skill architecture-fit-review -g -y
```

Install the `test-review` skill globally:

```bash
npx skills add vin568/v-skills --skill test-review -g -y
```

Install the `autopilot-grilling` skill globally:

```bash
npx skills add vin568/v-skills --skill autopilot-grilling -g -y
```

Install every skill in the repository globally:

```bash
npx skills add vin568/v-skills --all -g
```
