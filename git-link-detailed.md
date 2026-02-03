# Git Link - Project Draft

**Version:** 0.1.0  
**Last Updated:** February 3, 2026  
**Status:** Pre-Development Planning

---

## 🎯 Project Identity

**Name:** Git Link  
**Tagline:** The Local-to-Remote Bridge  
**Mission:** Eliminate developer context-switching by bridging the gap between local Git operations and GitHub remote status.

### Core Philosophy

Git Link is **not** a Git client replacement. It's a **companion tool** that provides the missing link between your local Git workflow and GitHub's remote state. Think of it as a real-time synchronization layer that answers the questions Git can't:

- "Is my push visible on GitHub yet?"
- "What's the CI status before I push?"
- "Are there PR comments I need to address?"
- "Am I about to cause a merge conflict?"

---

## 🔍 Problem Statement

### The Context-Switching Tax

Developers using Git + GitHub face constant friction:

1. **Push → Browser → Refresh Loop**
   ```
   $ git push
   [switches to browser]
   [refreshes GitHub]
   [checks if commits appeared]
   [switches back to terminal]
   ```

2. **CI Status Blindness**
   - Push code → wait 5 minutes → check if tests passed
   - No terminal-native way to monitor pipeline status

3. **PR Review Lag**
   - Reviewer leaves comments
   - Developer doesn't see them until they manually check GitHub
   - Delays response time by hours

4. **Merge Conflict Surprises**
   - Push fails because remote diverged
   - No warning system before attempting push

### Current Solutions Are Incomplete

- **Git CLI**: No awareness of GitHub state (CI, PRs, deployments)
- **GitHub CLI (`gh`)**: Great for scripting, but no live monitoring or TUI
- **Lazygit/Magit**: Excellent Git UIs, but no GitHub integration
- **GitHub Desktop**: GUI-only, not terminal-friendly
- **IDE Extensions**: Tied to specific editors, heavy resource usage

---

## 👥 Target Audience

### Primary Persona: Terminal-First Developer

**Profile:**
- Spends 60%+ of work time in terminal/IDE
- Uses Git CLI or TUI tools (Lazygit, tig)
- Works with GitHub for remote repositories
- Values speed and keyboard-driven workflows
- Frequently context-switches to browser for GitHub checks

**Pain Points:**
- Alt-tabbing breaks flow state
- Wants GitHub data without leaving terminal
- Needs pre-emptive conflict detection
- Desires CI status visibility during development

**Not Targeting:**
- Beginners learning Git (too advanced)
- Teams exclusively using GitLab/Bitbucket (v1 focus: GitHub)
- GUI-only developers (wrong interface paradigm)

---

## 🏗️ Technical Architecture

### Technology Stack

#### Language Options

**Option A: Go** *(Recommended)*
- ✅ Fast compilation, single binary
- ✅ Excellent concurrency (goroutines)
- ✅ Strong ecosystem: Bubble Tea, Lip Gloss
- ✅ Easy cross-compilation
- ❌ Garbage collection (minor for CLI tools)

**Option B: Rust**
- ✅ Maximum performance
- ✅ Memory safety guarantees
- ✅ Ratatui is mature
- ❌ Slower compile times
- ❌ Steeper learning curve for contributors

**Decision:** **Go** for faster iteration and better contributor onboarding.

#### Core Dependencies

```
┌─────────────────────────────────────────┐
│           Git Link Binary               │
├─────────────────────────────────────────┤
│  TUI Layer: Bubble Tea + Lip Gloss      │
│  Git Integration: go-git (libgit2)      │
│  HTTP Client: stdlib net/http           │
│  Config: viper                           │
│  CLI Framework: cobra                    │
└─────────────────────────────────────────┘
          │                    │
          ↓                    ↓
    ┌──────────┐        ┌──────────────┐
    │   Git    │        │  GitHub API  │
    │ (local)  │        │  (REST/GQL)  │
    └──────────┘        └──────────────┘
```

### API Integration Strategy

