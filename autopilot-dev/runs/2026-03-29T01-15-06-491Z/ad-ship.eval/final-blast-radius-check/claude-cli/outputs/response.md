@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Per the ad-ship process, here's what you should run before marking ready:

### 1. Final verification gate

Run the full chain even though tests passed — code may have changed:

```bash
npx tsc --noEmit && npx vitest run && npx biome check .
```

### 2. Blast radius check (critical for your changes)

You modified `Task` (a shared interface) and `formatTask` (a shared utility). These are consumed across the codebase. Grep for all consumers:

```bash
git diff main...HEAD --name-only
```

Then check every consumer of your changed types:

```bash
grep -r "Task\b" --include="*.ts" src/services/ src/cli/ src/api/
grep -r "formatTask" --include="*.ts" src/
```

Per AGENTS.md: "Changes to services or models must be tested through both entry points" — so verify that **both CLI and API paths** exercise the modified `Task` interface correctly. Any consumer file not touched in your PR is a potential miss.

### 3. Risk classification

Your PR is **elevated risk** — it includes:
- **Schema/data model changes** (Task interface modification)
- **Changes to shared types with many consumers** (Task + formatTask)

This means it will need explicit confirmation before merging rather than auto-merge.

The blast radius check (step 2) is the most important one here. If you find consumers of `Task` or `formatTask` that your PR didn't update, those are the gaps to investigate before marking ready.