# airwiki — Setup Prompt

A complete, self-contained instruction for building a personal LLM Wiki using Airtable as the substrate, designed to be pasted into Claude Code, Cursor, Claude Desktop, or any LLM agent with Airtable MCP access.

This is the **prompt** you paste. The accompanying README explains what you're building and why.

This repository is template-only. Reading, cloning, or editing this repo should not create an Airtable base or any live Airtable records. Airtable resources are created later, in the end user's own setup session, after they enable Airtable MCP and explicitly ask an agent to run the setup prompt.

---

## How to use this prompt

1. Create a new project directory in your IDE (Cursor, VS Code with Claude Code, etc.) or open Claude Desktop.
2. Make sure the Airtable MCP connector is enabled.
3. Save this file as `CLAUDE.md` (for Claude Code) or `AGENTS.md` (for Codex/OpenCode) at the project root, OR paste the **PROMPT** section below directly into a fresh chat.
4. Tell the agent: *"Read CLAUDE.md and walk me through setup."* (Or just paste the prompt and say "execute.")
5. In that setup session, the agent will ask you a few configuration questions, then build everything in your Airtable account: base creation, schema setup, skill file generation, initial verification.

Estimated time: 30-60 minutes of conversation, mostly the agent doing schema work while you make scope decisions.

---

## Prerequisites

- **Airtable account.** Free tier works for the first ~1,000 records; Team plan ($20/user/month annually) raises the cap to 50,000 plus full-year revision history.
- **Airtable MCP enabled in your agent.** For Claude Desktop: Settings → Connectors → Airtable → connect via OAuth. For Claude Code or Cursor: install the Airtable MCP server per their docs.
- **An LLM agent that can read this prompt and execute multi-step Airtable operations.** Claude Code, Cursor with Claude, Claude Desktop, OpenCode, Codex — any of them work.
- **About 30-60 minutes** of focused attention to walk through setup.

---

# THE PROMPT — paste everything below into your agent

You are helping me build a personal LLM Wiki on Airtable. This is a knowledge management system based on Andrej Karpathy's LLM Wiki pattern (https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), adapted to use Airtable instead of Obsidian for cross-device cloud-native operation.

Read this entire prompt before acting. Then begin the setup walkthrough described in the **Setup Workflow** section at the end.

Important boundary: only execute Airtable setup when this prompt is being used in an end-user setup session with Airtable MCP enabled. If you are reading this file while maintaining or editing the public AirWiki template repository, do not create Airtable resources, do not generate live Airtable IDs, and do not create personal wiki records.

## The pattern in one paragraph

Most LLM-powered knowledge systems work via RAG: documents get chunked, indexed, retrieved at query time, and synthesized fresh for every question. The LLM rediscovers knowledge from scratch on every query. Nothing accumulates. The LLM Wiki pattern is different: the LLM **incrementally builds and maintains** a structured, interlinked collection of "pages" (here, Airtable records with body fields) that sits between you and the raw sources. Each new source doesn't just get stored — it gets decomposed into updates across many existing pages: entity pages, concept pages, source records, syntheses. The wiki is a **persistent, compounding artifact**. Cross-references are already there. Contradictions have already been flagged. The synthesis already reflects everything that's been read. The human curates sources, asks questions, and directs the analysis. The LLM does the bookkeeping that makes a knowledge base actually useful over time — the summarizing, cross-referencing, filing, deduplicating, contradiction-flagging.

## Why Airtable instead of Obsidian

Karpathy's original instantiation uses Obsidian (local markdown files, wiki-links, graph view). That's excellent for single-machine personal use. Trading Obsidian for Airtable trades:

- **Lose:** local-first storage, file-based portability, graph visualization, smooth long-form markdown editing, free git versioning.
- **Gain:** multi-device cloud access (web, mobile, desktop), structured queries with filter views, enforced bidirectional linked records (foreign keys instead of fragile wiki-links), one-click sharing with collaborators (read-only invite for an attorney, advisor, etc.), native MCP integration so any LLM agent can query and update the wiki.

For users who work across multiple devices, want to share parts of the wiki with others, or rely on structured queries more than graph browsing, Airtable wins. For users who prioritize local data ownership and sustained writing in markdown, Obsidian wins. Pick based on your daily reality.

## The architecture: three layers

