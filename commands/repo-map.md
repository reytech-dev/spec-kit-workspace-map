---
description: "Generate repo-map.md for a multi-repo opencode workspace"
---

# Generate Workspace Repository Map

## User Input

$ARGUMENTS

## Purpose

Generate `repo-map.md` for the current Spec Kit feature.

The map must translate `spec.md`, `plan.md`, and `tasks.md` into a concrete multi-repository execution map for opencode agents.

## Inputs

Read:

1. `AGENTS.md`
2. `.specify/memory/constitution.md`, if present
3. The current feature's `spec.md`
4. The current feature's `plan.md`
5. The current feature's `tasks.md`
6. `.specify/extensions/workspace-map/workspace-map-config.yml`, if present
7. `.specify/extensions/workspace-map/workspace-map-config.template.yml` as fallback

## Output

Create or update:

```text
specs/<feature>/repo-map.md
```

Do not modify:

```text
spec.md
plan.md
tasks.md
constitution.md
AGENTS.md
```

## Hard Rules

* Assign every implementation task to one primary area:

  * backend
  * frontend
  * infrastructure
  * e2e
  * environment
* Allow secondary areas when a task has cross-repo impact.
* Do not invent repository names.
* If project names cannot be inferred, use placeholders such as `<backend-project>` and list them under `Unresolved Inputs`.
* Every assigned area must include validation commands.
* Validation commands must use only:

  * `./dev/scripts/wrapper.sh`
  * `./dev/scripts/exec.sh`
* Never output direct commands using:

  * `pnpm`
  * `npm`
  * `node`
  * `npx`
  * `gradlew`
  * `tofu`
  * `docker compose`
* Do not recommend automatic infrastructure apply unless explicitly approved.
* Mark the generated map as `Status: needs-review` by default.
* Add a `Mapping Confidence` section.
* Add `Low-Confidence Assignments` when ownership is uncertain.
* Add `Cross-Repository Dependencies` whenever more than one area is involved.
* Add an `Execution Order` section.

## Required `repo-map.md` Format

Generate the file with this structure:

