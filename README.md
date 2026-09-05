# skills

General-purpose skills I use to get things done on my computer.

## Install

```sh
npx skills add bholmesdev/skills
```

### Claude Code

Add this repo as a plugin marketplace, then install the plugin:

```sh
/plugin marketplace add bholmesdev/skills
/plugin install skills@bholmesdev-skills
```

### Codex

This repo ships a `.codex-plugin/plugin.json` pointing at `skills/`, so it works as a Codex plugin as-is. See [Codex's plugin docs](https://developers.openai.com/codex/skills) for how to add a local or remote plugin source.

### Cursor

Each skill also ships as a [Cursor rule](https://cursor.com/docs/context/rules) under `.cursor/rules/`. Copy the ones you want into your own project's `.cursor/rules/`, or clone this repo and open it directly so Cursor picks them up.

## Contributing

Contributions are welcome! Feel free to open an issue or submit a PR.
