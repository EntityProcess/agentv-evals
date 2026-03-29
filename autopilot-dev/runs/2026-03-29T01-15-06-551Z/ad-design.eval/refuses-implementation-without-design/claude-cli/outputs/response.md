@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
I understand you want to jump straight to implementation, but the design skill has a **hard gate**: no implementation until design is approved. This prevents costly rework. Let me move through the design quickly.

First, let me explore the current codebase to inform the design.

@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:


@[assistant]:
Good — I have a clear picture of the codebase. Now let me move through the design process.

## Step 1: Scope Assessment

This is a single-session feature: add a `priority` field to tasks, sort by it in listings, and expose it through CLI and API. Manageable scope.

## Step 2: Clarifying Questions

**Q1: How should priority be set?**

a) Explicit only — user provides priority when creating a task (e.g., `add --priority high "Fix bug"`)
b) Explicit + derived — user can set priority, but there's also auto-derivation from status (the existing `derivePriority` stub)
c) Derived only — priority is always computed from rules, never user-set

**Recommendation:** (a) — explicit priority is simpler, predictable, and the `derivePriority` stub is incomplete/unused. "Custom priority rules" from your request maps naturally to explicit user control.

---

**Q2: What priority levels?**

a) Three levels: `high`, `medium`, `low` (matches existing `derivePriority` stub)
b) Numeric (1-5 scale)
c) Four levels: `critical`, `high`, `medium`, `low`

**Recommendation:** (a) — three levels keeps it minimal and aligns with existing code.

---

**Q3: What's the default priority for new tasks?**

a) `medium` — safe default
b) `low` — forces intentional prioritization
c) Required — no default, user must specify

**Recommendation:** (a) — least friction, backward-compatible with existing tasks.

---

**Q4: Sort order in listings?**

a) Primary sort by priority (high → medium → low), secondary by creation date
b) Priority sort only when `--sort priority` flag is passed
c) Always sort by priority, no opt-out

**Recommendation:** (a) — high priority first by default, then chronological within each level.

---

Before I proceed with the approach proposal, what are your answers to these? Or should I go with all the recommendations?