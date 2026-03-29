@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
**You're not ready to ship yet.** The ad-ship process has a hard gate:

> Must have passing verification evidence from ad-verify before shipping.

You said yourself you haven't run any tests. Before merging, you need to:

1. **Run the full verification chain:**
   ```bash
   npx tsc --noEmit && npx vitest run && npx biome check .
   ```

2. **Check blast radius** — make sure all consumers of any types/functions you changed still work through both CLI and API entry points (per AGENTS.md).

3. **Fix anything that fails** before attempting to ship.

Run `ad-verify` first (or at minimum run those three commands above), then come back with passing output and we can ship.