# Workspace Repository Map

A Spec Kit extension that generates `repo-map.md` for multi-repository opencode workspaces.

## What It Does

After `/speckit.tasks` creates a `tasks.md` for your feature, this extension generates a `repo-map.md` that translates the spec, plan, and tasks into a concrete multi-repository execution map. The map assigns every task to a domain agent (backend, frontend, infrastructure, e2e, environment), defines validation commands, and documents cross-repository dependencies — bridging Spec Kit planning with opencode subagent orchestration.

## Why It Exists

In multi-repository workspaces (such as `opencode-environment` with `workspace/backend/`, `workspace/frontend/`, and `workspace/infrastructure/`), task ownership is often ambiguous. `repo-map.md` provides a routing contract so orchestrator agents know exactly which subagent handles each task, what commands are permitted, and how cross-repo dependencies must be sequenced.

## Installation

### Local Development

```bash
specify extension add --dev /path/to/spec-kit-workspace-map
```

### Configuration

Copy and customize the template:

```bash
cp .specify/extensions/workspace-map/workspace-map-config.template.yml \
   .specify/extensions/workspace-map/workspace-map-config.yml
```

Edit `workspace-map-config.yml` to match your workspace layout, agent names, and allowed commands.

## Usage

### Manual Invocation

Run the command after `/speckit.tasks`:

```text
/speckit.workspace-map.generate
```

The command reads your feature's `spec.md`, `plan.md`, `tasks.md`, and the workspace configuration, then generates:

```text
specs/<feature>/repo-map.md
```

### Automatic Hook

The extension registers an optional `after_tasks` hook. After `/speckit.tasks` completes, you will be prompted:

> Generate repo-map.md for this feature?

You can accept (run the generation) or decline (skip it). The hook is optional by design — it never executes silently.

## Reviewing the Generated Map

After generation, review `specs/<feature>/repo-map.md` for:

- **Mapping Confidence** — Are any assignments flagged as `low` or `medium`?
- **Unresolved Inputs** — Are repo/project names filled in or are there placeholders?
- **Low-Confidence Assignments** — Do any tasks need a human decision?
- **Cross-Repository Dependencies** — Are blocking dependencies correctly identified?
- **Execution Order** — Does the sequence respect cross-repo constraints?

The file starts with `Status: needs-review` — update it manually once you have verified the assignments.

## Known Limitations

- **Task ownership is inferred** — keywords and plan context drive classification, which may be imperfect.
- **Output requires human review** — always inspect assignments, dependencies, and validation commands.
- **Project names may need placeholders** — if repo/project names cannot be determined from the feature artifacts, the map uses `<backend-project>` style placeholders.
- **The extension does not execute implementation** — it produces a map; actual implementation is delegated to domain agents.
