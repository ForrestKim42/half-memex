# Brain, Eyes, Hands

A pattern for making a single LLM-native workspace that covers what you know, how your agent moves, and what you're working on — all in one directory.

This is an idea file, designed to be copy-pasted to your own LLM agent (Claude Code, Codex, OpenCode, etc.). It describes a synthesis of two earlier ideas: [Karpathy's LLM wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and [ForrestKim's LLM app exploration](https://github.com/ForrestKim42/llm-app-exploration). Its goal is to communicate the high-level pattern; your agent will build out the specifics with you.

## The core idea

Most people run two completely separate LLM setups. On one side, there's a knowledge setup — a RAG pipeline, a NotebookLM, an Obsidian vault the LLM occasionally reads. On the other side, there's an automation setup — a browser agent, a mobile MCP, a desktop control tool. The knowledge side knows who you are but can't do anything. The automation side can do things but has no memory of who you are or what you're working on. Every session starts from scratch on both sides.

The idea here is different. Instead of keeping these separate, you put **everything the LLM needs to be useful to you in one directory**: what you know, how it moves through the apps you use, and what the current work is. The LLM reads from this directory, writes back into it, and every session makes the next one cheaper. Knowledge accumulates in one layer. Operational memory of apps accumulates in another. Short-term work happens in a third, and references the first two freely. The three layers are independent but cohabitate, and they explicitly reference each other.

This is the key move: **a single workspace where brain, eyes, and hands all live, each with its own layer, each with its own lifetime.** The brain layer answers "who are you?" It grows slowly over months and years and never really shrinks. The eyes-and-hands layer answers "how does the agent move through apps?" It grows whenever you explore a new app and stays alive as long as that app does. The work layer answers "what are we doing right now?" It's fast, ephemeral, and archives itself when the work is done.

You rarely write any of it yourself. The LLM writes the wiki pages as you feed it sources. The LLM writes the ROUTES and TRANSITIONS as it explores apps. You describe projects at a high level and the LLM fills in the details. Your job is to decide what matters, direct the exploration, and ask the right questions. The LLM does the bookkeeping, the maintenance, and the tedious graph-walking. In practice, I keep a single Claude Code session open with `ObsidianVault/` as its working directory. In one turn it updates an entity page in the wiki. In the next turn it adds a transition to an app's memory. In the next turn it's editing a plan inside a project, citing both the wiki and the app memory as it goes. One session, three layers, no context switching.

This pattern applies to anything where a single person (or small team) wants a durable, LLM-first workspace that covers both knowledge and action:

- **Personal research + execution** — deep-diving a domain while also driving the tools that operate in it. The wiki accumulates what you've learned; the app memory accumulates how to act on it; projects push specific moves.
- **Solo founder** — tracking people, companies, and product decisions in the wiki; remembering how to drive Slack, Notion, Figma, your dashboard in the app memory; running each initiative as a project.
- **Long-term investigations** — the wiki holds the evolving thesis, the app memory holds how to pull data out of various tools, the projects layer holds each specific question.
- **Anything LLM-driven that currently forgets everything between sessions.**

## Architecture

There are three layers.

**The wiki** — a directory of LLM-generated markdown files describing people, organizations, projects, concepts, intel, and the contexts that tie them together. The LLM owns this layer entirely. It creates pages, updates them when new sources arrive, maintains cross-references, and keeps the graph consistent. This is the brain. It's permanent and grows slowly. You read it; the LLM writes it. (This layer is lifted directly from Karpathy's LLM wiki.)

