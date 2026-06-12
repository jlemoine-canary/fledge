# Changelog

All notable changes to the fledge plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2026-06-12

### Added

- Forbid tests that call private functions directly — new anti-pattern in
  `skills/fledge-test/SKILL.md` and item C1 in
  `references/code-review-checklist.md`
  ([#1](https://github.com/jlemoine-canary/fledge/pull/1)).

### Removed

- Per-skill `version:` frontmatter in `skills/*/SKILL.md` — the plugin version
  in `.claude-plugin/plugin.json` is the single release unit.

## [0.1.0] - 2026-06-08

### Added

- Initial plugin: lifecycle skills (`fledge`, `fledge-auth`, `fledge-ingest`,
  `fledge-plan`, `fledge-review`, `fledge-test`, `fledge-implement`,
  `fledge-qa`), subagent personas, shared references and templates.
- `marketplace.json` so the repo is installable as a local directory
  marketplace.