**Raw sources (immutable).** Files, articles, papers, podcasts, transcripts, screenshots — whatever you read or watch. These live wherever they live (Drive, Dropbox, local disk, web URLs). The Airtable Sources table holds *records about* these sources (URL, summary, metadata) but does not necessarily hold the full content. Source files are never modified.

**The wiki (LLM-maintained).** Multiple Airtable tables holding the structured knowledge: source records, entity pages, concept pages, article records (your own writing), notes (research excerpts and observations), syntheses (refined answers worth keeping), topics (tags), and an activity log. The LLM owns this layer entirely. It creates records, updates them when new sources arrive, maintains cross-references, keeps everything consistent. The user reads it; the LLM writes it.

**The schema (skill).** A SKILL.md file (or CLAUDE.md, AGENTS.md, depending on the agent) telling the LLM how the wiki is structured, what trigger phrases mean, how to ingest sources, how to answer queries, how to lint, what constraints apply. This is the configuration file that makes the LLM a disciplined wiki maintainer rather than a generic chatbot.

## The schema (Airtable base structure)

Create a single Airtable base. Suggested name: **`airwiki`** or **`Personal Wiki`** or **`Knowledge Vault`** (the user picks). The base contains eight tables.

### Table 1: Sources

External material the user reads, watches, or references. The source record holds metadata; the actual content lives wherever it natively lives.

| Field | Type | Notes |
|---|---|---|
| Title | singleLineText | Primary field |
| URL | url | If web-accessible |
| Author | singleLineText | |
| Type | singleSelect | Paper, Blog, Article, Video, Podcast, Book, Spec, Talk, Standard, Report, Newsletter, Tweet, Internal Document, Other |
| Date Published | date | When the source was originally published |
| Date Read | date | When the user consumed it |
| Summary | multilineText | One-paragraph distillation of the source's value |
| Key Quotes | multilineText | Important verbatim excerpts (preserved with source location) |
| Status | singleSelect | Queued, Reading, Ingested, Archive |
| Linked Articles | multipleRecordLinks → Articles | Articles that cite or build on this source |
| Linked Notes | multipleRecordLinks → Notes | |
| Linked Entities | multipleRecordLinks → Entities | Entities this source illuminates |
| Linked Concepts | multipleRecordLinks → Concepts | Concepts this source advances |
| Linked Topics | multipleRecordLinks → Topics | |

### Table 2: Articles

Articles, posts, or longer pieces the *user themself* has written or is drafting. Distinct from Sources (external). Body field holds the full text.

| Field | Type | Notes |
|---|---|---|
| Article ID | singleLineText | ART-NNN, primary |
| Title | singleLineText | |
| Body | richText | Full article content, markdown-flavored |
| Status | singleSelect | Draft, In Review, Submitted, Published, Withdrawn, Archive |
| Venue | singleLineText | Where published or submitted |
| Publication Date | date | |
| URL | url | Link to published version |
| Word Count | number | |
| Notes/Backstory | multilineText | Context — why this piece, version notes, decisions |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Notes | multipleRecordLinks → Notes | |
| Linked Entities | multipleRecordLinks → Entities | |
| Linked Concepts | multipleRecordLinks → Concepts | |
| Linked Topics | multipleRecordLinks → Topics | |

### Table 3: Notes

Research notes, observations, brainstorms, methodology learnings, captured conversations, anything worth remembering that's not big enough to be a Concept or Entity page.

| Field | Type | Notes |
|---|---|---|
| Note ID | singleLineText | NOTE-NNN, primary |
| Title | singleLineText | Short descriptive |
| Body | richText | Full content |
| Type | singleSelect | Research, Brainstorm, Methodology, Observation, Conversation Distillation, Decision Log, Voice Memo Transcript, Reading Note |
| Date Created | date | |
| Source URL | url | If prompted by external content |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Articles | multipleRecordLinks → Articles | |
| Linked Entities | multipleRecordLinks → Entities | |
| Linked Concepts | multipleRecordLinks → Concepts | |
| Linked Topics | multipleRecordLinks → Topics | |

### Table 4: Entities

People, organizations, products, places, events, tools, standards — concrete *things* you track. Each entity has a body that grows over time as you read more sources mentioning them.

