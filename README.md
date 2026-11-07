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

## Development Notes

- [`docs/review-loop-convergence.md`](docs/review-loop-convergence.md) — evidence-based analysis of why multi-round review/audit loops fail to converge, and the design rules review skills in this repo should follow (declared stopping condition, frozen cross-round checklist, round budget, defect-vs-opinion split, escalation of near-ties, oracle-backed high-severity claims).

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
