---
name: implementation-plan-review
description: Review an implementation plan from a senior repository maintainer perspective before code is written. Use when the user asks whether a proposed technical plan, design, approach, task breakdown, or implementation strategy fits the existing codebase; which planned changes are required versus incidental refactoring; and what the smallest low-risk, reversible, verifiable implementation path should be.
---

# Implementation Plan Review

Review the proposed plan only. Do not modify code or begin implementation unless the user explicitly asks after the review.

## Establish the Requirement

Restate the smallest external behavior or acceptance criteria the plan must satisfy. Identify constraints stated by the user.

If the requirement is incomplete, list the assumptions needed for the review. Do not treat choices already present in the plan as requirements, and do not expand the requirement to justify the proposed design.

## Find Repository Evidence

Before evaluating the plan, inspect the codebase for the closest existing implementations. Search relevant:

- module and service boundaries
- dependency direction and public entry points
- shared helpers, adapters, middleware, and configuration mechanisms
- naming, error-handling, and lifecycle conventions
- tests covering similar behavior

Use concrete file paths and key symbols as evidence. Explain why each example is similar and which pattern the plan should reuse.

Distinguish repository facts from general engineering advice. If no sufficiently similar implementation exists, say so explicitly.

## Evaluate the Plan

Judge the plan against three questions:

1. Does it follow the repository's existing architecture, boundaries, abstraction level, dependency direction, and conventions?
2. Which planned changes are required to satisfy the stated behavior, which are optional improvements, and which are unrelated cleanup or opportunistic refactoring?
3. What is the smallest low-risk, reversible, independently verifiable implementation path?

Prefer existing mechanisms. Recommend a new abstraction, infrastructure component, configuration surface, or behavior boundary only when existing mechanisms cannot satisfy the requirement. Explain that limitation with repository evidence.

Focus on concrete correctness, compatibility, migration, security, concurrency, data-integrity, lifecycle, rollout, and regression risks. Avoid generic style preferences and speculative future requirements.

## Output

Use the user's language and return:

1. `总体结论`
   - Choose `可以采用`, `修改后采用`, or `不建议采用`.
   - State the main reason briefly.
2. `代码库依据`
   - List relevant file paths, key symbols, similarities, and reusable patterns.
3. `必需改动`
   - Include only changes required to satisfy the stated behavior.
4. `可选但非必需`
   - List useful improvements that should not block delivery.
5. `不要改的范围`
   - Identify unrelated refactors, cleanup, abstractions, and modules to exclude.
6. `推荐实现路径`
   - Give ordered, minimal steps and identify affected files or modules when known.
7. `风险点`
   - Separate blocking risks, ordinary risks, and longer-term risks.
8. `测试与验证`
   - Recommend focused tests first, then necessary regression and manual checks.
   - Prefer the repository's existing test patterns and commands.

Keep the review concise and evidence-backed. Clearly label assumptions and unresolved questions. If the proposed plan is already minimal and consistent, say so instead of inventing changes.