| Field | Type | Notes |
|---|---|---|
| Entity ID | singleLineText | ENT-NNN, primary |
| Name | singleLineText | Canonical name |
| Type | singleSelect | Person, Organization, Product, Place, Event, Tool, Standard, Publication, Other |
| Body | richText | The wiki page for this entity — biographical, historical, your evolving picture, your interactions, salient facts |
| Status | singleSelect | Active, Dormant, Historical |
| Aliases | singleLineText | Other names this entity goes by |
| URL | url | Primary external reference |
| Last Substantive Update | date | When the body last got a real revision (not just a link added) |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Concepts | multipleRecordLinks → Concepts | |
| Linked Articles | multipleRecordLinks → Articles | |
| Linked Notes | multipleRecordLinks → Notes | |
| Related Entities | multipleRecordLinks → self | Other entities this one relates to |

### Table 5: Concepts

Abstract ideas, methodologies, frameworks, theories, recurring patterns. Each concept has a body that compounds as you read more.

| Field | Type | Notes |
|---|---|---|
| Concept ID | singleLineText | CON-NNN, primary |
| Name | singleLineText | |
| Body | richText | Current state of your understanding, evolving. Citations to sources for every substantive claim. |
| Status | singleSelect | Active, Mature, Deprecated |
| Last Substantive Update | date | |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Articles | multipleRecordLinks → Articles | |
| Linked Notes | multipleRecordLinks → Notes | |
| Linked Entities | multipleRecordLinks → Entities | |
| Parent Concept | multipleRecordLinks → self | Hierarchical relationships |
| Linked Topics | multipleRecordLinks → Topics | |

### Table 6: Syntheses

Refined answers worth keeping. When the user asks the LLM a substantive question and gets a useful answer, the LLM offers to file it back as a Synthesis page so the exploration compounds.

| Field | Type | Notes |
|---|---|---|
| Synthesis ID | singleLineText | SYN-NNN, primary |
| Title | singleLineText | The question or topic |
| Body | richText | The synthesized answer with inline citations |
| Type | singleSelect | Comparison, Analysis, Recommendation, Decision Rationale, FAQ, How-to, Trade-off Map |
| Date Created | date | |
| Last Substantive Update | date | |
| Status | singleSelect | Current, Superseded, Archive |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Notes | multipleRecordLinks → Notes | |
| Linked Articles | multipleRecordLinks → Articles | |
| Linked Concepts | multipleRecordLinks → Concepts | |
| Linked Entities | multipleRecordLinks → Entities | |

### Table 7: Topics

Lightweight tags. Distinct from Concepts (which have bodies). Topics are organizational — they let you slice the wiki by theme without committing to a full concept page.

| Field | Type | Notes |
|---|---|---|
| Topic Name | singleLineText | Primary |
| Description | multilineText | Optional one-liner about this topic |
| Linked Articles | multipleRecordLinks → Articles | |
| Linked Notes | multipleRecordLinks → Notes | |
| Linked Sources | multipleRecordLinks → Sources | |
| Linked Concepts | multipleRecordLinks → Concepts | |

### Table 8: Activity Log

Append-only record of every operation. Gives the user (and the agent) a timeline of how the wiki evolved.

| Field | Type | Notes |
|---|---|---|
| Event | singleLineText | Primary; format: `YYYY-MM-DD ActionVerb: [brief]` |
| Date | date | |
| Action Type | singleSelect | Source Ingested, Page Created, Page Updated, Synthesis Saved, Lint Pass, Note, Schema Change |
| Affected Records | multilineText | List of record IDs touched |
| Description | multilineText | What happened, what changed |

## Operations the LLM must support

These are the four core workflows that turn a database of records into a living wiki.

### INGEST workflow

Triggered when the user says: *"Ingest this source"* / *"Add this paper to the wiki"* / *"Process [URL or filename]"* / *"Read and file this"* / *"Add a source"*.

Steps the LLM follows:

1. **Read the source.** Fetch the URL, read the file, transcribe the audio — whatever applies. If the source is image-only or paywalled, ask the user to paste content or note the access limitation.
2. **Create a Source record** with metadata: Title, URL, Author, Type, dates, Summary (LLM drafts a 1-paragraph distillation; user can revise).
3. **Discuss key takeaways with the user before writing anything else.** Karpathy's pattern is explicit about this: discussion comes before page updates. The LLM proposes "Here's what I see as the main points and what they affect; should I proceed?" The user confirms, redirects, or amends.
4. **Identify affected pages.** Which existing Entities does this source mention? Which Concepts does it advance, contradict, or refine? Which Articles or Notes does it relate to?
5. **For each affected entity/concept page:** Update the Body field. Add new information. Note where the source contradicts existing claims. Cite the source explicitly.
6. **Create new Entity or Concept pages** if the source introduces things that don't yet have pages and that warrant them (judgment call — discuss with user if uncertain).
7. **Update linked record fields.** The Source record links to every page it touched; each touched page links back to the Source.
8. **Add an Activity Log entry:** `YYYY-MM-DD Source Ingested: [Title] | touched N pages: [list]`
9. **Reply to the user** summarizing what changed across the wiki.

