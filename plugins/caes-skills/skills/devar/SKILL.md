---
name: devar
description: Use when a request invokes @devar, mentions DEVAR work, or asks Codex to work on DEVAR-related projects. Scope the work only to Giving, Give, GivingService, GiftModels, and give-terminal unless the user explicitly names another repository or system.
---

# DEVAR

Treat requests that invoke this skill, including `@devar`, as DEVAR requests.

For DEVAR requests, the in-scope projects and repositories are:

- `Giving`
- `Give`
- `GivingService`
- `GiftModels`
- `give-terminal`

Do not assume any other repository, service, job, database, or system is in scope unless the user explicitly mentions it.

Prefer minimal, targeted changes. Avoid opportunistic refactors unless they are necessary for the requested change or the user explicitly asks for cleanup or refactoring.

Before editing files, follow any project-local agent instructions, repository guidance files, or coding standards that apply. If those instructions conflict with this skill, stop, identify the conflict clearly, and ask which instruction should take precedence.