#### GitHub REST API (Primary)
- `/repos/{owner}/{repo}/commits/{sha}` - Verify push
- `/repos/{owner}/{repo}/pulls/{number}` - PR details
- `/repos/{owner}/{repo}/commits/{ref}/status` - CI status
- `/repos/{owner}/{repo}/pulls/{number}/reviews` - PR reviews

#### GitHub GraphQL API (Secondary)
- Batch queries for PR + CI + reviews in single request
- Reduces rate limit consumption
- Used for dashboard view

#### Rate Limit Management

**Limits:**
- REST: 5,000 requests/hour (authenticated)
- GraphQL: 5,000 points/hour (query cost varies)

**Strategy:**
```
1. Cache aggressively (5-minute TTL for CI status)
2. Use ETags for conditional requests
3. Batch queries in GraphQL when possible
4. Exponential backoff on 429 responses
5. Local rate limit tracking with token bucket algorithm
```

### Authentication

**Supported Methods:**

1. **GitHub CLI Credential Sharing** *(Preferred)*
   ```bash
   gh auth status  # Check if gh is authenticated
   # Read credentials from ~/.config/gh/hosts.yml
   ```

2. **Personal Access Token**
   ```bash
   gitlink auth login
   # Prompts for token, stores in OS keychain
   ```

3. **OAuth Device Flow** *(Future)*
   - Full OAuth for teams

**Security:**
- Store tokens in OS keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service)
- Never log or display tokens
- Minimal scope: `repo`, `read:user`

---

## ✨ Feature Specification

### Phase 1: Core Bridge Functions (v0.1.0 - MVP)

#### 1. Push Verification

**Command:** `gitlink verify`

**Behavior:**
```bash
$ git push
$ gitlink verify

⏳ Verifying push status...
✓ Commit abc1234 visible on github.com/user/repo
✓ Branch 'feature/new-widget' updated remotely
⏱️  Push completed 3 seconds ago
```

**Implementation:**
- Poll GitHub API for commit SHA
- Retry with exponential backoff (max 30s)
- Display progress spinner
- Exit code: 0 (success), 1 (not found), 2 (error)

#### 2. Remote Status Dashboard

**Command:** `gitlink status` or `gitlink` (default)

**TUI Layout:**
```
╭─ Git Link Status ────────────────────────────────────────╮
│ Repository: username/awesome-project                      │
│ Branch: feature/auth-system                               │
│                                                            │
│ 📊 Sync Status                                            │
│   Local:  3 commits ahead of origin/main                  │
│   Remote: 0 commits behind (up to date)                   │
│                                                            │
│ 🔀 Pull Request: #142                                     │
│   Status: ✓ Approved (2/2)                                │
│   Checks: ✓ All passed (4/4)                              │
│   - build-linux    ✓ 2m 34s                               │
│   - build-windows  ✓ 3m 12s                               │
│   - test-unit      ✓ 1m 45s                               │
│   - lint           ✓ 0m 32s                               │
│                                                            │
│ 💬 Comments: 3 unresolved threads                         │
│   - src/auth.go:45 "Consider using bcrypt"                │
│   - README.md:12 "Typo: 'teh' → 'the'"                    │
│   - src/main.go:89 "Add error handling here"              │
│                                                            │
│ 🚀 Deployment: staging (abc1234)                          │
│   Production: 2 commits behind                            │
╰───────────────────────────────────────────────────────────╯

Press 'q' to quit, 'r' to refresh, 'p' to view PR
```

**Refresh Strategy:**
- Auto-refresh every 30 seconds
- Manual refresh on 'r' key
- Shimmer effect on stale data

#### 3. Pre-Push Safety Check

**Command:** `gitlink check` (or auto-run via Git hook)

**Traffic Light System:**

```bash
$ gitlink check

🔍 Running pre-push safety checks...

🟢 Remote Sync
   ✓ No remote commits since last fetch

🟢 Merge Conflicts
   ✓ No conflicts detected with origin/main

🟡 CI Status
   ⚠️  Previous commit (def5678) has failing tests
   ⚠️  Push anyway? This may fail CI again.

🟢 Branch Protection
   ✓ All required reviewers approved

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Overall: ⚠️  CAUTION - Proceed with care
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Continue with push? [y/N]:
```

