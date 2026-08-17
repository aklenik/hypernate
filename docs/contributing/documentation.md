# Contributing to the Docs

This documentation site is built with [MkDocs](https://www.mkdocs.org/) and the
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme. All content lives as
Markdown under the `docs/` directory, and the site is published automatically — you only need to
edit Markdown.

## Setup

The documentation toolchain is Python-based. Create a virtual environment and install the pinned
dependencies (CI uses Python 3.12):

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r docs/requirements.txt
```

## Previewing Locally

Run the live-reload preview server from the repository root:

```bash
mkdocs serve
```

Then open <http://127.0.0.1:8000>. The site rebuilds automatically as you save files.

Before pushing, reproduce the strict build that CI effectively performs:

```bash
mkdocs build --strict
```

The site is configured with `strict: true`, so **warnings are treated as errors** — a broken
internal link or a page missing from the navigation will fail the build.

## Editing and Adding Pages

- **Edit a page:** change the relevant `.md` file under `docs/`; the preview reloads.
- **Add a page:** create the `.md` file in the appropriate subdirectory, then register it in the
  `nav:` section of `mkdocs.yml`. A page that is not listed in `nav` fails the strict build.
- **Link between pages:** link to the Markdown *file*, not the built URL — for example
  `[Reporting Issues](issues.md)`. MkDocs resolves and validates these links.

## Markdown Conventions

### Indent Nested Lists by 2 Spaces

Indent each nesting level of a list by **2 spaces**, exactly as you would on GitHub:

```markdown
- Parent item
  - Nested item
    - Deeply nested item
```

This "just works" only because the site overrides its renderer.
[Python-Markdown](https://python-markdown.github.io/) (which MkDocs uses) requires 4 spaces per
level by default and silently flattens 2-space nesting, even though CommonMark and GFM accept it —
so lists that looked correct on GitHub used to break on the built site. The
[`mdx_truly_sane_lists`](https://github.com/radude/mdx_truly_sane_lists) extension configured in
`mkdocs.yml` aligns the site with the CommonMark behavior; only the indentation rule is overridden
(`truly_sane: false` keeps every other rendering behavior stock).

Two safety nets exist:

- CI lints all Markdown files with [markdownlint](https://github.com/DavidAnson/markdownlint-cli2)
  (rule `MD007`, at its default 2-space indent), which fails on deeper indents — under the
  override, a 4-space-nested item would collapse into run-on text on the site. Run it locally with
  `npx markdownlint-cli2`; the configuration lives in `.markdownlint-cli2.jsonc` at the repository
  root.
- When in doubt, check multi-level lists in the `mkdocs serve` preview rather than trusting the
  GitHub rendering.

## How the Site Is Validated and Published

Every pull request runs the **Build Documentation** workflow
(`.github/workflows/docs-build.yml`), which lints the Markdown sources (see
[Markdown Conventions](#markdown-conventions)) and executes the same `mkdocs build --strict` you
run locally — a broken link, a page missing from `nav`, or a lint violation fails the check before
the change can merge.

You do not deploy the site manually. The **Deploy Documentation** workflow
(`.github/workflows/docs-deploy.yml`) handles it:

- On every push to `main`, the site is published as the `latest` version.
- On a `v*.*.*` tag, that release is published as a versioned snapshot.

Versioning is managed by [mike](https://github.com/jimporter/mike), so there is no need to run
`mike` locally. Each page also has an *edit* link in the top-right that points back to its source
file on GitHub.
