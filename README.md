# Brain, Eyes, Hands

A pattern for a single LLM-native workspace covering what you know, how your agent moves, and what you're working on — all in one directory.

This is an idea file, meant to be copy-pasted into your own LLM agent (Claude Code, Codex, OpenCode, or whichever one you use). It describes a synthesis of two earlier ideas — [Karpathy's LLM wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and [ForrestKim's LLM app exploration](https://github.com/ForrestKim42/llm-app-exploration) — lifted to the same altitude and put under one roof. Its only job is to communicate the pattern. Your agent will work out the specifics with you.

## Two bottlenecks, one shape

**Knowledge.** Most LLM-and-documents setups look like RAG: upload files, retrieve at query time, answer. The LLM rediscovers everything from scratch on every question. A subtle question that needs five documents synthesized gets answered by re-finding and re-piecing those five documents every single time. Nothing accumulates.

**Action.** Most LLM-and-apps setups look like a blind agent: read the screen, guess, tap, forget. The agent doesn't know the terrain it's moving through. A five-screen flow is five separate guesses, and tomorrow it's five more. Nothing accumulates.

The two look like different universes — knowledge vs. action, documents vs. apps, RAG vs. browser agents. They have the same shape. **Every session starts at zero.** The bottleneck in knowledge work was never the reading or the thinking; it was the bookkeeping nobody wants to do. The bottleneck in app automation was never the tapping; it was remembering the terrain so the next tap isn't a guess. Humans abandon wikis for the same reason hand-written automations rot: maintenance grows faster than value.

## Two fixes, one shape

**Karpathy's fix, for knowledge.** The LLM incrementally builds and maintains a persistent wiki — entity pages, concept pages, cross-references — and updates the whole graph each time a new source arrives. One source touches ten or fifteen pages. The wiki is a compounding artifact: the synthesis is already there, the contradictions are already flagged, the answer to your next question is already half-written. You curate sources and ask good questions; the LLM does the bookkeeping. Obsidian is the IDE, the LLM is the programmer, the wiki is the codebase.

**ForrestKim's fix, for action.** The LLM incrementally builds and maintains a persistent map of the app — every screen a node, every action an edge — and updates the graph each time it explores or acts. One exploration pass touches dozens of screens. The route map is a compounding artifact: the agent's next action is no longer a guess, it's a lookup. You pick the app and direct the depth; the LLM does the mapping. The app is the terrain, the LLM is the surveyor, the route map is the agent's map.

Same shape. Same division of labor. Same reason it finally works: **the LLM doesn't get bored, and meaning-based identifiers don't rot.** A wiki entity called `Acme Inc` is still `Acme Inc` next month. A button called `BUTTON:Save` is still `BUTTON:Save` after an A/B test. Both halves of the workspace sit on the same substrate.

## The synthesis

Karpathy gave the brain a compounding artifact. ForrestKim gave the hands the same thing. Put both under one roof and a third thing falls out for free: **context travels between the layers without anyone shipping it.** The moment you start a project about negotiating with a partner company, the LLM already knows who works there (wiki), how to drive the tools you'll use to reach them (app memory), and where the negotiation currently stands (project). No re-explaining, no re-uploading, no re-indexing. Every new move is informed by everything that came before.

This is the move: **a single workspace where brain, eyes, and hands all live, each in its own layer, each with its own lifetime.**

- **Brain** answers *who and what do you know?* It grows over months and years and never really shrinks.
- **Eyes and hands** answer *how does the agent move through the apps you use?* Each app is a unit, alive as long as the app is.
- **Work** answers *what are we doing right now?* Fast, ephemeral, self-archiving.

You rarely write any of it yourself. The LLM writes the wiki pages as you feed it sources. The LLM writes the ROUTES and TRANSITIONS as it explores apps. You sketch a project and the LLM fills in the rest. In practice, I keep a single Claude Code session open with the workspace as its working directory, and the three layers rotate under one conversation: one turn updates an entity page, the next adds a transition to an app's memory, the next edits a plan inside a project and cites both as it goes. One session, three layers, no context switching.

## Architecture

Three layers plus a schema.

**The wiki.** A directory of LLM-generated markdown files covering people, organizations, projects, concepts, intel, and the contexts that bind them. The LLM owns this layer outright — it creates pages, updates them as new sources arrive, maintains cross-references, and keeps the graph consistent. Permanent, slow-growing, read-mostly for humans. The pattern is lifted directly from Karpathy.

**The app memory.** One subdirectory per app, each holding a README, a route map of every screen (`ROUTES.md`), a transition table of `screen + action → next screen` (`TRANSITIONS.md`), and the screenshots that serve as evidence. The LLM creates and updates these during exploration and reads them whenever it needs to act. Permanent but app-scoped; archivable if the app itself dies. The pattern is lifted directly from ForrestKim.

**The projects.** Short-lived workspaces, one per initiative. Each has a README describing its goal and a STATE file tracking where it is. Projects reference the wiki freely for context on people and organizations, and the app memory freely for knowing how to drive the tools they depend on. Any durable knowledge a project surfaces goes back into the wiki — not into the project folder. Projects borrow from the permanent layers; they never duplicate them. When the work is done, the folder moves to `_archive/`.

**The schema.** A `CLAUDE.md` at the root, plus one inside each layer. The root file lays out the three-layer structure and the entry points. Each layer's `CLAUDE.md` covers conventions, link rules, and page formats specific to that layer. This is what turns the LLM into a disciplined maintainer instead of a generic chatbot. You co-evolve it with the model over time.

## What stable identity buys you

Every layer uses meaning-based identifiers. None rely on coordinates, list positions, or indexes.

```
wiki         → entity name        (e.g. "Acme Inc", "Jane Doe")
               = filename = H1 title
app-memory   → TYPE:Label          (e.g. BUTTON:Save, INPUT:Email, TAB:Home)
projects     → phase / task id     (e.g. phase-02, task-02-03)
```

Same state, same ID — regardless of session, screen resolution, UI layout, or time of day. In the wiki you write `[[Acme Inc]]` from any page and never think about paths. In the app memory you plan a five-screen flow as a flat sequence — `tap BUTTON:Confirm`, `tap BUTTON:Continue`, `tap BUTTON:Done` — with no intermediate reads and no guessing. In projects, a task keeps its ID across restarts, so plans survive context resets. This is the substrate the rest of the workspace sits on.

## Three rules the workspace falls apart without

Three additions that aren't in either original, and that the whole thing collapses without.

**Relationships are first-class. Entities never link directly.** When two people are related, both are linked from a *context page* that describes the relationship. "Alice knows Bob" carries almost no information; "Alice and Bob co-lead the strategy unit at Acme and make top-level product decisions together" carries a lot. The context page holds the latter. No entity ever turns into a link-hub that collapses under its own inbound weight. This is a direct extension of Karpathy's graph.

**Pattern and instance never share a file.** Every layer has two kinds of document: `CLAUDE.md` is the pattern — slow-changing, conventions, rules, page formats. `INDEX.md` is the instance catalog — fast-changing, one line per entry, a list of what's actually in the layer right now. Mix them and both rot: the methodology gets buried in churn, and the catalog gets heavy with rules nobody reads.

**Load the unit whole.** Pick a wiki page, an app, or a project, and load *all of it*. For an app that means README + ROUTES + TRANSITIONS together — a few kilobytes of text that make every subsequent action deterministic. Trying to save tokens by drilling in further is false economy: the tokens you avoid come back as bad decisions, and bad decisions are more expensive than tokens.

## Operations

**Ingest.** Drop a source into the raw collection — document, chat export, transcript, screenshot — and hand it to the LLM. The model reads it, extracts facts, updates the wiki pages it touches, flags anything that contradicts existing claims, and appends to the log. One source might touch fifteen pages. I prefer to ingest sources one at a time and stay in the loop, reading each update as it lands. For larger batches — a full Slack export, a year of chat logs — `scripts/ingest.sh` wraps the whole thing in a resumable three-phase scaffold (chunk → parallel extract → consolidate → delta) that makes skipped chunks loud instead of silent.

**Explore.** Point the LLM at an app and ask it to build the route map. The agent reads the accessibility tree (mobile-mcp for Android and iOS, desktop-mcp for macOS), takes a screenshot, records the elements, and walks the app depth-first as a graph. Every new screen earns a numbered screenshot and a row in `ROUTES.md`; every action that changes the screen earns a line in `TRANSITIONS.md`. Depth is a decision you make upfront: **L1** — visit every screen; **L2** — open every dropdown, toggle every switch, exhaust every list; **L3** — execute real tasks (send a message, submit a form, complete a payment) and capture the confirmation, error, and edge-case screens that UI-only exploration never finds. Explore, organize, and analyze stay strictly separated — exploring while analyzing means you see what you want to see, and analyzing on incomplete data means confident but wrong.

**Work.** Start a project and the LLM sets up `projects/<name>/` with a README and a STATE file. Inside, it reads from the wiki freely (for the people and the context) and from the app memory freely (for the tools it needs to drive). Durable knowledge the project surfaces goes back into the wiki — never into the project folder. Projects borrow from the permanent layers; they never duplicate them.

**Lint.** Ask the LLM to health-check the workspace every so often. It looks for wiki pages that contradict each other, entities with no inbound links, apps whose ROUTES and TRANSITIONS have drifted apart, and projects whose STATE hasn't moved in weeks. The wiki layer has a dedicated linter — `scripts/lint.sh` — that enforces its graph rules mechanically. The other layers get linted by the act of reading them.

## Tools

Very little earns its keep. Most of this is just markdown and an LLM agent.

- **`scripts/ingest.sh`** — resumable fact extraction over large sources. Stops chunks from silently dropping.
- **`scripts/lint.sh`** — enforces the wiki's graph rules: context-mediated linking, no orphans, stable titles. `wiki/` only.
- **[mobile-mcp](https://github.com/ForrestKim42/mobile-mcp)** — Android and iOS via the accessibility tree. Single `mobile_do` tool that reads the screen and acts in one call.
- **[desktop-mcp](https://github.com/ForrestKim42/desktop-mcp)** — macOS native apps via the Accessibility API, Electron apps via the Chrome DevTools Protocol. Single `desktop_do` tool, cross-app batching via the `App/TYPE:Label` prefix.
- **`app-memory/schema/METHODOLOGY.md`** — a copy of ForrestKim's original `README.md` kept as the canonical reference for how exploration works.

Tools are scaffolding around an LLM that does the actual judgment. Swap any of them for an equivalent. The pattern is what matters.

## Why this works

*Knowledge's bottleneck was maintenance.* Humans abandon wikis because the bookkeeping grows faster than the value — cross-references to update, contradictions to flag, dozens of pages to hold in your head at once. LLMs don't get bored and can touch fifteen files in a single pass. **The wiki stays alive for the first time, and it compounds.** (That's Karpathy's argument.)

*Action's bottleneck was memory.* Humans and agents alike forget which screens they visited, which buttons they tried, which flows the last UI update broke. Meaning-based identifiers survive layout changes, resolution changes, and A/B tests; an LLM that writes down every screen it sees doesn't forget them next session. **The app map stays alive for the first time, and it compounds.** (That's ForrestKim's argument, lifted.)

Both halves are the same fix: a permanent, meaning-based, compounding artifact that an LLM maintains and a human curates. Put both under one roof and the third thing follows for free — **context travels between the layers without anyone shipping it.** Start a project, and the LLM already knows the people, the tools, and the state. Every move is informed by everything that came before.

The human's job is to curate sources, direct the exploration, decide which projects matter, and think about what it all means. The LLM's job is everything else.

## Memex, finished

Vannevar Bush drew the Memex in 1945 — a personal, curated store with associative trails between documents. Karpathy noted that Bush's unresolved problem was maintenance: who keeps the trails current? The LLM does.

Bush's picture had a second gap, one he didn't even think to ask about. The Memex only ever described what its operator *knew*, never how the operator *moved through the world*. Brain without eyes and hands is half a Memex. ForrestKim's pattern is the other half — and it has the same answer to the maintenance question. The LLM draws the map, updates the trails, and keeps both alive.

This workspace is the two halves under one roof.

## Note

This document is intentionally abstract. It describes the pattern, not a specific implementation. The exact directory layout, file formats, linting rules, MCP tools, and ingestion scripts will all depend on your domain, your apps, and your LLM of choice. Everything above is optional and modular — pick what's useful and ignore the rest. The right way to use it is to share it with your LLM agent and work out a version that fits your life. The document's only job is to communicate the pattern. Your LLM can handle the rest.