```markdown
# Repository Map

Feature: <feature-id-and-name>
Status: needs-review

## Source Artifacts

- Specification: `specs/<feature>/spec.md`
- Plan: `specs/<feature>/plan.md`
- Tasks: `specs/<feature>/tasks.md`
- Workspace policy: `AGENTS.md`

## Mapping Confidence

Status: needs-review

Summary:
- <short summary of confidence and assumptions>

## Unresolved Inputs

- <list unresolved repo/project names, missing paths, or ambiguous ownership>
- Use `None` if there are no unresolved inputs.

## Repository Allocation

### Backend

- Workspace path: `workspace/backend/<backend-project>`
- Agent: `backend`
- Runner: `java-runner`
- Primary tasks:
  - T...
- Secondary tasks:
  - T...
- Allowed commands:
  - `./dev/scripts/wrapper.sh backend:test <backend-project>`
- Validation:
  - `./dev/scripts/wrapper.sh backend:test <backend-project>`
- Notes:
  - <backend-specific notes>

### Frontend

- Workspace path: `workspace/frontend/<frontend-project>`
- Agent: `frontend`
- Runner: `node-runner`
- Primary tasks:
  - T...
- Secondary tasks:
  - T...
- Allowed commands:
  - `./dev/scripts/wrapper.sh frontend:test <frontend-project>`
  - `./dev/scripts/wrapper.sh frontend:build <frontend-project>`
- Validation:
  - `./dev/scripts/wrapper.sh frontend:test <frontend-project>`
  - `./dev/scripts/wrapper.sh frontend:build <frontend-project>`
- Notes:
  - <frontend-specific notes>

### Infrastructure

- Workspace path: `workspace/infrastructure/<infrastructure-project>`
- Agent: `infrastructure`
- Runner: `opentofu-runner`
- Primary tasks:
  - T...
- Secondary tasks:
  - T...
- Allowed commands:
  - `./dev/scripts/wrapper.sh infrastructure:validate <infrastructure-project>`
  - `./dev/scripts/wrapper.sh infrastructure:plan <infrastructure-project>`
- Validation:
  - `./dev/scripts/wrapper.sh infrastructure:validate <infrastructure-project>`
  - `./dev/scripts/wrapper.sh infrastructure:plan <infrastructure-project>`
- Notes:
  - Do not run apply unless explicitly approved.

### E2E

- Agent: `e2e`
- Runner: `playwright-runner`
- Primary tasks:
  - T...
- Validation:
  - `./dev/scripts/wrapper.sh backend:start <backend-project>`
  - `./dev/scripts/wrapper.sh frontend:start <frontend-project>`
  - `./dev/scripts/wrapper.sh frontend:e2e <frontend-project>`
  - `./dev/scripts/wrapper.sh frontend:stop`
  - `./dev/scripts/wrapper.sh backend:stop`

### Environment

- Workspace path: `.`
- Agent: `environment`
- Runner: `opencode`
- Primary tasks:
  - T...
- Validation:
  - `./dev/scripts/exec.sh compose:ps`
- Notes:
  - Use this area only for changes to the development environment, wrapper scripts, runner images, or Docker Compose topology.

## Task Mapping

| Task | Primary Area | Secondary Areas | Reason | Validation |
|---|---|---|---|---|
| T001 | backend | frontend | <reason> | `<command>` |

## Cross-Repository Dependencies

| From | To | Dependency | Blocking? | Notes |
|---|---|---|---|---|
| frontend | backend | API contract | yes | Frontend depends on backend contract stability |

## Execution Order

1. <first task or group>
2. <second task or group>
3. <third task or group>

## Low-Confidence Assignments

| Task | Proposed Area | Confidence | Reason | Needs Human Decision |
|---|---|---|---|---|
| T... | backend/frontend | medium | <reason> | yes |

Use `None` if all assignments are high confidence.

## Agent Handoff Instructions

### Orchestrator

Use this file as the routing contract. Do not implement directly. Delegate only the relevant task subset to each domain agent.

### Backend Agent

Implement only backend-assigned tasks. Do not edit frontend, infrastructure, or environment files unless explicitly listed as secondary impact. Use only approved wrapper or exec commands.

### Frontend Agent

Implement only frontend-assigned tasks. Do not edit backend, infrastructure, or environment files unless explicitly listed as secondary impact. Use only approved wrapper or exec commands.

### Infrastructure Agent

Implement only infrastructure-assigned tasks. Do not run apply unless explicitly approved. Use only approved wrapper or exec commands.

### E2E Agent

Run cross-service validation only after backend and frontend implementation are complete.

### Environment Agent

Modify only the development environment, wrapper scripts, runner definitions, or Compose configuration when tasks explicitly require it.
```

## Classification Behavior

When mapping tasks:

* Backend indicators:

  * API
  * Java
  * Spring
  * database
  * persistence
  * migration
  * service
  * domain
  * integration test

* Frontend indicators:

  * UI
  * page
  * component
  * state
  * client
  * route
  * form
  * accessibility
  * generated client
  * TypeScript

* Infrastructure indicators:

  * OpenTofu
  * Terraform
  * IaC
  * environment variable
  * secret
  * bucket
  * database provisioning
  * network
  * deployment resource

* E2E indicators:

  * Playwright
  * end-to-end
  * user journey
  * full flow
  * browser test
  * frontend plus backend validation

* Environment indicators:

  * Docker Compose
  * runner image
  * wrapper script
  * exec script
  * opencode config
  * local development environment
  * service topology

If a task clearly touches multiple areas, assign one primary area and list the others as secondary areas.

## Confidence Rules

Use confidence levels:

```text
high
medium
low
```

Use `high` when task ownership is explicit.

Use `medium` when task ownership is inferred from keywords or plan context.

Use `low` when:

* repo/project name is unknown,
* task wording is ambiguous,
* task may require multiple repos,
* validation command cannot be determined,
* execution order is unclear.

Low-confidence items must appear in `Low-Confidence Assignments`.

## Validation Requirements

The generated repo map must validate that:

1. No direct forbidden commands are present.
2. Every area with assigned tasks has validation commands.
3. Every implementation task from `tasks.md` appears in the task mapping.
4. Unknown repo/project names are not invented.
5. Infrastructure apply is not recommended automatically.
6. E2E validation is included when both frontend and backend are affected.
7. The file starts with `Status: needs-review`.
