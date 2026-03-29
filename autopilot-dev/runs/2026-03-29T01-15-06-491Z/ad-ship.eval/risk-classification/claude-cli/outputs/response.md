@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
**Do not auto-merge.** This PR hits three elevated-risk categories from the ship checklist:

| Risk factor | Why it applies |
|---|---|
| Breaking API changes | `priority` is a new **required** field — any existing consumer that creates/updates tasks without it will break |
| Schema / data model changes | The `Task` interface changed |
| Changes to shared types with many consumers | `Task` is used across CLI, API, and services per the architecture |

You should get review first.

---

Before we can actually ship, the ad-ship process requires running the full check chain and blast radius check. Want me to proceed with the ship process (Steps 1-3: final verification, blast radius grep, push and mark ready for review)?