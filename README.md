# AI Project Vault

**One vault per codebase that works as the project's wiki, its living spec, and your AI coding agent's memory.**

`ai-project-vault.md` is a build file. Give it to your coding agent (Claude Code, Codex, or any agent that reads `AGENTS.md`) inside a project, and the agent sets up a `vault/` folder next to your code:

- **A wiki you can browse in Obsidian.** It holds specs, implementation notes, conventions, and decisions, all interlinked and kept current by the agent.
- **Memory for the agent.** Every session starts from the wiki's index, so the agent picks up where the last one (or a teammate's agent) left off.
- **An inbox for rough input.** You drop idea notes and change requests in, and they come out as finished specs, or as working code with updated docs.

## Why

Coding agents forget everything between sessions. Their built-in memory is private to one person and one tool, and it drifts away from the project's docs. Meanwhile specs go stale because nobody wants to update them after the code changes.

This project combines two ideas to fix both:

- From [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): an LLM incrementally builds and maintains a persistent, interlinked markdown wiki from raw sources, with an index, a log, and ingest, query, and lint operations. Knowledge is compiled once and kept current instead of being rediscovered on every question.
- From [Jared Rhodenizer's AI Memory Vault](https://github.com/jaredrhod/ai-memory-vault): the vault is the agent's external memory. A short boot file loads every session, a manual defines the rules, indexes stay in sync, and the agent's native memory is redirected into the vault so there is only one memory.

What is different here:

| | LLM Wiki | AI Memory Vault | AI Project Vault |
|---|---|---|---|
| Scope | a personal knowledge base | a person's life and work | **one software project** |
| Lives in | its own folder | the home folder | **`vault/` inside the repo, committed with the code** |
| Main content | summaries and concepts | profile, projects, daily notes | **feature specs, implementation notes, conventions, decisions** |
| Typical loop | ingest a source | have conversations | **idea → spec**, and **change request → code → updated spec** |
| Shared with | you | you | **the whole team and every agent** |

## What gets installed

```
<project>/
├── AGENTS.md                 short boot block: read the wiki first, rules that never lapse
├── CLAUDE.md                 @AGENTS.md
└── vault/                    open this folder in Obsidian
    ├── schema.md             operating manual: layout, conventions, workflows, templates
    ├── raw/
    │   ├── inbox/            your rough input, waiting to be processed
    │   └── archive/YYYY-MM/  processed input, unchanged
    └── wiki/                 the project's memory
        ├── index.md          the map, read at the start of every session
        ├── log.md            recent activity (older months rotate into log/YYYY-MM.md)
        ├── overview.md
        ├── code/             coding style, dev setup, build, testing, architecture
        ├── design/           planning material, idea reviews, cross-feature research
        ├── ui/               design system, shared UI guidelines
        ├── decision/         decision records
        └── feature/
            └── <feature-name>/
                ├── index.md
                ├── spec.md            what and why
                └── implementation.md  how it is built
```

Everything is plain markdown, and nothing gets installed on your machine. All changes stay inside the repository, except the optional native-memory redirect described below, which the agent writes only if you approve it.

## Install

Open your agent in the project root and paste:

> Read https://raw.githubusercontent.com/coolerx/ai-project-vault/main/ai-project-vault.md and set it up in this project.

The agent will:

1. Check the project and ask a few questions: the project name, the wiki language, and whether this is a new or existing codebase.
2. For an existing codebase, scan the README, docs, build files, and source tree, then propose an overview, `code/` pages, and a feature list for you to edit.
3. Show you every file it will create, and wait for your yes.
4. Build the vault, the boot block, and the schema.
5. Offer to migrate the agent's existing native memory for this project into the wiki. It copies and never deletes, and it redirects future memory to the vault only if you agree.
6. Verify links, naming, and frontmatter, then show you how to use it.

**Wiki language:** page content is written in the language you choose. Folder names, file names, and frontmatter are always English.

## Connect Obsidian

This part is up to you. In Obsidian, choose **Open folder as vault** and select `<project>/vault`.

Recommended settings (Settings → Files and links), so links you add by hand match the vault's conventions:

- **Use [[Wikilinks]]**: off
- **New link format**: Relative path to file
- **Automatically update internal links**: on

Optional: set **Default location for new attachments** to *Same folder as current file*. Images you paste into an inbox note then stay next to it and get archived with it.

The vault uses standard relative markdown links, so it also reads fine on GitHub and in your editor.

## Daily use

You talk to the agent in plain language. There are no commands to learn.

### Idea → spec

Write a rough note, for example `vault/raw/inbox/coupons.md`:

> customers should be able to enter a coupon at checkout. one per order? percent and fixed amount. expires.

Then say:

> Turn raw/inbox/coupons.md into a spec.

The agent reads the note and the related wiki pages, tells you what it understood, and lists open questions with proposed defaults. Once you've discussed them, it writes `wiki/feature/checkout/spec.md` (or creates a new feature). It records any real decision in `wiki/decision/`, moves your note to `raw/archive/2026-09/2026-09-15-checkout-coupons.md`, and updates the indexes and the log.

If the note should end as research rather than a spec:

> Organize raw/inbox/payment-vendors.md as research.

### Change request → code → wiki

Drop a rough change into the inbox, for example `vault/raw/inbox/coupon-limit.md`:

> allow stacking two coupons, but never more than 50% off total

Then say:

> Implement raw/inbox/coupon-limit.md.

The agent reads the note plus the feature's spec, implementation notes, and decisions. It shows you a plan covering the change against the current spec, the affected code, the steps, the tests, and any conflicts. After you confirm, it implements and tests the change, updates `spec.md` and `implementation.md` to match what was actually built, archives the note, and logs it. The same flow works when you ask for a change directly in chat.

### Everything else

- **Ask about the project.** *"How are coupons validated?"* The agent answers from the wiki with links, and can file a useful answer as a new page.
- **Lint.** *"Lint the wiki."* The agent checks for drift between wiki and code, broken links, missing index entries, naming problems, stale drafts, and waiting inbox items.
- **Memory while coding.** When the agent learns something a future session needs (a build gotcha, an agreed convention, a clarified behavior), it writes it into the right page right away.

## Conventions at a glance

- **Names:** English, lowercase, `-` only. Folders are singular.
- **Links:** relative markdown links, never `[[wikilinks]]`. Code is referenced by repo-root path.
- **Frontmatter:** `type`, `status`, `updated`, plus `feature`, `sources`, and `code` where relevant.
- **Spec status:** `draft` → `approved` → `implemented` → `deprecated`.
- **Raw files** are never edited. They move from `raw/inbox/` to `raw/archive/YYYY-MM/` and are renamed with a date prefix, keeping their original extension (`coupons.md` → `raw/archive/2026-09/2026-09-15-checkout-coupons.md`).
- **Log:** each entry is a title line plus at most 3 body lines. When `log.md` passes 200 lines, previous months rotate into `wiki/log/YYYY-MM.md`.
- **Truth:** code is the truth for current behavior, and the spec is the truth for intent. When they disagree, the agent asks you instead of silently changing either one.

`vault/schema.md` has the full rules. It is meant to evolve with your project, and the agent changes it only with your approval.

## Upgrading

When a new version of `ai-project-vault.md` is released, run it again in the project. The agent detects the existing `vault/schema.md` and switches to upgrade mode. It shows you a diff of the schema, keeps your project-specific rules, updates only the marked block in `AGENTS.md`, and leaves your wiki content alone.

## FAQ

**Should the vault be committed?**
Yes. The vault is part of the project, so specs and code are versioned together and the whole team shares one memory. Only Obsidian's per-user workspace files are ignored.

**Which agents does it work with?**
It works with any agent that reads `AGENTS.md`, such as Codex. Claude Code reads it through the `@AGENTS.md` import in `CLAUDE.md`. For a tool that uses a different instruction file, point that file at `AGENTS.md`.

**What happens to my agent's built-in memory?**
During install, the agent offers to copy this project's native memory into the vault, fold the project knowledge into the wiki, and replace the native memory with a short redirect. Personal items that don't belong in a shared repo are left for you to decide on. Originals are never deleted by the agent. Teammates don't need this step, because the boot block tells every agent to keep project knowledge in the wiki.

**Does the agent read the whole vault every session?**
No. It reads `wiki/index.md` and the last few log entries, then only the pages the current task needs.

**Can I edit wiki pages by hand?**
Yes. The agent will keep indexes and links consistent the next time it touches that area, and "lint the wiki" catches anything missed.

**Is Obsidian required?**
No. Agents only need the files. Obsidian is the most comfortable way for humans to browse and follow the links.

## Credits

- [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) by Andrej Karpathy: the wiki pattern (raw sources, a maintained wiki, a schema, ingest/query/lint, index and log).
- [AI Memory Vault](https://github.com/jaredrhod/ai-memory-vault) by Jared Rhodenizer: the vault-as-agent-memory approach (boot config, operating manual, index discipline, native memory redirect).

This project is an independent write-up inspired by those ideas and does not reuse their text.

## License

[MIT](LICENSE)
