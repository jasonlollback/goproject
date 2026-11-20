# AGENTS.md

## Scope

This repository is goproject, free and open source: the CLI, the generator with its template, and the website. Everything runs on the user's machine. There is no goproject server, account or license.

## Brand

- Always write the product name as `goproject` in lowercase, including at the start of sentences and headings.
- Keep technical identifiers unchanged.

## Code Structure

The repository contains three parts: the CLI binary, the generator with its embedded template, and the goproject.lollback.com website.

### CLI (`cmd/goproject`)

Entry point in `cmd/goproject/main.go`. It wires the `cli.App` struct with function fields for dependency injection, then calls `App.Run(ctx, os.Args[1:])`. Commands:

- `new` — generates a new project via the generator
- `update` — updates an existing generated project on a new Git branch
- `changelog` — fetches and displays GitHub release notes
- `doctor` — inspects a generated project's tools and environment
- `version` — prints the CLI version

### Internal packages

- `internal/cli` — command implementations, argument parsing, styled output
- `internal/doctor` — tool and environment inspection for generated projects
- `internal/github` — GitHub API client for release notes
- `internal/project` — project extraction (reading `goproject.lock`, Git operations)
- `internal/tui` — interactive TUI wizard for `goproject new` (Charm/bubbletea)
- `internal/update` — template fetching from Go module proxy and Git branch creation

### Generator (`generator`)

Owns the template, the configuration types, and `goproject.lock`. Key files:

- `generator/config.go` — `Config` struct with all generation options (framework, database, payment, mail, OAuth, content, workspaces, storage, API, MCP)
- `generator/generate.go` — `Generate()` function that plans and renders template files
- `generator/select_test.go` — tests for template file selection logic

The template lives in `generator/_template/` and is embedded via `//go:embed`. Template files use Go `text/template` syntax for conditional inclusion based on the `Config`. File names encode the applicable axes, e.g. `pages.htmx.datastar.go` applies to both HTMX and Datastar frameworks.

### Website (`site`)

The goproject.lollback.com marketing and documentation site. Built with `templ` for Go templates, Tailwind CSS for styling, and Goldmark for Markdown rendering.

- `site/cmd/server` — HTTP server with routing, asset serving, and Markdown pages
- `site/cmd/devport` — helper to find a free port for dev mode
- `site/ui` — `templ` components (pages, layouts, blocks, shadcn-templ components)
- `site/docs` — documentation page definitions
- `site/markdown` — Goldmark parser with front matter support
- `site/brand` — brand constants
- `site/githubstars` — GitHub star count fetcher
- `content/` — Markdown content for docs and changelog

### Content (`content`)

Markdown files for the website documentation, read by the site server.

## Engineering

- Prefer the Go standard library.
- Keep packages small and concrete. Add interfaces only at real I/O boundaries used by tests.
- `generator` owns the template, the configuration and `goproject.lock`. The CLI uses its types directly.
- The CLI makes no network call except `changelog` (GitHub releases) and `update`, which downloads a released template from the Go module proxy. No telemetry.
- Nothing private enters this repository: no secrets, no customer data, no server names or internal plans.
- Run `go test -race ./...`, `go vet ./...`, `go build ./...`, and `go mod tidy -diff` before pushing.
- Then check what you changed, and only that: a small change gets a quick look at the changed page or generated project; a release with many changes gets the full run (generate every stack, click through it in Chromium and WebKit).

### Development workflow

Use `Taskfile.yml` for local development:

- `task dev` — runs the website with hot reload (templ, Tailwind CSS, Markdown watch)
- `task test` — runs `go mod tidy -diff`, `go vet ./...`, and `go test -race ./...`

The site uses `templ` for type-safe HTML components and Tailwind CSS for styling. Run `go tool templ generate` to regenerate `.templ.go` files.

### Template file naming

Files in `generator/_template/` use dot-separated suffixes to indicate which configuration axes they apply to. For example, `pages.htmx.datastar.go` is included when the framework is HTMX or Datastar. The generator resolves these names to determine which files to include for a given `Config`.

## Git

- Use a feature branch and pull request for substantial work.
- Keep commits attributed only to the human author. Do not add automation attribution.
- Commits follow Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`, with an optional scope like `feat(cli):`. A short first line, then a sentence or two of why when it helps.
- Commits and pull requests carry no AI attribution: no `Co-Authored-By` for a tool, no "Generated with" line.
- Release notes follow one format, the same in goproject, moin and shadcn-templ, so the eye finds things fast. The same text is the GitHub release (which `goproject changelog` shows) and the newest section of `CHANGELOG.md` (`## v…`, the date like `November 3, 2026`, then the notes), which `/changelog` shows:
  - First line: `**In short:**` and one sentence, what is new and why to update.
  - Then these sections in this order, empty ones left out: `### New` (what you can do now), `### Changed` (what behaves differently, everything that could surprise), `### Fixed` (what was broken and works now), `### Removed` (what is gone, and what to use instead), `### Upgrade` (always there: how to update, the exact command).
  - One bullet per thing: `- **Name**: one sentence of what the user gets.` Commands in backticks. What the user sees, never internals or commit hashes.
  - A big turning point may add a short `### Why` under the first line. Longer stories go in a blog post or the docs, linked.

## Prose

- Write project text naturally in the project owner's voice.
- Do not use em dashes or standalone hyphens as sentence separators.
