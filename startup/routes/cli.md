# CLI Tool — Architecture Topics

Read this file when the user selected project type: **CLI tool**.

Walk through each topic below **one at a time** via AskUserQuestion.

---

## Common Topics

### 1. Language

This matters more for CLIs than any other project type:

Ask:
> What language for this CLI?
>
> - **A)** **Rust** — single binary, blazing fast, great CLI ecosystem (clap), cross-platform
> - **B)** **Go** — single binary, fast compilation, good CLI ecosystem (cobra), cross-platform
> - **C)** **TypeScript/Node** — fastest to build, npm distribution, requires Node runtime
> - **D)** **Python** — fastest to prototype, rich ecosystem, requires Python runtime
>
> **Recommendation by use case:**
> - Performance-critical / distributed widely → Rust (best UX for end users — single binary, instant startup)
> - Team already uses Go / internal tooling → Go
> - Node ecosystem / rapid prototyping → TypeScript (with bun for single-binary compile)
> - Data processing / scripting → Python

### 2. Database

CLIs rarely need a traditional database:
> Does this CLI need to persist data locally?
>
> - **A)** No — stateless, processes input and outputs
> - **B)** Config file — save user preferences
> - **C)** Local cache / state — SQLite or JSON file
> - **D)** Remote state — connects to an API or database

### 3. Authentication

> Does this CLI need to authenticate with a service?
>
> - **A)** No — standalone tool
> - **B)** API key — stored in config file or env var
> - **C)** OAuth — browser-based login flow (like `gh auth login`)
> - **D)** Service account — GCP or other cloud service

### 4. Hosting & Scaling

CLIs don't need hosting unless they have a backend component:
> Does this CLI have a server component?
>
> - **A)** No — purely client-side tool → no hosting needed
> - **B)** Yes — CLI talks to a backend API → the API part follows the API service route

### 5. CI/CD

> How should CI work?
>
> - **A)** GitHub Actions: lint + test on every push, build releases on tag
> - **B)** Minimal: just lint + test

---

## CLI-Specific Topics

### 6. Distribution

Ask:
> How will users install this CLI?
>
> - **A)** npm (`npm install -g my-cli`) — easiest for Node/Bun CLIs
> - **B)** Homebrew (`brew install my-cli`) — macOS/Linux, needs a tap repo
> - **C)** Binary releases on GitHub (users download from Releases page)
> - **D)** cargo install (`cargo install my-cli`) — for Rust CLIs
> - **E)** Multiple — npm + Homebrew + binary releases
>
> **Recommendation:** For wide distribution, provide binary releases at minimum. Add npm or Homebrew based on audience.

### 7. Config File Format & Location

Ask:
> Where should the CLI store its configuration?
>
> - **A)** XDG standard: `~/.config/{tool-name}/config.toml` (Linux standard, recommended)
> - **B)** Home directory: `~/.{tool-name}/config.json` (simple, visible)
> - **C)** Project-local: `.{tool-name}rc` or `.{tool-name}.json` (per-project config)
> - **D)** No config needed
>
> **File format recommendation:**
> - TOML — most readable, standard for CLI tools
> - JSON — widest tooling support, less human-friendly
> - YAML — readable but whitespace-sensitive (error-prone)

### 8. Update Mechanism

Ask:
> How should the CLI update itself?
>
> - **A)** Self-update command (`my-cli update`) — checks GitHub Releases, downloads new binary
> - **B)** Package manager update (`npm update -g`, `brew upgrade`, `cargo install`)
> - **C)** Manual — user downloads new release
>
> **Recommendation:** A + B. Self-update for binary users, package manager update for package users.

### 9. Shell Completions

Ask:
> Do you want shell completions (tab-complete commands and flags)?
>
> - **A)** Yes — generate for bash, zsh, fish (most CLI frameworks support this)
> - **B)** No — not needed
>
> **Recommendation:** A. It takes 2 lines of code in most CLI frameworks (clap, cobra, yargs) and dramatically improves UX.

---

## Scaffold Spec

For Rust (clap):
```
{project-slug}/
├── src/
│   ├── main.rs                 # Entry point + clap argument parsing
│   ├── cli.rs                  # CLI definition (commands, args)
│   └── commands/               # Command implementations
│       └── mod.rs
├── tests/
│   └── integration.rs
├── Cargo.toml
├── .github/
│   └── workflows/
│       └── release.yml         # Build + publish binaries on tag
└── rustfmt.toml
```

For TypeScript (bun):
```
{project-slug}/
├── src/
│   ├── index.ts                # Entry point
│   ├── cli.ts                  # Argument parsing (yargs/commander)
│   └── commands/               # Command implementations
├── tests/
│   └── cli.test.ts
├── package.json                # bin field configured
├── tsconfig.json
└── biome.json
```
