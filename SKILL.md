---
name: ux-research-orchestra
description: |
  Market intelligence orchestra. Multi-agent competitive analysis, benchmarking,
  quant research, and hypothesis framing. Outputs to Notion and/or Linear.
  Trigger: /market-analysis
---

# UX Research Orchestra — Conductor

## Inject

You are the Conductor. When `/market-analysis` is invoked, run this skill exactly.

### Step 1 — Parse parameters

Extract from the user's message:

| Param | Example | Default if missing |
|-------|---------|-------------------|
| `product` | "Klaro" | ask the user |
| `category` | "fiscal apps for freelancers" | ask the user |
| `region` | "DACH" / "Global" / "US" | "Global" |
| `competitors` | "Taxfix, ELSTER" | "auto" (agents discover) |
| `audience` | "product team" / "C-level" | "product team" |
| `output` | "notion" / "linear" / "both" / "file" | "file" |
| `language` | "English" / "German" | "English" |

If `product` or `category` are missing, ask before proceeding.

### Step 2 — Announce the run

Print a brief mission brief to the user:

```
Orchestra starting:
  Product:     {product}
  Category:    {category}
  Region:      {region}
  Competitors: {competitors}
  Audience:    {audience}
  Output:      {output}
  Language:    {language}

Spawning agents...
```

### Step 3 — Run agents in sequence

Run each agent by reading its SKILL.md and following it exactly.
Pass the full template block (Step 4) as context to every agent.
Collect each agent's `## Output` block before moving to the next.

**Sequence:**
```
Phase 1 — parallel (spawn both, wait for both)
  ├── Market Scout       → reads skills/market-scout/SKILL.md
  └── Quant Analyst      → reads skills/quant-analyst/SKILL.md

Phase 2 — sequential (needs Phase 1 output)
  └── Competitor Profiler → reads skills/competitor-profiler/SKILL.md
                            Input: Market Scout's competitor list

Phase 3 — parallel (needs Phase 2 output)
  ├── Benchmarker        → reads skills/benchmarker/SKILL.md
  └── Comparative Analyst → reads skills/comparative-analyst/SKILL.md

Phase 4 — sequential (needs Phase 3 output)
  └── Hypothesis Framer  → reads skills/hypothesis-framer/SKILL.md

Phase 5 — final
  └── Report Writer      → assembles all outputs → delivers
```

### Step 4 — Template block (inject into every agent)

```
--- RESEARCH CONTEXT ---
PRODUCT:     {product}
CATEGORY:    {category}
REGION:      {region}
COMPETITORS: {competitors}
AUDIENCE:    {audience}
LANGUAGE:    {language}
DATE:        {today's date}
------------------------
```

### Step 5 — Delivery

After the Report Writer completes:

**Notion destination (hardcoded):**
- Workspace: JGV DESIGN
- Parent page: 🗿 UX DESIGN FATHER PAGE (ID: 2e2e0dea8e0180f4a32ded87aab9bf07)
- Reports land inside a sub-page called "🔬 UX Research" (create it if missing)

- `output = notion` → call `mcp__claude_ai_Notion__notion-create-pages` with parent page_id `2e2e0dea8e0180f4a32ded87aab9bf07`
- `output = linear` → export hypothesis list as Linear issues (markdown format if MCP unavailable)
- `output = both`   → do both
- `output = file`   → save to `research/outputs/{product}-{date}-competitive-analysis.md`

Always confirm delivery to the user with a direct Notion URL or file path.

### Step 6 — Output contract (enforce for every agent)

Every agent MUST return exactly this structure — reject and re-run if missing:

```markdown
## Agent: [name]
## Status: complete | partial | blocked
## Blocked reason: [only if blocked]
## Output
[agent's actual content here]
## Passed to: [next agent name]
```

---

## Model routing

| Agent | Model | Tool |
|-------|-------|------|
| Market Scout | Claude Sonnet | firecrawl_search (fallback: WebSearch) |
| Quant Analyst | Claude Sonnet | firecrawl_search (fallback: WebSearch) |
| Competitor Profiler | Claude Sonnet | firecrawl_map + firecrawl_scrape |
| Benchmarker | Claude Sonnet | firecrawl_search → synthesis |
| Comparative Analyst | Claude Sonnet | Synthesis only |
| Hypothesis Framer | Claude Sonnet | Reasoning |

---

## Reference

### Adding a new agent
1. Create `skills/{agent-name}/SKILL.md`
2. Add it to the sequence in Step 3 above
3. Define where it fits in the phase diagram
4. Follow the output contract in Step 6

### Adding a new output destination
Add a condition in Step 5. Use the MCP tool if connected; fall back to markdown export.

### Adapting for Phase 2 (user research)
Phase 2 agents live in `skills/phase-2-users/`. The Conductor will route to them
when `/user-research` is triggered. They receive this orchestra's output as input.

### Canonical run
```
/market-analysis product="Klaro" category="fiscal apps for German freelancers" region="DACH" competitors="auto" audience="product team" output="notion" language="English"
```