A single source may touch 5-15 wiki pages. That is normal and is the whole point.

### QUERY workflow

Triggered when the user asks a substantive question against the wiki: *"What do I know about X?"* / *"How does X relate to Y?"* / *"Compare A and B based on what I've read"* / *"Show me my view on Z."*

Steps:

1. **Search the wiki.** Use Airtable's `search_records` and `list_records_for_table` across relevant tables. Filter by Topics, Linked Concepts, or full-text search on body fields.
2. **Read the relevant pages.** Pull the bodies from Entities, Concepts, Articles, Notes, Sources, Syntheses that match.
3. **Synthesize the answer with inline citations.** Every substantive claim references the specific record it came from.
4. **At the end of the answer, ask:** "Is this answer worth saving as a Synthesis page so it compounds in the wiki?" If the user says yes, run the SYNTHESIZE-AND-FILE workflow.

If the wiki doesn't contain enough information to answer, say so plainly. Don't fabricate. Suggest sources to ingest that would fill the gap.

### SYNTHESIZE-AND-FILE workflow

Triggered when an answer is worth keeping (either user requests, or LLM offers and user agrees).

Steps:

1. **Create a Synthesis record:**
   - Synthesis ID: SYN-NNN (next available)
   - Title: the question or a clean restatement
   - Body: the synthesized answer with inline citations
   - Type: appropriate single-select value
   - Date Created: today
   - Status: Current
2. **Set linked record fields** to all the pages that contributed to the answer.
3. **Activity Log entry:** `YYYY-MM-DD Synthesis Saved: SYN-NNN [Title]`
4. **Reply:** confirm filed, note Synthesis ID.

This is the operation that stops good explorations from disappearing into chat history.

### LINT workflow

Triggered when the user says: *"Lint the wiki"* / *"Audit the vault"* / *"Health check"* / *"What's stale?"*

The LLM scans the wiki for problems and reports findings as a numbered list with suggested fixes. Categories to check:

1. **Orphan pages.** Entities, Concepts, or Syntheses with no inbound links from other tables.
2. **Stale pages.** Pages with `Last Substantive Update` more than 6 months ago — flag for re-evaluation.
3. **Concepts mentioned but lacking pages.** Scan Article and Note bodies for capitalized terms or repeated phrases that don't yet exist as Concepts. Suggest creating pages.
4. **Sources never decomposed.** Source records with `Status = Ingested` but no Linked Articles / Notes / Entities / Concepts. These got added but never integrated.
5. **Possible contradictions.** Pages where the body claims X while another page claims not-X. (Hard to detect perfectly; the LLM flags candidates for human review.)
6. **Missing citations.** Substantive factual claims in body fields without a linked source. Flag the page and the specific claim.
7. **Broken cross-references.** Linked record fields that point to deleted records (rare but happens after manual cleanup).
8. **Naming inconsistencies.** Two Entities or Concepts with very similar names that might be the same thing — propose merging.

The user reviews findings and approves which fixes to apply. The LLM applies approved fixes and adds an `Activity Log` entry: `YYYY-MM-DD Lint Pass: applied N fixes from M findings`.

Lint is manual — the LLM does NOT auto-lint. Run it monthly or whenever the wiki feels messy.

## The skill / SKILL.md

After the base is set up, the agent generates a SKILL.md (or equivalent) that captures the trigger vocabulary and operational discipline for daily use. The skill is what enables the user to say "ingest this paper" in any future conversation and have the right operations execute automatically.

The SKILL.md must include:

