# phpbbdocs-crowdin

Shared Crowdin sync tooling for phpBB documentation projects built on the
DocBook-4-XML convention used by
[phpbbdocs-hugo](https://github.com/phpbbmodders/phpbbdocs-hugo) (hand-authored
end-user chapters, pandoc-converted developer docs, and a top-level Proteus
book file). Given a docs project directory, it creates or updates the
project's Crowdin project, then uploads source XML to Crowdin or downloads
completed translations back into the docs project.

This is the sibling of
[phpbb-crowdin](https://github.com/phpbbmodders/phpbb-crowdin), which does the
same job for phpBB extensions' `.php` language files. The two tools diverge
in a few deliberate ways — see [Design notes](#design-notes) below.

## Contents

- **`crowdin-init.sh`** — the sync script. Run with `-h`/`--help` for the full
  option list.
- **`crowdin.conf`** — shared configuration applied to every docs project
  (source language, source-sync toggle, CLI template path).
- **`crowdin.yml.template`** — the fixed header of the Crowdin CLI
  configuration; `crowdin-init.sh` appends the applicable fragments below and
  fills in the language mapping at runtime.
- **`fragments/`** — one Crowdin CLI `files:` entry per known DocBook-XML
  source tree (the Proteus book file, hand-authored chapters, pandoc dev
  docs). Only a fragment whose source files actually exist in the target
  docs project is included in the rendered config — see
  [Design notes](#design-notes).
- **`docs-project.conf.example`** — template for the optional
  `docs-project.conf` override file `crowdin-init.sh` reads from the
  *target* docs project's own repository root (not from this tool's
  repository) — see the file for what it controls.

## Requirements

- `bash`, `curl`, `jq`
- [Crowdin CLI](https://developer.crowdin.com/cli-tool/) (only required for
  source upload / translation download)

## Usage

```bash
export CROWDIN_API_TOKEN=your-personal-access-token
./crowdin-init.sh /path/to/docs-project
```

Add `-n`/`--dry-run` to preview project changes and the source upload
without applying them, or `--download`/`--download-dry-run` to pull completed
translations back into the docs project.

Add `-s`/`--seed-untranslated` to also upload this docs project's own local
translation for any language that is still at 0% translated on Crowdin
(every one of its files, not just some) — this only ever adds a starting
point for translators; it never touches a language once real work exists for
it on Crowdin, however partial. Requires a local translation that actually
differs from the English source; a language directory that's just an
untouched copy of English is not seeded.

## Design notes

A few places this tool deliberately departs from phpbb-crowdin's approach,
because the two domains aren't actually the same:

- **Target languages and custom honorific variants are the same fixed list
  phpbb-crowdin uses**, not discovered from the docs project's own
  `content/<lang>/` directories. A docs project invites translation into
  phpBB's whole language ecosystem the same way a phpBB extension does, even
  before it has a directory for a given language — e.g. phpbbdocs-hugo
  currently only has English, Danish, French, and German content, but its
  Crowdin project offers every language phpbb-crowdin's `PHPBB_LANGUAGES` and
  `CUSTOM_LANGUAGES` do (German's formal/casual split included), since
  translators shouldn't have to wait for a directory to exist first. See
  `DOCS_LANGUAGES`/`CUSTOM_LANGUAGES` in `crowdin-init.sh` — kept identical to
  phpbb-crowdin's own arrays.
- **Project identity has no composer.json to read.** A docs project built
  this way has no package manifest. Identity is derived from the docs
  project's git remote (or directory name with no remote), with an optional
  `docs-project.conf` override — see `docs-project.conf.example`.
- **`translatable_elements` (Crowdin's XPath-based per-element opt-in) is not
  used.** The XML config here uses the verified-safe `translate_content: 1` /
  `translate_attributes: 0` pair instead — confirmed against Crowdin's own
  documentation and a live `crowdin config lint` run against a real docs
  project. It correctly leaves every structural attribute alone
  (`imagedata`'s `fileref`, `link`/`xref`'s `linkend`, every element's `id`)
  since none of those are ever attributes worth translating in this DocBook
  vocabulary. It does **not** exclude `<programlisting>`/`<literallayout>`
  (code examples, ASCII art) from translation — Crowdin's advanced
  `translatable_elements` XPath option could do that, but its exact
  path-matching dialect (whether it supports `//` descendant matching or
  requires a full absolute path per nesting depth) could not be confirmed
  against Crowdin's documentation. Verify that against a real Crowdin
  project before relying on it, and update `fragments/*.yml.fragment`
  accordingly.
- **A files: entry with a zero-match source glob is a hard `crowdin config
  lint` failure** (confirmed live, not assumed from docs). Since not every
  docs project this tool targets is guaranteed to have all three known
  source trees, `crowdin-init.sh` detects which ones actually exist in the
  target project and builds the runtime `crowdin.yml` from only the matching
  fragments in `fragments/`, rather than shipping one static template with
  all three always present.

## Contributing

Contributions are welcome!

- **Bug reports**: [Open an issue](https://github.com/phpbbmodders/phpbbdocs-crowdin/issues).
- **Everything else** (questions, feature requests, ideas, general discussion): open an issue as well — this repository does not currently use Discussions.
- Pull requests are welcome for bug fixes or discussed features.

## Acknowledgments

- Code review, bug fixes, and documentation assisted by [Claude](https://www.anthropic.com/claude).

## License

This project is licensed under the **GNU General Public License v2.0**, matching its sibling tool [phpbb-crowdin](https://github.com/phpbbmodders/phpbb-crowdin).

See [LICENSE](LICENSE) for more information.
