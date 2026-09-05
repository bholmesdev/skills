# Packaging skills as plugins for Claude Code, Codex, and Cursor

Reference rules for turning a directory of `SKILL.md` files into an installable plugin for all three major coding agents, with the reasoning and sources behind each rule. Written so an agent reading this file can apply the rules directly without re-fetching docs or re-deriving the format.

Derived while packaging `bholmesdev/skills` as a plugin (PR: https://github.com/bholmesdev/skills/pull/1, branch `add-plugin-manifests` on fork `UkrainianCitizen/bholmesdev-skills-as-plugin`). Confirmed 2026-09-05.

## Baseline: the shared `SKILL.md` format

All three tools consume the same underlying primitive: a directory per skill containing a `SKILL.md` file with YAML frontmatter (`name`, `description`) followed by markdown instructions. This is the "open agent skills standard" referenced in Codex's docs. Do not rewrite `SKILL.md` content when adding plugin manifests — the manifests are additive wrappers, not replacements.

Evidence: `bholmesdev/skills/skills/*/SKILL.md` already used this format before any plugin work; Codex's own skill docs describe the identical shape (see below).

## Rule 1 — Claude Code: `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`

**Files:**
```
repo-root/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

**`plugin.json` fields observed:** `name`, `description`, `version`, `author` ({`name`, `url`}), `homepage`, `repository`, `license`, `keywords` (array), `skills` (path to skills dir, e.g. `"./skills/"` or an array of explicit skill paths).

**`marketplace.json` fields observed:** top-level `name`, `id`, `owner` ({`name`}), `metadata` ({`description`, `version`}), and a `plugins` array where each entry has `name`, `source` (relative path, e.g. `"./"`), `description`, `version`, `author`, `keywords`, `category`.

**Source of truth — do not treat as inferred, these are live installed plugins on this machine, verifiable by reading the files directly:**
- `~/.claude/plugins/marketplaces/karpathy-skills/.claude-plugin/plugin.json` and `marketplace.json`
- `~/.claude/plugins/marketplaces/diagram-design/.claude-plugin/plugin.json` and `marketplace.json`
- `~/.claude/plugins/marketplaces/humanizer/.claude-plugin/`
- `~/.claude/plugins/marketplaces/claude-plugins-official/.claude-plugin/`

These are functioning plugins actively installed and loaded by this Claude Code instance — their manifests are proof the schema is accepted at runtime, not just documented somewhere. Re-reading them costs one `Read` call and settles any doubt about field names or nesting.

**Reasoning:** `plugin.json` describes one installable unit (what `/plugin install` pulls in); `marketplace.json` describes a listing of one-or-more plugins (what `/plugin marketplace add` reads). A single-plugin repo needs both: `marketplace.json` so the repo itself can be added as a marketplace source, `plugin.json` so that marketplace has something to point at ("./" as `source`).

## Rule 2 — Codex: `.codex-plugin/plugin.json`

**Files:**
```
repo-root/
├── .codex-plugin/
│   └── plugin.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

**Required fields, quoted directly from Codex's own documentation** (fetched 2026-09-05 from `https://developers.openai.com/codex/skills`, which 308-redirects to `https://learn.chatgpt.com/docs/build-skills`, and from `https://developers.openai.com/codex/build-plugins` → `https://learn.chatgpt.com/docs/build-plugins`):

> "A skill is a directory with a `SKILL.md` file plus optional scripts and references. The `SKILL.md` file must include `name` and `description`."

> Skill directory shape: `my-skill/{SKILL.md (required), scripts/ (optional), references/ (optional), assets/ (optional), agents/openai.yaml (optional)}`. Local skills for a single repo live in `.agents/skills` at the current directory, a parent, or the repo root — this is the ad hoc per-repo convention, separate from the plugin-bundling convention below.

> On bundling: "Plugins can include one or more skills. They can also optionally bundle registered MCP server connections, bundled MCP server configuration, and presentation assets in a single package." And: "When you want to distribute a reusable skill beyond a single repo, or optionally bundle them with connectors, use plugins" — "prefer plugins" for "reusable distribution of your own skills."

> Manifest location and minimal shape, quoted from `build-plugins`: manifest must be **`.codex-plugin/plugin.json`** at plugin root, with required fields `name` (kebab-case), `version` (semver), `description`, and `skills` (path to skills dir relative to the manifest, e.g. `"./skills/"`). Example tree given in the docs:
> ```
> plugin-name/
> ├── .codex-plugin/
> │   └── plugin.json
> └── skills/
>     └── skill-name/
>         └── SKILL.md
> ```

**No `marketplace.json` equivalent is documented for Codex.** The docs mention a "local marketplace" for testing via a `@plugin-creator` skill, but do not specify its schema — do not invent one.

**Optional `interface` block** (seen in `~/.claude/plugins/marketplaces/diagram-design/.codex-plugin/plugin.json`, a working reference, not just docs): `displayName`, `shortDescription`, `longDescription`, `developerName`, `category`, `capabilities` (array, e.g. `["Read", "Write"]`), `websiteURL`, `defaultPrompt` (array of example prompts), `brandColor`. This block configures the ChatGPT desktop app's presentation of the plugin; it is optional and additive, safe to include or omit.

**Reasoning:** Codex's plugin manifest is deliberately close to Claude's `plugin.json` (same core fields: name/version/description/skills), which is why the same `skills/` directory serves both tools without duplication — only the manifest wrapper differs, plus its directory name (`.codex-plugin` vs `.claude-plugin`).

## Rule 3 — Cursor: one `.cursor/rules/<skill-name>.mdc` per skill

**Files:**
```
repo-root/
└── .cursor/
    └── rules/
        ├── skill-one.mdc
        ├── skill-two.mdc
        └── ...
```

**Why one file per skill, not one combined file — quoted directly from Cursor's docs** (fetched 2026-09-05 from `https://cursor.com/docs/context/rules`):

> "Keep rules under 500 lines" and "Split large rules into multiple, composable rules."
> Effective rules should be "focused, actionable, and scoped."
> Example structure shown in the docs is per-topic, not monolithic:
> ```
> .cursor/rules/
>   react-patterns.mdc
>   frontend/
>     components.mdc
> ```

**Rule format:** YAML frontmatter + markdown body, same as `SKILL.md`. Cursor supports three scoping mechanisms, chosen by which frontmatter fields are present:
1. `globs: <pattern>` → Auto Attached (loads when a matching file is open/edited).
2. `description: <text>`, no `globs`, no `alwaysApply` → Agent Requested (the agent reads the description and decides whether to pull in the rule; this is the correct mode for skills that are triggered by task/intent rather than by file type).
3. `alwaysApply: true` → Always (loaded into every request; reserve for genuinely universal guidelines, e.g. `karpathy-skills` uses this for its four behavioral principles that should apply to all coding work).

**For a bundle of distinct, task-triggered skills (the common case), use mode 2 per skill**, copying each skill's own `description` from its `SKILL.md` frontmatter verbatim into the rule's `description` field — this preserves the same trigger condition Claude Code and Codex already use to decide when to invoke the skill, so behavior stays consistent across all three tools without maintaining three separate descriptions.

**Reasoning against a single combined file:** a bundle of unrelated skills (e.g. alt-text, disk cleanup, code simplification, transcription) has no shared trigger condition. One combined `.mdc` would either force `alwaysApply: true` (injecting all six skills' instructions into every single request, most of them irrelevant) or use one generic `description` that can't discriminate between six unrelated intents. Splitting by skill lets Cursor's agent match the one relevant rule per request, exactly mirroring how Claude Code and Codex already dispatch skills independently by their individual descriptions.

**Live reference confirming this pattern predates this exercise:** `~/.claude/plugins/marketplaces/karpathy-skills/.cursor/rules/karpathy-guidelines.mdc` — a single rule file, because that plugin ships exactly one behavioral-guideline skill; the one-file-per-skill rule scales down to one file when there is only one skill, it does not contradict that case.

## Applying all three together

For a repo with an existing `skills/<name>/SKILL.md` structure, adding plugin support for all three tools requires **no changes to the existing skill content** — only additive files:

1. `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` → `skills` field points at `"./skills/"`.
2. `.codex-plugin/plugin.json` → same `skills` field, `"./skills/"`, pointing at the identical directory.
3. `.cursor/rules/<name>.mdc` for each skill → body is the skill's existing markdown instructions, frontmatter is `description:` copied from the skill's own frontmatter.

This is why the same source skill can serve three plugin ecosystems from one canonical copy of the instructions (in `SKILL.md`) plus one mirrored copy per skill for Cursor (which cannot read `SKILL.md` directly and needs its own `.mdc` file).

## Sources (fetched and quoted 2026-09-05, do not need re-verification unless the vendor changes docs)

- Cursor rules: https://cursor.com/docs/context/rules
- Codex skills: https://developers.openai.com/codex/skills (→ https://learn.chatgpt.com/docs/build-skills)
- Codex plugins: https://developers.openai.com/codex/build-plugins (→ https://learn.chatgpt.com/docs/build-plugins)
- Claude Code plugin schema: no single public doc page was fetched for this; verified instead against multiple live, currently-installed, functioning plugins on this machine (see file paths under Rule 1) — this is direct runtime evidence, stronger than documentation that could be stale.