**Checks:**
1. **Remote Drift** - `git fetch` + compare HEAD with origin
2. **Merge Conflicts** - Test-merge with target branch
3. **CI Status** - Check if last commit on current branch passed
4. **Branch Protection** - Verify PR approval requirements met

**Exit Codes:**
- 0: All green, safe to push
- 1: Warnings, proceed with caution
- 2: Blocking issues, do not push

### Phase 2: Workflow Enhancers (v0.2.0)

#### 4. PR Comment Overlay

**Command:** `gitlink pr-comments [file]`

**TUI View:**
```
╭─ src/auth.go ─────────────────────────────────────────────╮
│  42 | func HashPassword(password string) string {            │
│  43 |     // TODO: Implement secure hashing                  │
│  44 |     return password                                    │
│  45 | }                                                       │
│     │                                                         │
│     │ 💬 @reviewer (2 hours ago)                             │
│     │ This is insecure! Use bcrypt.GenerateFromPassword()    │
│     │                                                         │
│     │ 💬 @you (1 hour ago)                                   │
│     │ Good catch, will fix in next commit                    │
│     │                                                         │
│  46 |                                                         │
│  47 | func ValidatePassword(password, hash string) bool {    │
╰─────────────────────────────────────────────────────────────╯
```

**Implementation:**
- Fetch PR comments via GitHub API
- Match comment line numbers to current file
- Handle line number drift (comments may reference old line numbers)
- Show comment thread with timestamps

#### 5. Branch Sync Assistant

**Command:** `gitlink sync`

**Interactive Mode:**
```bash
$ gitlink sync

📡 Fetching latest from origin...

⚠️  Your branch is 3 commits behind origin/main:

  def5678 feat: Add OAuth support (Jane, 2 hours ago)
  abc1234 fix: Resolve memory leak (Bob, 5 hours ago)
  789wxyz docs: Update API guide (Alice, 1 day ago)

Options:
  [p] Pull (merge) these changes now
  [r] Rebase your branch on top of main
  [v] View full diff
  [s] Skip for now

Your choice:
```

**Smart Detection:**
- Warns if pull would create merge commits
- Suggests rebase if history is clean
- Detects if you're in the middle of a merge/rebase

#### 6. Watch Mode (Daemon)

**Command:** `gitlink watch`

**Behavior:**
- Runs in background
- Monitors GitHub for PR events:
  - New reviews
  - Comment replies
  - CI status changes
  - Approvals/rejections
- Sends desktop notifications
- Logs events to `~/.gitlink/watch.log`

**Notification Examples:**
```
[Desktop Notification]
Git Link
✓ PR #142: All checks passed! Ready to merge.

[Desktop Notification]
Git Link
💬 New comment on PR #142 from @reviewer
```

---

## 🎨 User Experience Design

### Design Principles

1. **Speed First**: Sub-100ms startup time
2. **Non-Intrusive**: Never block Git operations
3. **Glanceable**: Key info visible in 1 second
4. **Keyboard-Driven**: No mouse required
5. **Beautiful Defaults**: Works great out of the box

### Visual Style

**Color Palette** (using Lip Gloss):
```
Primary:   #7C3AED (Purple)
Success:   #10B981 (Green)
Warning:   #F59E0B (Amber)
Error:     #EF4444 (Red)
Muted:     #6B7280 (Gray)
```

**Typography:**
- Headers: Bold
- Code/Commits: Monospace
- Status indicators: Emoji + text

**Status Indicators:**
```
✓ Success / Complete
⚠️ Warning / Caution
✗ Error / Failed
⏳ Loading / In Progress
🔄 Syncing
🚀 Deployed
💬 Comments
🔀 Pull Request
```

### Command Design Philosophy

**Format:** `gitlink [command] [flags]`

**Commands should be:**
- Memorable verbs (verify, check, sync)
- Self-explanatory
- Tab-completable

**Flags should be:**
- Short form: `-f`
- Long form: `--format`
- Consistent across commands

---