**The app memory** — a directory, one subdirectory per app, containing the agent's operational memory for that app: a route map of every screen, a transition table for every `screen + action → next screen`, and the screenshots that prove it. The LLM owns this layer too. It creates and updates these during exploration, and reads them to act efficiently when the user asks for something. This is the eyes and hands. It's permanent but app-scoped — if the app dies, that subdirectory can be archived. (This layer is lifted directly from ForrestKim's LLM app exploration.)

**The projects** — a directory of short-lived workspaces for specific initiatives. Each project has a README stating its goal, a STATE file tracking where it is, and whatever planning and scratch material the current work needs. Projects freely reference the wiki (for context about people and organizations) and the app memory (for knowing how to operate the tools they need). When a project ends, it moves to an archive. This is the work surface. It's fast and ephemeral.

**The schema** — a `CLAUDE.md` at the root plus one `CLAUDE.md` per layer. The root file describes the three-layer structure and the entry points. Each layer's own `CLAUDE.md` describes the conventions, link rules, and page formats specific to that layer. This is the configuration — what makes the LLM a disciplined workspace maintainer rather than a generic chatbot. You and the LLM co-evolve these over time.

## Identity and stability

Every layer uses stable, meaning-based identifiers. None use coordinates, list positions, or indexes.

```
wiki         → entity name       (e.g. "IoTrust", "Kim Seontae")
                = filename = H1 title
app-memory   → TYPE:Label         (e.g. BUTTON:Save, INPUT:Email, TAB:Home)
projects     → phase / task id    (e.g. phase-02, task-02-03)
```

Same state → same ID, regardless of session, screen resolution, UI layout, or time of day. This is what makes everything work. In the wiki, you can link `[[IoTrust]]` from any page without worrying about paths. In the app memory, you can pre-plan a five-screen flow as a flat sequence of `tap BUTTON:Confirm`, `tap BUTTON:Continue`, `tap BUTTON:Done` — no intermediate reads, no guessing. In projects, a task keeps its ID across restarts so plans survive context resets.

The wiki takes this one step further than Karpathy's original. Entities never link to each other directly. When two people are related, they're both linked from a **context page** that describes the relationship. "Kim Seontae knows Byun Sangmin" carries almost no information; "Kim Seontae and Byun Sangmin co-lead the strategy unit of Forest Crew, making top-level product decisions together" carries a lot. The context page holds the latter. This makes relationships first-class and prevents any single entity from becoming a link-hub black hole.

## Operations

**Ingest.** You drop a source into the raw collection — a document, a chat export, a transcript, a screenshot — and tell the LLM to process it. The LLM reads the source, extracts facts, updates the relevant wiki pages, flags contradictions against existing claims, and appends to the log. A single source might touch ten or fifteen wiki pages. Personally I like to ingest sources one at a time and stay involved, reading the updates as they happen. For large batches (a full Slack export, a year of chat logs) there's a deterministic scaffolding script, `scripts/ingest.sh`, that chunks the source, runs parallel fact extraction, consolidates with a larger model, and writes a delta file that gets applied back to the wiki. The scaffolding makes the process resumable and makes missed chunks visible.

**Explore.** You point the LLM at an app and tell it to build the route map. The agent reads the accessibility tree (mobile-mcp for Android/iOS, desktop-mcp for macOS), captures a screenshot, records the elements, and walks the app as a graph — depth-first. Every new screen gets a numbered screenshot and a row in `ROUTES.md`. Every action that changes the screen gets a line in `TRANSITIONS.md` as `screen + action → next screen`. Exploration can be shallow (visit every screen) or deep (execute real actions and capture confirmation screens, error states, and post-action flows). The depth is your call. (This operation is lifted verbatim from ForrestKim's pattern — see `app-memory/schema/METHODOLOGY.md` for the canonical version.)

**Work.** You start a project and the LLM sets up `projects/<name>/` with a README and a STATE file. Inside the project, the LLM can freely read from the wiki (to understand people and context) and the app memory (to know how to operate whatever tools the project touches). When the project needs a new piece of knowledge, that knowledge goes into the wiki, not the project folder — work borrows from the permanent layers but doesn't duplicate them. When the project ends, the folder moves to `projects/_archive/`.

**Lint.** Periodically, ask the LLM to health-check the workspace. Look for: wiki pages that contradict each other, orphaned entities with no inbound links, apps whose ROUTES and TRANSITIONS have drifted out of sync, projects whose STATE hasn't been touched in weeks. The wiki layer has a dedicated linter (`scripts/lint.sh`) that enforces the graph rules. The other layers are lint-ed by reading.

## Loading discipline

When the LLM works inside this directory, it loads one unit at a time, and loads that unit whole.

```
wiki layer       → one page at a time
app-memory layer → one app at a time: README + ROUTES + TRANSITIONS together
projects layer   → one project at a time: README + STATE together
```

ROUTES and TRANSITIONS are both small — a few kilobytes of markdown each. Loading them together for a given app costs almost nothing and makes every subsequent action on that app deterministic. Trying to save tokens by drilling in further (loading only parts of ROUTES, or only the relevant TRANSITIONS) is false economy: you pay for the next decision in accuracy instead of tokens, and accuracy is more expensive. Pick the unit, load it whole, finish inside it.

## Pattern and instance

Every layer has two kinds of files, and they never mix.

- A `CLAUDE.md` — the pattern. Slow-changing. Describes conventions, rules, page formats, link rules for that layer.
- An `INDEX.md` — the instance catalog. Fast-changing. A list of what's actually in the layer right now, with a one-line summary per entry.

This split is the single strongest rule in the workspace. Methodology and data never share a file. A catalog never contains TODOs. A schema never contains war stories. When they mix, both rot.

## Tools

You need very little to build this. Most of it is just markdown and an LLM agent. But a few specific tools earn their keep:

- **`scripts/ingest.sh`** — a deterministic 3-phase pipeline (chunk → parallel extract → consolidate → delta) that wraps LLM fact extraction and makes it resumable. Prevents silent dropouts on large sources.
- **`scripts/lint.sh`** — enforces the wiki's graph rules (context-mediated linking, no orphan entities, stable titles). `wiki/` only.
- **[mobile-mcp](https://github.com/ForrestKim42/mobile-mcp)** — the mobile side of app exploration. Android and iOS, accessibility tree based, single `mobile_do` tool that reads the screen and acts in one call.
- **[desktop-mcp](https://github.com/ForrestKim42/desktop-mcp)** — the desktop side. macOS native apps via the Accessibility API, Electron apps via Chrome DevTools Protocol. Single `desktop_do` tool, cross-app batching via `App/TYPE:Label`.
- **The methodology doc itself** — a copy of ForrestKim's original `README.md` lives at `app-memory/schema/METHODOLOGY.md` as the canonical reference for how exploration works.

Tools are scaffolding, not authority. They're deterministic helpers around an LLM that does the judgment. If any of them disappears, swap in an equivalent. The pattern is what matters.

## Why this works

Two separate bottlenecks get solved at the same time.

The bottleneck in knowledge work is maintenance. Humans abandon wikis because the bookkeeping burden grows faster than the value — updating cross-references, noting contradictions, keeping summaries current. LLMs don't get bored and can touch fifteen files in one pass. The wiki stays maintained because maintenance is near-free. (This is Karpathy's argument.)

The bottleneck in app automation is fragility. Coordinate-based automation breaks whenever the UI shifts. LLMs that read the accessibility tree target elements by semantic identity, which survives layout changes, resolution changes, and A/B tests. Once an app is mapped, acting on it becomes deterministic. (This is ForrestKim's argument.)

Put them in the same directory and a third thing happens: **context travels between layers for free**. When you start a project about negotiating with a partner company, the LLM already knows who works there (wiki), how to drive the tools you use to talk to them (app memory), and what the current state of the negotiation is (project). No explaining, no re-uploading, no re-indexing. The LLM's next action is always informed by everything you've ever told it and everywhere it's ever been.

The human's job is to curate sources, direct the exploration, decide which projects matter, and think about what it all means. The LLM's job is everything else.

This pattern is related in spirit to Vannevar Bush's Memex — a personal, curated store with associative trails between documents. Karpathy already noted the connection for the knowledge side. What's missing from Memex, and missing from every personal-wiki project that came after it, is the other half: a record of how the operator moves through the world, not just what they know about it. Brain without eyes and hands is half a Memex. This is the other half.

## Note

This document is intentionally abstract. It describes the pattern, not a specific implementation. The exact directory layout, file formats, linting rules, MCP tools, and ingestion scripts will depend on your domain, your apps, and your LLM of choice. Everything above is optional and modular — pick what's useful, ignore what isn't. The right way to use this is to share it with your LLM agent and instantiate a version that fits your needs. The document's only job is to communicate the pattern. Your LLM can figure out the rest.
