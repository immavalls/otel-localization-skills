---
name: otel-es-translation
description: >-
  Translate an opentelemetry.io English page to Spanish — creates a branch,
  copies the source under `content/es/`, adds the `default_lang_commit` front
  matter pin and stable heading anchors, translates with glossary-aware
  terminology, runs the repo's lint/format/spelling/link checks, commits with
  DCO sign-off, and pushes. Does not run the site locally — review happens on
  the translated text and diff. Use when the user wants to localize a single
  page under `content/en/` to `content/es/`, or invokes
  `/otel-localization:otel-es-translation`.
argument-hint: "[source path under content/en/]"
allowed-tools: Read Edit Write Grep Glob Bash AskUserQuestion TaskStop
effort: medium
---

# OpenTelemetry English → Spanish translation

End-to-end workflow for localizing a single English page from `content/en/...`
to `content/es/...` in the `opentelemetry.io` repo. The repo's npm scripts do
the mechanical checks; this skill sequences them around the human judgment
steps (terminology and review). Kept deliberately simple: one page, one PR,
no local Hugo server.

Always reference the End User SIG localization tracking issue
<https://github.com/open-telemetry/opentelemetry.io/issues/10252> in the PR
description.

## Arguments {#arguments}

The user may pass the source path as an argument. If absent, **ask the user**
for the path with `AskUserQuestion` — do not guess. The path must be relative
to repo root and start with `content/en/` (e.g. `content/en/docs/guidance/_index.md`).

## Working-directory rule {#cwd-rule}

The Bash tool's working directory **persists between calls**. If a step needs
to `cd` somewhere else, wrap it in a subshell: `(cd dir && cmd)`. Otherwise
later `npm run …` calls will run against the wrong project.

## Workflow {#workflow}

### 1. Confirm starting state

Confirm `git status` is clean and `git branch --show-current` is `main` (or
warn and ask whether to proceed). Note the source path the user provided.

### 2. Create the translation branch

Derive a kebab-case slug from the source path (drop `content/en/`, strip
`.md`/`_index`, collapse separators). Examples:

- `content/en/docs/guidance/_index.md` → `guidance-index`
- `content/en/blog/2026/some-post.md` → `blog-2026-some-post`

```sh
git checkout -b translate-es-<slug>
```

### 3. Copy the source file

Target path = source path with `content/en/` → `content/es/`. Create parent
directories as needed. If the target already exists, ask the user whether to
overwrite, resume, or abort.

```sh
mkdir -p <target-dir>
cp <source> <target>
```

Copy only the `.md` file — see [Gotchas](#gotchas) about page-bundle images.

### 4. Pin `default_lang_commit` in the target's front matter

Look up the latest commit that touched the source file:

```sh
git log -1 --format=%H -- <source>
```

Add a line to the target's YAML front matter (after `weight:` or before the
closing `---`):

```yaml
default_lang_commit: <hash>
```

This is how the repo's i18n-drift check detects when a translation falls
behind the source.

### 5. Add stable anchors to every heading

For each `##`, `###`, etc. heading in the target file, append
`{#anchor-slug}` where the slug matches the **English** heading's Hugo
auto-id. Example:

```markdown
## How to contribute → ## How to contribute {#how-to-contribute}
```

This keeps existing English fragment URLs
(`https://opentelemetry.io/docs/.../#how-to-contribute`) working after the
heading text is translated.

### 6. Translate prose

Translate only natural-language prose. **Do not translate**:

- Code blocks, code spans, inline code
- Image paths, link URLs, link anchor IDs
- GitHub team/repo names (e.g. `End User SIG`, `Communication SIG`)
- Proper nouns (OpenTelemetry, Hugo, Netlify, etc.)
- Front matter keys

Check [`resources/es/glossary.csv`](../../resources/es/glossary.csv) first —
it's a local mirror of the official OTel-ES glossary spreadsheet (see
[`resources/README.md`](../../resources/README.md) for provenance) and reads
as a plain file, no network access needed. For a term not listed there, fall
back to terminology already established under `content/es/` (grep there for
prior choices) and ask the user when a term is still ambiguous.

Established conventions in this repo:

- `OpenTelemetry`, `SIG` → unchanged
- `telemetría`, `observabilidad`
- `backend`, `pipeline`, `issue` → kept as loanwords (already in use)
- `adopters` → `organizaciones` (**not** `adoptantes` — explicitly rejected)
- `living documents` → `documentos en constante evolución` (not `documentos vivos`)
- `Blueprints` → kept (proper noun, matches the End User SIG document type)
- Use Spanish angle quotes `«...»` instead of straight `"..."`

Surface any non-obvious terminology calls back to the user in chat. Do not
silently invent unfamiliar terms.

Once done, wait here for the user to review the translated text and diff
directly — no local site preview is needed.

### 7. Quality checks (in order)

Run from repo root, sequentially. Stop and surface output if any fails:

```sh
npm run check:text
npm run check:text -- --fix
npm run check:markdown
npm run check:spelling
npm run fix:dict
npm run fix:filenames
npm run fix:format
```

### 8. Stage and commit (DCO sign-off required)

Stage **only** the translated file — nothing else. CNCF requires `-s` for the
DCO sign-off trailer.

```sh
git add <target>
git commit -s -m "[es] Localize <target-relative-path>"
```

### 9. Push

```sh
git push -u origin translate-es-<slug>
```

GitHub returns a "Create a pull request" URL. Surface it to the user, with a
reminder:

> Reference issue
> <https://github.com/open-telemetry/opentelemetry.io/issues/10252> in the PR
> description. Title should follow the pattern `[es] Localize <path>`.

### 10. Final summary

Post a concise summary to the user covering:

- The branch name and target file path
- The PR URL (from step 9)
- Notable terminology decisions made during step 6 (especially any that
  deviated from the glossary, were inferred from prior `content/es/` usage,
  or were invented because no precedent existed)
- Any heading anchors that required judgment (e.g. English heading slugs
  that differ from a literal slug of the original text)
- Any quality-check warnings that were suppressed or accepted as
  pre-existing (step 7)
- Open questions worth flagging to the expert reviewer — phrases you were
  unsure about, ambiguous source wording, or terms where the user's input
  would improve future translations

Keep it short and scannable. The goal is to give the expert user enough
context to spot-check the translation and answer remaining terminology
questions before the PR is merged.

## Gotchas {#gotchas}

- **Don't copy images — translate the `.md` file only.** If the source page
  is a Hugo page bundle (`_index.md` plus local images/diagrams in the same
  folder), copy and translate only the Markdown file into `content/es/...`.
  Leave image files where they are; the Spanish page keeps referencing the
  same English-tree image paths instead of duplicating binary assets.
- **This skill does not run the site locally.** No `hugo serve`/`npm run
serve` step — review happens on the translated text and the diff, not a
  rendered page. Keeps the workflow simple and avoids depending on a working
  local Hugo toolchain.
- **`cd` persists in Bash tool calls.** Always use `(cd dir && cmd)`
  subshells.
- **The `.htmltest.yml` config is generated, not committed.** If
  `check:links` fails with "default config `.htmltest.yml` does not exist",
  run `npm run generate:config:links` first.
- **Don't stage anything but the translated file.** Repo-setup steps
  (`npm install`, submodule init) may touch other paths; verify the commit
  contains exactly one file.