1. **Description** (under 1024 characters for Anthropic skills): trigger phrases, base ID reference, summary of scope.
2. **Connection details:** Airtable workspace ID, base ID, table IDs, field IDs (looked up after creation; field IDs are the most reliable identifiers for write operations).
3. **Schema reference:** every table with field name, field ID, type, notes. Include exact strings for every singleSelect option (Airtable rejects unknown values silently).
4. **ID conventions:** how IDs are formatted (ART-NNN, NOTE-NNN, ENT-NNN, CON-NNN, SYN-NNN). How to compute the next available ID.
5. **Trigger vocabulary:** for each user phrase pattern, the exact Airtable operations to perform (which tables, which fields, in what order).
6. **The four operations** (INGEST, QUERY, SYNTHESIZE-AND-FILE, LINT) spelled out as numbered playbooks.
7. **Disambiguation rules:** how to resolve fuzzy references like "the [Person Name] page" or "the [Concept] concept."
8. **Constraints (non-negotiable rules):** anything domain-specific the user wants encoded — no certain words, no certain topics, citation requirements, voice/tone constraints.
9. **Citation discipline:** every body update must reference its source. Use a consistent format (e.g., `(source: SRC-NNN)` or `[source link]`).
10. **Self-check section:** what the LLM verifies before sending each reply.

The agent generates this file in collaboration with the user during setup. It evolves over time as the user refines what works.

## Setup workflow (what the agent does after reading this prompt)

Execute these steps in order. Pause at decision points.

Before Step 1, confirm the user is asking to set up their own Airtable wiki, not merely editing or reviewing the AirWiki template repository. If the user is maintaining the template repo, stop before any Airtable operation.

### Step 1: Confirm prerequisites and ask scope questions

- Confirm Airtable MCP is connected.
- Ask: *"What should the base be called? Suggested names: `airwiki`, `Personal Wiki`, `Knowledge Vault`, `Second Brain`. You can also pick something specific to your domain."*
- Ask: *"What domains will this wiki cover? Examples: research, professional work, hobbies, personal life. This shapes which Topics and Concepts you might seed."*
- Ask: *"Are there constraints I should bake into the skill? Examples: words to avoid, topics to keep private, voice/tone preferences, citation format."*
- Ask: *"Do you want the agent to create a starter set of Topics, or leave Topics empty and add them as you go?"*

### Step 2: Create the base and tables

Use Airtable MCP `create_base` with all eight tables, all non-link fields. Capture the base ID, table IDs, and field IDs from the response.

### Step 3: Add linked record fields

Linked record fields require both target tables to exist, so add them in a second pass via `create_field`. The full set:

- Sources: Linked Articles, Linked Notes, Linked Entities, Linked Concepts, Linked Topics
- Articles: Linked Sources (auto-created from Sources.Linked Articles), Linked Notes, Linked Entities, Linked Concepts, Linked Topics
- Notes: Linked Sources, Linked Articles, Linked Entities, Linked Concepts, Linked Topics
- Entities: Linked Sources, Linked Concepts, Linked Articles, Linked Notes, Related Entities (self-link)
- Concepts: Linked Sources, Linked Articles, Linked Notes, Linked Entities, Parent Concept (self-link), Linked Topics
- Syntheses: Linked Sources, Linked Notes, Linked Articles, Linked Concepts, Linked Entities
- Topics: Linked Articles, Linked Notes, Linked Sources, Linked Concepts

Each `create_field` call with `type: multipleRecordLinks` auto-creates the inverse link on the target table, so you only need one direction per pair.

### Step 4: Generate the SKILL.md

Produce a SKILL.md file using the template described above. Fill in actual base ID, table IDs, field IDs, and option strings from the created base. Save it in the end user's target project or agent skill directory, such as `/skill/SKILL.md`, unless the user specifies another location.

Do not save a live, user-specific SKILL.md with real Airtable IDs into the public AirWiki template repository. The template repo may contain a placeholder skill template, but generated skills belong to the user's own setup environment.

Validate: description under 1024 characters, all table/field IDs resolved, all single-select option strings exact.

If using Anthropic Claude Code or Claude Desktop with skill installation support, package the skill folder using the skill packaging script:

```bash
python -m scripts.package_skill /path/to/skill-folder
```

Output is a `.skill` file the user installs via Save skill in the Claude UI.

### Step 5: Project knowledge upload (workaround for current Claude.ai behavior)

When a user-defined skill is saved via the Claude.ai UI, the description loads into the agent's `<available_skills>` block at conversation start, but the SKILL.md body content does NOT get mounted at a readable filesystem path. This means triggers fire correctly but the agent cannot read the schema, field IDs, or playbooks.

Workaround: upload the same SKILL.md as a regular project knowledge file. It appears in `<project_files>` and the agent reads it via `view` when the skill triggers.

