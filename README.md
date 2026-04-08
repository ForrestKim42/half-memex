# Brain, Eyes, Hands

A pattern for making a single LLM-native workspace that covers what you know, how your agent moves, and what you're working on — all in one directory.

This is an idea file, meant to be copy-pasted into your own LLM agent (Claude Code, Codex, OpenCode, or whichever one you use). It describes a synthesis of two earlier ideas — [Karpathy's LLM wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and [ForrestKim's LLM app exploration](https://github.com/ForrestKim42/llm-app-exploration) — and its only job is to communicate the pattern. Your agent will work out the specifics with you.

## The core idea

Most people end up running two completely separate LLM setups. One is for knowledge: a RAG pipeline, a NotebookLM, an Obsidian vault the model occasionally reads. The other is for action: a browser agent, a mobile MCP, some desktop control tool. The knowledge side knows who you are but can't do anything. The action side can do things but has no idea who you are or what you're working on. Both sides start from zero every session, and neither ever hears about the other.

The idea here is different. Instead of keeping the two halves apart, you put **everything the LLM needs to be useful to you in a single directory** — what you know, how the agent moves through the apps you use, and what you're actually working on right now. The LLM reads from this directory, writes back into it, and every session makes the next one cheaper. Knowledge accumulates in one layer. Operational memory of apps accumulates in another. Short-term work happens in a third, and it pulls freely from the other two. The layers are independent but they share a roof, and they reference each other out in the open.

This is the key move: **a single workspace where brain, eyes, and hands all live, each in its own layer, each with its own lifetime.** The brain layer answers *who are you?* It grows slowly over months and years, and it never really shrinks. The eyes-and-hands layer answers *how does the agent move through this app?* It grows whenever you explore something new and stays alive as long as that app does. The work layer answers *what are we doing right now?* — fast, ephemeral, and self-archiving when the work is done.

You rarely write any of it yourself. The LLM writes the wiki pages as you feed it sources. The LLM writes the ROUTES and TRANSITIONS as it explores apps. You sketch a project at a high level and the LLM fills in the rest. Your job is to decide what matters, point the agent at the next thing, and ask good questions. Everything else — the bookkeeping, the maintenance, the tedious graph-walking — belongs to the model. In practice, I keep a single Claude Code session open with `ObsidianVault/` as its working directory, and the three layers rotate under one conversation: one turn updates an entity page in the wiki, the next adds a transition to an app's memory, the next edits a plan inside a project and cites both as it goes. One session, three layers, no context switching.

This pattern applies to anything where a single person (or a small team) wants a durable, LLM-first workspace that covers both knowledge and action:

- **Personal research paired with execution** — going deep on a domain while also driving the tools that operate in it. The wiki holds what you've learned; the app memory holds how to act on it; projects drive specific moves.
- **Solo founder** — people, companies, and product decisions in the wiki; Slack, Notion, Figma, and your dashboard in the app memory; every initiative as its own project.
- **Long-term investigations** — the wiki carries the evolving thesis, the app memory carries how to pull data out of the tools you rely on, and projects carry each specific question as it comes up.
- **Anything LLM-driven that currently forgets everything between sessions.**

## Architecture

There are three layers, plus a schema that ties them together.

**The wiki** — a directory of LLM-generated markdown files covering people, organizations, projects, concepts, intel, and the contexts that bind them. The LLM owns this layer outright. It creates pages, updates them as new sources arrive, maintains cross-references, and keeps the graph consistent. This is the brain: permanent, slow-growing, and read-mostly for humans. The pattern is lifted directly from Karpathy.

**The app memory** — one subdirectory per app, each holding the agent's operational memory for that app: a route map of every screen, a transition table recording `screen + action → next screen` for every action worth remembering, and the screenshots that serve as evidence. The LLM creates and updates these during exploration and reads them when it needs to act. This is the eyes and hands: permanent but app-scoped, and archivable if the app itself goes away. The pattern is lifted directly from ForrestKim.

**The projects** — short-lived workspaces, one per initiative. Each has a README describing its goal, a STATE file tracking where it is, and whatever planning and scratch material the current work needs. Projects reference the wiki freely for context on people and organizations, and the app memory for knowing how to drive the tools they depend on. When a project ends, it moves to an archive. This is the work surface: fast, ephemeral, and always in motion.

**The schema** — a `CLAUDE.md` at the root, plus one inside each layer. The root file lays out the three-layer structure and the entry points. Each layer's own `CLAUDE.md` covers the conventions, link rules, and page formats specific to that layer. This is the configuration that turns the LLM into a disciplined workspace maintainer instead of a generic chatbot, and you co-evolve it with the model over time.

## Identity and stability

Every layer uses stable, meaning-based identifiers. None of them rely on coordinates, list positions, or indexes.

```
wiki         → entity name        (e.g. "Acme Inc", "Jane Doe")
               = filename = H1 title
app-memory   → TYPE:Label          (e.g. BUTTON:Save, INPUT:Email, TAB:Home)
projects     → phase / task id     (e.g. phase-02, task-02-03)
```

Same state, same ID — regardless of session, screen resolution, UI layout, or time of day. This is what makes the rest of the workspace possible. In the wiki you can write `[[Acme Inc]]` from any page and not think about paths. In the app memory you can plan a five-screen flow as a flat sequence — `tap BUTTON:Confirm`, `tap BUTTON:Continue`, `tap BUTTON:Done` — with no intermediate reads and no guessing. In projects, a task keeps its ID across restarts, so plans survive context resets.

The wiki takes this one step past Karpathy's original. Entities never link to each other directly. When two people are related, both are linked from a **context page** that describes the relationship. "Alice knows Bob" carries almost no information; "Alice and Bob co-lead the strategy unit at Acme and make top-level product decisions together" carries a lot. The context page holds the latter. Relationships become first-class citizens, and no entity ever turns into a link-hub that collapses under its own inbound weight.

## Operations

**Ingest.** You drop a source into the raw collection — a document, a chat export, a transcript, a screenshot — and hand it to the LLM. The model reads it, pulls out facts, updates the relevant wiki pages, flags anything that contradicts existing claims, and appends to the log. One source might touch ten or fifteen pages. I personally prefer to ingest sources one at a time and stay in the loop, reading each update as it lands. For larger batches — a full Slack export, a year of chat logs — there's `scripts/ingest.sh`, a deterministic three-phase scaffold (chunk, parallel fact extraction, consolidation, delta) that wraps the whole thing, keeps it resumable, and makes skipped chunks loud instead of silent.

**Explore.** You point the LLM at an app and ask it to build the route map. The agent reads the accessibility tree (mobile-mcp for Android and iOS, desktop-mcp for macOS), takes a screenshot, records the elements, and walks the app depth-first as a graph. Every new screen earns a numbered screenshot and a row in `ROUTES.md`; every action that changes the screen earns a line in `TRANSITIONS.md` of the form `screen + action → next screen`. Exploration can be shallow — just visit every screen — or deep, meaning the agent actually executes real actions and captures confirmation screens, error states, and post-action flows. The depth is your call. (The canonical version of this operation lives at `app-memory/schema/METHODOLOGY.md`, a copy of ForrestKim's original.)

**Work.** You start a project and the LLM sets up `projects/<name>/` with a README and a STATE file. Inside the project the model reads from the wiki freely (for the people and the context) and from the app memory freely (for the tools it needs to drive). If the project turns up a new piece of durable knowledge, that knowledge goes into the wiki — not into the project folder. Projects borrow from the permanent layers; they never duplicate them. When the work is done, the folder moves to `projects/_archive/`.

**Lint.** Every so often, ask the LLM to health-check the workspace. It looks for wiki pages that contradict each other, entities with no inbound links, apps whose ROUTES and TRANSITIONS have drifted apart, and projects whose STATE hasn't moved in weeks. The wiki layer has a dedicated linter — `scripts/lint.sh` — that enforces its graph rules mechanically. The other layers get linted by the act of reading them.

## Loading discipline

When the LLM works inside this directory, it loads one unit at a time, and it loads that unit whole.

```
wiki         → one page at a time
app-memory   → one app at a time: README + ROUTES + TRANSITIONS together
projects     → one project at a time: README + STATE together
```

ROUTES and TRANSITIONS are small — a few kilobytes each. Loading them together for a given app costs almost nothing and makes every subsequent action on that app deterministic. Trying to save tokens by drilling in further — reading only part of ROUTES, or only the transitions that look relevant — is false economy. The cost you avoid in tokens comes back in bad decisions, and bad decisions are much more expensive. Pick the unit, load it whole, and finish inside it.

## Pattern and instance

Every layer has two kinds of files, and they never share the same document.

- **`CLAUDE.md`** — the pattern. Slow-changing. Conventions, rules, page formats, link rules for that layer.
- **`INDEX.md`** — the instance catalog. Fast-changing. A list of what's actually in the layer right now, one line per entry.

This split is the single strongest rule in the workspace. Methodology and data never share a file. A catalog never contains TODOs. A schema never contains war stories. Mix them and both rot — the methodology gets buried in churn, and the catalog gets heavy with rules nobody reads.

## Tools

You need very little to build this. Most of it is just markdown and an LLM agent. A few specific pieces earn their keep:

- **`scripts/ingest.sh`** — a three-phase pipeline (chunk → parallel extract → consolidate → delta) that wraps LLM fact extraction and makes it resumable. Stops large sources from silently dropping chunks.
- **`scripts/lint.sh`** — enforces the wiki's graph rules: context-mediated linking, no orphan entities, stable titles. `wiki/` only.
- **[mobile-mcp](https://github.com/ForrestKim42/mobile-mcp)** — the mobile side of exploration. Android and iOS, accessibility-tree based, exposing a single `mobile_do` tool that reads the screen and acts in one call.
- **[desktop-mcp](https://github.com/ForrestKim42/desktop-mcp)** — the desktop side. macOS native apps via the Accessibility API, Electron apps via the Chrome DevTools Protocol. Single `desktop_do` tool, cross-app batching through the `App/TYPE:Label` prefix.
- **The methodology itself** — a copy of ForrestKim's original `README.md` lives at `app-memory/schema/METHODOLOGY.md` as the canonical reference for how exploration works.

Tools are scaffolding, not authority. They're deterministic helpers wrapped around an LLM that does the actual judgment. If any of them disappears, swap in an equivalent. The pattern is what matters.

## Why this works

Two separate bottlenecks get solved at the same time.

The bottleneck in knowledge work has always been maintenance. People abandon wikis because the bookkeeping grows faster than the value — cross-references to update, contradictions to flag, summaries to keep current, dozens of pages to hold in your head at once. LLMs don't get bored and can touch fifteen files in a single pass. The wiki stays alive because maintenance costs almost nothing. (That's Karpathy's argument.)

The bottleneck in app automation has always been fragility. Coordinate-based automation breaks the moment the UI shifts a few pixels. An agent that reads the accessibility tree identifies elements by their semantic identity, and semantic identity survives layout changes, resolution changes, and A/B tests. Once an app is mapped, acting on it becomes deterministic. (That's ForrestKim's argument.)

Put both in the same directory and a third thing happens on top: **context travels between the layers for free**. The moment you start a project about negotiating with a partner company, the LLM already knows who works there (wiki), how to drive the tools you'll use to talk to them (app memory), and where the negotiation currently stands (project). No explaining, no re-uploading, no re-indexing. Each new move is informed by everything that came before.

The human's job is to curate sources, direct the exploration, decide which projects matter, and think about what it all means. The LLM's job is everything else.

The whole thing is related in spirit to Vannevar Bush's Memex — a personal, curated store with associative trails between documents. Karpathy already drew the connection on the knowledge side. What's missing from Memex, and from every personal-wiki project that came after it, is the other half: a record of how the operator *moves* through the world, not just what they know about it. Brain without eyes and hands is half a Memex. This is the other half.

## Note

This document is intentionally abstract. It describes the pattern, not a specific implementation. The exact directory layout, file formats, linting rules, MCP tools, and ingestion scripts will all depend on your domain, your apps, and your LLM of choice. Everything above is optional and modular — pick what's useful and ignore the rest. The right way to use it is to share it with your LLM agent and work out a version that fits your life. The document's only job is to communicate the pattern. Your LLM can handle the rest.
