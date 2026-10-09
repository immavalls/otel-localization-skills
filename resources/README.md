# resources

Reference data that translation/review skills read directly, instead of
fetching a [live Google Sheet](https://docs.google.com/spreadsheets/d/1Nh0RNGuHjfPB3aDdq4_AvfdFJ0zFvyzg4Rzdu6uR3QA/edit?gid=0#gid=0). One folder per target language.

```
resources/
└── es/
    └── glossary.csv   # en,es term pairs
```

## Glossary files

`<lang>/glossary.csv` is a two-column CSV (`en,es`) of canonical term
mappings — the local mirror of the "OpenTelemetry Terminology for
Localization" spreadsheet the End User SIG maintains. Skills read it with a
plain file read/grep; no network access or Sheets API needed.

To add a language, create `resources/<lang>/glossary.csv` with the same
`en,<lang>` header and shape.

**Keeping it in sync:** this CSV is a snapshot, not a live view. When the
source spreadsheet gains new terms, update the matching CSV by hand (or ask
a maintainer who has edit access to the sheet). If a skill needs a term
that's genuinely new to the glossary, add it here and flag it for the
spreadsheet too, so the two don't drift apart.