This is a belt-and-suspenders setup. The skill description handles triggering; the project knowledge file handles operational content. Same source file, two install locations. Treat it as standard practice until the platform mounts user skill bodies natively.

For Cursor or Claude Code in an IDE, the SKILL.md (or equivalent) lives in the project directory and is read directly — this workaround isn't needed.

### Step 6: Seed initial content (optional)

Ask the user: *"Want to seed the wiki with anything to start? Examples: import an existing reading list, paste a research note you've been meaning to file, add the first Concept page for a topic you've been thinking about. Or skip and add things organically."*

If the user wants to seed: do a small INGEST or page creation as a live test of the operations. This validates the schema is right and the skill triggers work.

### Step 7: Test the trigger vocabulary

Run a few test commands to confirm the skill works:

- *"Add a source: [paste a URL or paste content]"* → SOURCE record created.
- *"Add a note about [topic]"* → NOTE record created.
- *"Pull up everything I have about [topic]"* → query returns results.
- *"Show me the wiki index"* → agent generates a markdown summary of all tables and counts.

If any test fails, iterate on the SKILL.md.

### Step 8: Document and finish

Generate or update a README.md in the user's setup project describing what was built. The README should be shareable (no private data baked in). If the user's setup project is version-controlled, it can contain the generated SKILL.md and README.md, but not Airtable exports or private source contents unless the user explicitly chooses to include them.

Activity Log: `YYYY-MM-DD Schema Change: Wiki base created with 8 tables, schema v1.0`.

## Decisions to ask the user (consolidated)

Before executing, the agent confirms:

1. Base name?
2. Workspace (if multiple Airtable workspaces are accessible)?
3. Domain scope — what kinds of content will this wiki hold?
4. Domain constraints — words to avoid, topics to flag, citation format preferences?
5. Starter Topics (if any)?
6. Free tier vs. Team plan?
7. Multi-base or single base architecture? (Default: single base. If the user has multiple distinct domains they want strictly separated, suggest one base per domain.)
8. Naming convention for IDs (default: ART-NNN, NOTE-NNN, ENT-NNN, etc.)?
9. Frequency of LINT passes (default: manual only, on demand)?

## Constraints the LLM must respect during setup

- Do not modify or delete any existing Airtable bases without explicit user approval.
- Confirm before overwriting fields or changing schema.
- Do not auto-ingest any content during setup unless the user explicitly seeds.
- All ID generation must be conflict-free (query the table, find the highest existing prefix-NNN, increment).
- All `create_records_for_table` and `update_records_for_table` calls require Claude.ai approval. Warn the user upfront when a sequence of approvals is coming.
- Citation discipline applies from the moment the wiki has its first page. Every body update references its source.

## After setup: ongoing usage

The wiki grows by daily use, not by big batch loads. Typical patterns:

- Read an article → "ingest this" (5 minutes including discussion + LLM updates).
- Have a useful conversation with the LLM → "save this as a Synthesis."
- Capture a thought → "add a note about X."
- Identify a recurring theme → "promote [X] to a Concept page."
- Look up something → "what do I know about Y?"
- Periodic health check → "lint the wiki."

The wiki compounds slowly. After a month, you'll have ~50-100 records and the cross-references start showing patterns. After six months, ~500-1,000 records and the wiki becomes a real second brain. After two years, it's the primary place you go to think about your domain.

## Honest limits

This pattern works well for single-user, slow-moving, human-curated knowledge bases at the scale of a few hundred to a few thousand records. It does NOT work for:

- High-volume, multi-user, real-time knowledge bases (use a proper RAG system).
- Domains where exact original wording matters (legal, regulatory, scientific) — wiki summaries are lossy compression. Always link back to sources.
- Cases requiring strong access control (Airtable's permissions are coarse).

The wiki is also vulnerable to vendor lock-in. Periodic CSV exports as backups are recommended. If Airtable disappears or pricing becomes prohibitive, you'd need to export and rebuild the structure elsewhere — possible but not free.

---

# END OF PROMPT

That's everything the agent needs. Paste from "You are helping me build a personal LLM Wiki on Airtable" through "EXACT ORIGINAL WORDING MATTERS" into your fresh chat with Claude Code, Cursor, or Claude Desktop and the agent will walk you through setup.

The accompanying README.md (separate file) describes what gets built. Use that to share the architecture with others or refer back to as the wiki grows.
