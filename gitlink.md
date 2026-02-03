# Git Link — The Local-to-Remote Bridge

**Git Link** is a fast, terminal-native CLI that connects your local Git workflow directly to GitHub’s remote state.
It brings push verification, pull-request context, and CI/CD visibility **into the terminal**, so you don’t have to break focus by jumping between your IDE and a browser.

If you live in the terminal, Git Link makes sure the *entire* Git experience lives there too.

---

## 🧠 Ideology: *Stay in the Flow*

Modern development is plagued by **context switching**. Every time you alt-tab to a browser just to confirm a push, read a PR comment, or check CI status, you lose momentum.

Git Link is built around three simple beliefs:

* **Terminal-First**
  If the work happens in the terminal, verification and feedback should happen there as well.

* **Visual Confidence**
  Commands like `git add` or `git push` shouldn’t fail silently. Developers should *see* what just happened and why it matters.

* **Proactive Guardrails**
  It’s better to prevent bad pushes (conflicts, outdated branches, failing CI) than to discover them after the remote rejects your work.

---

## 🛠️ Tech Stack

Git Link is designed to be fast, portable, and IDE-agnostic. It ships as a single binary with zero runtime dependencies.

* **Language:** Go or Rust — for instant startup and predictable performance
* **TUI Framework:** Bubble Tea (Go) or Ratatui (Rust) — to build rich, interactive terminal interfaces
* **Git Integration:** Native Git plumbing commands for accurate local state
* **Remote Sync:** GitHub REST API and GraphQL API
* **Syntax Highlighting:** Chroma (Go) or Syntect (Rust)

---

## 📋 Core Features

| Feature                     | Description                                                                       | Feedback                                      |
| --------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------- |
| **Visual Command Feedback** | Enhances commands like `git add` with clear summaries instead of silent execution | Progress bars, staged file counts             |
| **Side-by-Side Diff View**  | Dual-pane horizontal diffs for reviewing changes in context                       | Syntax highlighting, intra-line diffs         |
| **Push Verification**       | Confirms that pushed commits are visible on GitHub                                | Verified checkmark when remote sync completes |
| **Inline PR Comments**      | Displays pull-request review comments directly inside the diff view               | Sticky-note style annotations                 |
| **Pre-Push Safety Guard**   | Traffic-light system that checks branch status and CI health                      | 🟢 Safe · 🟡 Behind · 🔴 Blocked              |
| **IDE-Agile**               | Runs in any terminal, independent of editor or IDE                                | Works in VS Code, JetBrains, Neovim, tmux     |

---

## 📐 How It Works

Git Link acts as an orchestration layer between your local repository and GitHub.

* It continuously reads state from your local `.git` directory
* In parallel, it queries GitHub for remote commits, PR metadata, and CI/CD status
* The TUI reconciles both views into a single, consistent source of truth

The result: you always know whether your local state *actually* matches what exists on the remote.

---

## 🚀 Future Roadmap

* **One-Click Fixes** — Apply PR review suggestions directly to local files
* **CI Log Streaming** — View GitHub Actions logs live inside the TUI
* **Multi-Repo Dashboard** — Monitor multiple repositories from a single interface
* **Provider Expansion** — GitLab and Bitbucket support through modular adapters

---

Git Link is not trying to replace Git or GitHub.
It exists to **close the gap between them** — without ever leaving the terminal.
