# otel-localization-skills

A [Claude Code](https://code.claude.com) plugin with Agent Skills for
translating and maintaining localized documentation on
[opentelemetry.io](https://github.com/open-telemetry/opentelemetry.io).

It starts narrow — one skill, one language — and is meant to grow into a
shared base other locales can reuse. See [Roadmap](#roadmap) for what's
deliberately not built yet.

## Skills

### `otel-es-translation`

Translates a single English page (`content/en/...`) to Spanish
(`content/es/...`): creates a branch, copies and pins the source commit
(`default_lang_commit`), adds stable heading anchors, translates prose with
glossary-aware terminology, runs the repo's lint/format/spelling checks,
commits with DCO sign-off, and pushes — ready for a PR.

It's intentionally simple: one page per run, no local Hugo server. Review
happens on the translated text and diff, not a rendered preview.

See [`skills/otel-es-translation/SKILL.md`](skills/otel-es-translation/SKILL.md)
for the full workflow and terminology conventions.

## Resources

Reference data skills read directly instead of hitting a live Google
Sheet — one folder per target language, starting with
[`resources/es/`](resources/es/glossary.csv). See
[`resources/README.md`](resources/README.md) for how it's structured and
kept in sync with the source spreadsheet.

## Installing

This plugin isn't published to a marketplace yet. To try it locally:

```sh
claude --plugin-dir /path/to/otel-localization-skills
```

Or drop a copy/symlink of this repo under `~/.claude/plugins/` to have it
load automatically in every session. Once installed, the skill is invoked as
`/otel-localization:otel-es-translation`, or by describing the task in
chat (e.g. "translate this page to Spanish").

## Roadmap

Tracked as GitHub issues rather than built up front, to keep each skill
small and reviewable:

- **`otel-es-translation-review`** — reviews an open ES translation PR
  (anchors, drift, terminology, link integrity) without editing anything.
- **Drift-fix skill** — `default_lang_commit` drift is already detected by
  the main repo's i18n check; a future skill would fix a drifted page
  end-to-end (diff old vs. new English, update the existing translation,
  refresh the pin).
- **Refactor for other locales** — `otel-es-translation` is Spanish-specific
  today (glossary path, established-conventions list, PR title pattern all
  hardcoded to `es`). Once the workflow has proven itself, factor out the
  language-agnostic parts (branch/commit/push mechanics, anchor pinning,
  drift detection) into a common base, parameterized by language, so a new
  locale skill is a thin wrapper plus its own `resources/<lang>/glossary.csv`
  — the `resources/` layout is already set up for this.
- **Evals** — automated checks that a translation run actually followed the
  skill (terminology conventions honored, anchors correct, no stray files
  staged).

## License

Apache License 2.0 — see [LICENSE](LICENSE).
