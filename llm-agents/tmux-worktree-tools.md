---
created: 2026-02-15
---

# Parallel LLM coding agents with tmux and git worktrees

_February 15, 2026_

The intersection of LLM-powered coding agents, tmux session management, and git worktree isolation has become one of the most active categories in developer tooling. The core pattern is simple: git worktrees provide filesystem isolation so multiple agents don't collide, tmux provides persistent and observable terminal sessions, and an orchestration layer ties the lifecycle together — from spinning up an agent on a feature branch to merging the result and cleaning up.

This document catalogs every significant tool, script, and workflow in the space, with feature comparisons, architectural analysis, and links to the community commentary that has shaped the ecosystem.

---

## Table of Contents

- [Taxonomy](#taxonomy)
- [Tool Profiles](#tool-profiles)
  - [Full-Featured TUI Session Managers](#1-full-featured-tui-session-managers)
  - [Opinionated Workflow Tools](#2-opinionated-workflow-tools)
  - [Tmux-Focused Tools](#3-tmux-focused-tools)
  - [Worktree-Focused Tools](#4-worktree-focused-tools)
  - [Lightweight Scripts and Gists](#5-lightweight-scripts-and-gists)
  - [First-Party: Claude Code Agent Teams](#6-first-party-claude-code-agent-teams)
- [In-Category Comparison Tables](#in-category-comparison-tables)
- [Cross-Category Comparison Matrix](#cross-category-comparison-matrix)
- [Architecture and Design Analysis](#architecture-and-design-analysis)
- [Community Commentary and Discussion](#community-commentary-and-discussion)
- [Blog Posts and Long-Form Writeups](#blog-posts-and-long-form-writeups)
- [Recommendations by Use Case](#recommendations-by-use-case)

---

## Taxonomy

The tools in this space fall into six broad categories, though some straddle boundaries:

| Category | Philosophy | Examples |
|---|---|---|
| **TUI Session Managers** | General-purpose "mission control" for multiple agent sessions | claude-squad, ccmanager, agent-deck, agent-of-empires |
| **Opinionated Workflow Tools** | End-to-end lifecycle management with strong conventions | workmux, dmux, barrel, forestui, multi-agent-workflow-kit |
| **Tmux-Focused** | Enhance the tmux experience specifically for agent workflows | claude-tmux, vscode-ext-tmux-worktree |
| **Worktree-Focused** | Center the workflow around git worktree management | agenttools/worktree, worktree-manager-skill |
| **Lightweight Scripts** | Minimal shell scripts or slash commands for quick parallelism | andynu's gist, worksfornow's /delegate |
| **First-Party** | Anthropic's own agent teams feature built into Claude Code | Claude Code Agent Teams (experimental) |

---

## Tool Profiles

### 1. Full-Featured TUI Session Managers

These tools provide a standalone terminal UI that acts as "mission control" for managing multiple concurrent agent sessions, each backed by tmux and git worktrees.

---

#### claude-squad

| | |
|---|---|
| **Repository** | [github.com/smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad) |
| **Language** | Go |
| **Install** | `curl -fsSL https://raw.githubusercontent.com/smtg-ai/claude-squad/main/install.sh \| bash` |
| **Supported Agents** | Claude Code, Aider, Codex, OpenCode, Amp |
| **Maturity** | One of the earliest tools in the space; well-established community |

**Overview.** claude-squad was one of the first tools to formalize the tmux + worktree + agent pattern into a proper TUI. It uses tmux to create isolated terminal sessions for each agent and git worktrees to isolate codebases so each session works on its own branch. The binary is called `cs` and provides a clean terminal interface for creating, monitoring, and switching between agent sessions.

**Key Features:**

- TUI for creating, listing, attaching to, and killing agent sessions
- Each session gets its own tmux session and git worktree
- Supports multiple agent programs via `--program` flag
- Experimental `--autoyes` flag for auto-accepting agent prompts
- `reset` command to clean up all stored instances
- Custom binary name via install script (`--name`)

**Limitations.** claude-squad is tmux-native and requires tmux to be installed. It lacks some of the more advanced features found in newer tools (devcontainer support, session forking, MCP management, auto-approval via AI). Its README explicitly compares itself to ccmanager: "If you love tmux-based workflows, stick with Claude Squad!"

---

#### ccmanager

| | |
|---|---|
| **Repository** | [github.com/kbwo/ccmanager](https://github.com/kbwo/ccmanager) |
| **Language** | TypeScript (Node.js) |
| **Install** | `npx ccmanager` |
| **Supported Agents** | Claude Code, Gemini CLI, Codex CLI, Cursor Agent, Copilot CLI, Cline CLI, OpenCode, Kimi CLI |
| **Maturity** | Feature-rich; broadest agent support of any tool |

**Overview.** ccmanager is the most feature-dense tool in this category, supporting the widest range of coding agents while being completely self-contained — it does not require tmux to be installed. It manages sessions, worktrees, and the full agent lifecycle with an impressive set of advanced capabilities.

**Key Features:**

- **Session data copying:** Can copy Claude Code conversation history, context, and project state when creating new worktrees, preserving context across branches
- **Worktree hooks:** Execute custom commands when worktrees are created (e.g., `npm install`, `bundle install`), with environment variables for worktree path, branch name, and git root
- **Auto-directory generation:** Automatically generates worktree directory paths from branch names using customizable patterns
- **AI-powered auto-approval:** Uses Claude Haiku to analyze prompts and determine if they need manual approval; safe fallback to manual on errors/timeouts; interruptible with any key
- **Devcontainer support:** Run agents inside devcontainers while keeping the manager on the host machine, enabling sandboxed environments with restricted network access
- **Multi-project mode:** Manage multiple git repositories from a single interface with `CCMANAGER_MULTI_PROJECT_ROOT`; automatic project discovery, recent projects surfacing, and visual indicators for session counts per project
- **State hooks:** Execute custom commands on Claude Code session status changes for desktop notifications, logging, or integration with other tools
- **Vi-like search:** Press `/` to filter projects or worktrees

**Limitations.** Being Node.js/TypeScript, it has a heavier runtime dependency than Go or Rust alternatives. The TUI is less visually polished than agent-deck's Bubble Tea interface.

---

#### agent-deck

| | |
|---|---|
| **Repository** | [github.com/asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck) |
| **Language** | Go (Bubble Tea TUI framework) |
| **Install** | Installer script or `make build` |
| **Supported Agents** | Claude Code, Gemini CLI, OpenCode, Codex CLI, and more |
| **Maturity** | Active development; comprehensive documentation |

**Overview.** agent-deck brands itself as "mission control for your AI coding agents" and focuses on the experience of managing many concurrent sessions efficiently. It adds AI-specific intelligence on top of tmux: smart status detection that knows when Claude is thinking vs. waiting, session forking with context inheritance, MCP management, and organized groups.

**Key Features:**

- **Session forking:** Fork any Claude conversation instantly; each fork inherits the full conversation history
- **MCP server management:** Attach MCP servers without touching config files; toggle per project or globally; socket pooling shares MCP processes across all sessions via Unix sockets, reducing MCP memory usage by 85-90%; connections auto-recover from MCP crashes in ~3 seconds
- **Tmux status bar integration:** Waiting sessions appear in your tmux status bar; press `Ctrl+b 1-6` to jump directly
- **Global search:** Fuzzy-search across all conversations with `/`
- **Worktree locations:** Configurable as "sibling" (default), "subdirectory", or a custom path; `--location` flag overrides per session
- **Groups:** Organize sessions into logical groups
- **Auto-update:** Checks for updates automatically; `agent-deck update` or `auto_update = true`
- **WSL support:** Works via WSL2 on Windows

**Limitations.** Creates its own tmux sessions with the prefix `agentdeck_*`, which means it doesn't integrate into existing tmux session workflows as seamlessly as workmux. The installer modifies `~/.tmux.conf` (though it backs it up and you can skip with `--skip-tmux-config`).

---

#### agent-of-empires

| | |
|---|---|
| **Repository** | [github.com/njbrake/agent-of-empires](https://github.com/njbrake/agent-of-empires) |
| **Language** | Rust |
| **Install** | `brew install njbrake/aoe/aoe` or curl script or `cargo build --release` |
| **Supported Agents** | Claude Code, OpenCode, Mistral Vibe, Codex CLI, Gemini CLI (auto-detected) |
| **Maturity** | Newer; inspired by agent-deck |

**Overview.** agent-of-empires (`aoe`) is a Rust-based TUI that takes the agent-deck concept and adds Docker sandboxing. Sessions are tmux sessions running in the background — you can open and close `aoe` as often as you like, and sessions persist until explicitly deleted.

**Key Features:**

- **Docker sandboxing:** Isolate agents in containers with shared auth
- **TUI controls:** `n` to create, `Enter` to attach, `t` for terminal view, `D` for diff view, `d` to delete
- **Auto-detection:** Automatically detects which agents are installed on your system
- **Git worktree integration:** `aoe add . -w feat/my-feature -b` creates a session on a new branch
- **Mobile access:** Run `aoe` inside a tmux session when connecting from mobile
- **Installable via Homebrew** on macOS

**Limitations.** Newest of the session managers, so smallest community. Debug logging requires an environment variable (`AGENT_OF_EMPIRES_DEBUG=1`). Has a known interaction issue with Claude Code's tmux handling (references `anthropics/claude-code#1913`).

---

### 2. Opinionated Workflow Tools

These tools take a stronger stance on how the agent-worktree-tmux lifecycle should work, embedding conventions for branching, merging, environment setup, and cleanup.

---

#### workmux

| | |
|---|---|
| **Repository** | [github.com/raine/workmux](https://github.com/raine/workmux) |
| **Language** | TypeScript |
| **Install** | npm (see repo) |
| **Supported Agents** | Claude Code, Codex, Gemini, and any CLI agent via config |
| **Maturity** | Well-designed; author also maintains a constellation of related tmux tools |

**Overview.** workmux is self-described as a "giga opinionated zero-friction workflow tool for managing git worktrees and tmux windows as isolated development environments." Rather than providing a separate TUI, workmux maps each worktree to a tmux window within your existing tmux session. This makes it the most natural fit for developers who already live in tmux.

**Key Features:**

- **Tmux-native model:** Each worktree is a tmux window; context switching is just switching tabs
- **Full lifecycle:** `workmux add feature-name` creates worktree + tmux window + launches agent; `workmux merge` merges branch into main and cleans up everything (tmux window, worktree, local branch); `workmux remove` for PR-based workflows where merge happens on remote
- **Agent status in window list:** Status icons show in tmux window names: 🤖 working, 💬 waiting for input, ✅ done
- **`.workmux.yaml` config:** Per-project configuration for panes, file operations, hooks, and agent choice
  - `files.symlink`: Share build caches (e.g., `.turbo`, `node_modules`) across worktrees via symlinks
  - `files.copy`: Per-worktree copies of files like `.env`
  - `post_create` hooks: Run commands at worktree lifecycle points (receive `WM_HANDLE`, `WM_WORKTREE_PATH`, `WM_PROJECT_ROOT`, `WM_CONFIG_DIR` env vars)
  - `panes`: Configure the pane layout, commands, and focus for each worktree window
- **Claude Code `/worktree` slash command:** Delegate tasks from a Claude Code conversation directly into new worktrees
- **Monorepo support:** Nested `.workmux.yaml` configs with `<global>` includes
- **Sensible defaults:** Projects with a `CLAUDE.md` file auto-launch the agent; others open a shell
- **Experimental WezTerm backend** as alternative to tmux
- **`prefix + Tab`** to toggle between your two most recent agents

**Related tools by the same author (raine):**

| Tool | Description |
|---|---|
| [tmux-tools](https://github.com/raine/tmux-tools) | Collection of tmux utilities including file picker, smart sessions |
| [tmux-file-picker](https://github.com/raine/tmux-file-picker) | Pop up fzf in tmux to insert file paths; designed for AI coding assistants |
| [tmux-bro](https://github.com/raine/tmux-bro) | Smart tmux session manager with project-specific sessions |
| [git-surgeon](https://github.com/raine/git-surgeon) | Non-interactive hunk-level git staging for AI agents |
| [claude-history](https://github.com/raine/claude-history) | Search and view Claude Code conversation history with fzf |
| [consult-llm-mcp](https://github.com/raine/consult-llm-mcp) | MCP server that lets Claude Code consult stronger AI models (o3, Gemini, GPT-5.1 Codex) |

**Limitations.** Requires buy-in to the "one window per worktree" model. All worktrees share the same tmux session, which means you can't easily isolate groups of worktrees across different tmux sessions. The TypeScript runtime adds some overhead compared to Go/Rust alternatives.

---

#### dmux

| | |
|---|---|
| **Repository** | [github.com/formkit/dmux](https://github.com/formkit/dmux) |
| **Language** | TypeScript (from the FormKit team) |
| **Install** | See repo |
| **Supported Agents** | Claude Code, OpenCode |
| **Maturity** | Newer; simpler and more focused than workmux |

**Overview.** dmux is a more minimalist take on the agent multiplexer concept. Each project gets its own tmux session (`dmux-projectname`), and new panes within that session automatically create git worktrees in sibling directories. It uses OpenRouter for AI-generated branch names and commit messages.

**Key Features:**

- **Session-per-project isolation:** Each project gets its own tmux session
- **AI-generated branch names** via OpenRouter
- **Smart merging:** One-command merge workflow with auto-commit, AI-generated commit messages, and worktree cleanup
- **Agent integration:** Launches Claude Code with `--accept-edits` or submits prompts to OpenCode automatically
- **Simple TUI:** `n` for new pane, `j` to switch focus, `m` to merge
- **Worktree directory convention:** Main source in a `main` directory; worktrees as siblings

**Limitations.** Requires an OpenRouter API key for the AI branch naming feature. Supports fewer agents than the session managers. Less configurable than workmux.

---

#### barrel

| | |
|---|---|
| **Repository** | [github.com/txtx/barrel](https://github.com/txtx/barrel) |
| **Language** | (See repo) |
| **Install** | See repo |
| **Supported Agents** | Claude Code, Codex (OpenAI), OpenCode, Antigravity (Google) |
| **Maturity** | Active development; unique agent portability angle |

**Overview.** barrel's pitch is "portable agents across LLMs, reproducible terminal workspaces." The core insight is that you should write agent definitions once and have them work across any LLM-powered coding tool. barrel handles the symlinking and configuration so that a single `agents/` directory gets picked up by Claude Code, Codex, OpenCode, or Antigravity.

**Key Features:**

- **Agent portability:** Write agent definitions once; barrel symlinks them into each tool's expected location
- **`barrel.yaml` workspace config:** Declarative definition of your entire workspace layout — which shells, which agents, which commands, which pane positions
- **Git worktree integration:** `barrel -w feat/auth` launches directly in a worktree
- **Session management:** `barrel session list`, `barrel session join`, `barrel session kill`
- **Agent management:** `barrel agent list`, `barrel agent import <path>`, `barrel agent new`, `barrel agent fork`, `barrel agent link`
- **Tmux terminal profiles:** Configurable grid layout for panes (col/row positioning)
- **Reproducible environments:** Close everything, come back tomorrow, run `barrel` again — same layout

**Limitations.** The agent portability angle means it's doing more configuration management than runtime orchestration. Less focus on status monitoring and lifecycle management compared to workmux or agent-deck.

---

#### forestui

| | |
|---|---|
| **Repository** | [github.com/flipbit03/forestui](https://github.com/flipbit03/forestui) |
| **Language** | Python (Textual framework) |
| **Install** | `uv tool install forestui` or curl installer |
| **Supported Agents** | Claude Code (with YOLO mode) |
| **Maturity** | Recently posted on Hacker News (Feb 2026); active |

**Overview.** forestui is a Textual-based TUI that manages git worktrees across multiple repositories, with tmux window integration and Claude Code session tracking. It organizes tmux windows with structured naming: `edit:repo:branch`, `claude:repo:branch`, `term:repo:branch`.

**Key Features:**

- **Multi-repository management:** Add and track multiple Git repositories in a "forest"
- **Claude session tracking:** Start Claude sessions (normal or YOLO mode) per worktree; track and resume sessions
- **TUI editor integration:** Opens TUI editors (vim, nvim, helix, etc.) in named tmux windows
- **Multi-forest support:** Manage multiple forest directories — `forestui ~/work` and `forestui ~/personal` maintain independent state
- **Settings:** Configurable default editor, branch prefix, and theme via `~/.config/forestui/settings.json`
- **Keyboard shortcuts:** `a` add worktree, `e` open editor, `t` terminal, `n` Claude session, `y` YOLO mode, `h` toggle archive, `d` delete

**Limitations.** Requires Python 3.14+, which is a very recent requirement. Claude Code-specific rather than multi-agent. Focused on worktree and session management rather than the merge/cleanup lifecycle.

---

#### multi-agent-workflow-kit (maw)

| | |
|---|---|
| **Repository** | [github.com/laris-co/multi-agent-workflow-kit](https://github.com/laris-co/multi-agent-workflow-kit) |
| **Language** | Shell scripts (via uvx) |
| **Install** | `uvx --no-cache --from git+https://github.com/Soul-Brews-Studio/multi-agent-workflow-kit.git@v0.5.1 multi-agent-kit init` |
| **Supported Agents** | Any CLI agent |
| **Maturity** | Self-described as "proof of concept"; exploratory |

**Overview.** maw provides a reusable toolkit for tmux + git worktree multi-agent workflows, with a unique inter-agent communication system. Each agent gets its own git worktree and branch, and all agents are visible in a single tmux session with split panes.

**Key Features:**

- **Inter-agent messaging:** `maw hey <agent> <msg>` sends a message to a specific agent
- **Broadcast commands:** `maw send "<cmd>"` sends to all panes
- **Navigation:** `maw warp <target>` navigates to a worktree (by agent number or 'root')
- **Smart git sync:** `maw sync` does context-aware git operations
- **CLAUDE.md guidelines:** `maw catlab [gist-url]` downloads guidelines, optionally from a custom gist
- **`agents.yaml` config:** Defines the agents and their layout
- **Direnv support:** `maw direnv` enables direnv in all worktrees

**Limitations.** Proof of concept status means rough edges. Requires git ≥2.5, tmux ≥3.2, `yq`, and `uvx`. The inter-agent messaging is basic (sends tmux keys rather than structured communication).

---

### 3. Tmux-Focused Tools

These tools enhance the tmux experience specifically for agent session management, rather than building a separate TUI layer.

---

#### claude-tmux

| | |
|---|---|
| **Repository** | [github.com/nielsgroen/claude-tmux](https://github.com/nielsgroen/claude-tmux) |
| **Language** | Rust (Ratatui) |
| **Install** | Build from source with Cargo |
| **Focus** | Claude Code session management within tmux |

**Overview.** claude-tmux provides a TUI that opens as a tmux popup (via `Ctrl-b, Ctrl-c`) showing all your Claude Code sessions. It identifies sessions by looking for panes running the `claude` command, making it zero-config for discovery.

**Key Features:**

- **Live preview:** See the last lines of the selected session's Claude Code pane with full ANSI color support
- **Session management:** Create, kill, and rename sessions without leaving the TUI
- **Expandable details:** View metadata like window count, pane commands, uptime, and attachment status
- **Fuzzy filtering:** Quickly filter sessions by name or path
- **Git worktree support:** Create and manage sessions in worktrees
- **Pull request support:** Integrates with `gh` CLI for PR workflows

**Limitations.** Claude Code-specific (doesn't detect other agents). Rust/Ratatui means building from source. AGPL-3.0 license may be a consideration for some.

---

#### vscode-ext-tmux-worktree

| | |
|---|---|
| **Repository** | [github.com/kargnas/vscode-ext-tmux-worktree](https://github.com/kargnas/vscode-ext-tmux-worktree) |
| **Language** | TypeScript (VS Code Extension) |
| **Install** | VS Code Marketplace |
| **Focus** | Bridging VS Code and tmux worktree sessions |

**Overview.** This VS Code extension auto-attaches to the right tmux session when you open a worktree folder. The key value proposition: agents keep running in the background via tmux even when VS Code is closed, and you can reconnect from anywhere — including from a phone via Termux.

**Key Features:**

- **Auto-attach:** Automatically connects to the matching tmux session when opening a worktree
- **Persistent sessions:** Sessions survive VS Code closes
- **Sidebar panel:** Dedicated sidebar showing all git worktrees and their tmux sessions
- **Split panes and windows** from the context menu
- **New task creation:** Click `+` in the panel, enter a branch name, and you're working in a new worktree with its own tmux session
- **Orphan cleanup:** Detect and clean up tmux sessions without matching worktrees

**Limitations.** VS Code-specific. Requires both tmux and VS Code, which may not appeal to terminal-purist workflows.

---

### 4. Worktree-Focused Tools

These tools center the workflow around git worktree management, with agent and tmux integration as supporting features.

---

#### agenttools/worktree

| | |
|---|---|
| **Repository** | [github.com/agenttools/worktree](https://github.com/agenttools/worktree) |
| **Language** | (See repo) |
| **Install** | See repo |
| **Focus** | GitHub issue → worktree → Claude Code pipeline |

**Overview.** This tool ties git worktrees to GitHub issues, creating a structured pipeline: fetch an issue, create a worktree, generate a CLAUDE.md with project context, and launch one or more Claude Code workers with specialized roles.

**Key Features:**

- **GitHub issue integration:** `worktree open 42 "fix-login-bug"` validates the issue, creates a worktree with a new branch, and fetches issue details via `gh`
- **Auto-generated CLAUDE.md:** Per-worktree context document with project info
- **Multi-worker coordination:** `worktree open 78 "refactor-api" -w 3` spawns 3 Claude instances with coordination
- **Coding agent archetypes:** Assign specialized roles to workers:
  - 🏗️ The Architect — System design & architecture patterns
  - 🔍 The Detective — Debugging, edge cases & security
  - 🛠️ The Craftsman — Code quality & best practices
  - 🚀 The Explorer — Innovation & alternative approaches
  - 🎨 The Aesthete — Elegant solutions & simplicity
- **Coordination document:** Workers communicate through `WORKTREE_COORDINATION.md` (auto-added to `.gitignore`)
- **Progress monitoring:** Optional overseer worker that tracks progress
- **`.worktree.yml` config:** Project-specific settings for session names, Claude context, and commands

**Limitations.** Tightly coupled to GitHub issues and `gh` CLI. The archetype system is creative but adds cognitive overhead. Coordination via markdown document is inherently racy.

---

#### worktree-manager-skill

| | |
|---|---|
| **Repository** | [github.com/Wirasm/worktree-manager-skill](https://github.com/Wirasm/worktree-manager-skill) |
| **Language** | Claude Code Skill (Markdown + JSON config) |
| **Install** | Clone into `~/.claude/skills/worktree-manager` |
| **Focus** | Teaching Claude Code to manage worktrees natively |

**Overview.** Unlike every other tool in this survey, worktree-manager-skill is not a standalone program. It's a Claude Code "skill" — a set of instructions and configuration that Claude Code reads to learn how to manage worktrees on your behalf. This means the orchestration happens through conversation with Claude rather than through a separate CLI or TUI.

**Key Features:**

- **Terminal-agnostic:** Supports Ghostty, iTerm2, tmux, WezTerm, Kitty, Alacritty
- **Port pool management:** Allocates ports from a configurable range (default 8100-8199) for dev servers
- **Global worktree registry:** Tracks all worktrees across all projects for status and cleanup
- **Configurable via `config.json`:** Shell, Claude command, port pool, worktree base directory
- **Zero external tooling:** Just Claude Code and git

**Limitations.** Relies entirely on Claude Code's ability to follow skill instructions reliably. No TUI or status monitoring. The skill approach means no persistent process watching over sessions.

---

### 5. Lightweight Scripts and Gists

These are minimal, copy-paste-ready implementations of the core pattern.

---

#### andynu's tmx-claude / tmx-worktree

| | |
|---|---|
| **Source** | [gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a](https://gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a) |
| **Language** | Shell scripts |
| **Focus** | "Mission control" for 100+ projects |

**Overview.** A lightweight pattern designed for developers managing large numbers of projects from a central directory. The key insight articulated in this gist: "a single Claude session has one context. Spawning secondary sessions gives each task its own isolated context and working directory."

**Key Scripts:**

- **`tmx-claude`:** Spawn a Claude session in any directory without creating a branch. Handles both inside-tmux (creates window) and outside-tmux (creates session) scenarios.
- **`tmx-worktree`:** Create a worktree + branch + launch Claude in one command. Supports branching from non-main branches.
- **`worktree-clean`:** Cleanup with safety checks for uncommitted/unpushed work. Supports `--merged` to clean all merged worktrees across projects.
- **Integration with "beads"** task management system (Steve Yegge's project).

**Convention:** Worktrees live at `~/work/src/<project>-<branch>/`.

---

#### worksfornow's /delegate slash command

| | |
|---|---|
| **Source** | [gist on GitHub](https://gist.github.com/worksfornow/df0cb6c4f6fd4966cd17133bc711fd20) |
| **Blog Post** | [worksfornow.pika.page](https://worksfornow.pika.page/posts/note-to-a-friend-how-i-run-claude-code-agents-in-parallel) |
| **Language** | Claude Code slash command |
| **Focus** | Task-master integration for parallel delegation |

**Overview.** A Claude Code slash command that spawns parallel agents based on task IDs from claude-task-master (a CLI tool for structuring tasks with markdown). Usage: `/delegate 1 3 5` (or `/task-master 1 3 5`) to spin up agents for task IDs 1, 3, and 5.

**Mechanism:** For each task ID, it creates a tmux session named after the task, cd's into the worktree, and launches Claude with restricted tools (`Edit,Write,Bash,MultiEdit`).

---

### 6. First-Party: Claude Code Agent Teams

| | |
|---|---|
| **Documentation** | [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams) |
| **Enable** | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` in settings.json or environment |
| **Status** | Experimental; disabled by default |

**Overview.** Anthropic has productized the community pattern directly into Claude Code. Agent teams coordinate multiple Claude Code instances working together, with a lead session coordinating work, assigning tasks, and synthesizing results. Teammates work independently in their own context windows and communicate through inter-agent messaging.

**Key Features:**

- **Team lead model:** One session acts as coordinator; teammates work independently
- **Split-pane mode via tmux/iTerm2:** Each teammate gets its own pane; default is "auto" (split panes if already in tmux, in-process otherwise)
- **Plan-before-implement:** Require teammates to work in read-only plan mode until the lead approves their approach
- **Direct teammate interaction:** Unlike subagents, you can interact with individual teammates directly
- **CLAUDE.md support:** Teammates read CLAUDE.md files from their working directory

**Known Limitations (from the docs and GitHub issues):**

- No session resumption: `/resume` and `/rewind` do not restore in-process teammates
- Task status can lag: teammates sometimes fail to mark tasks as completed
- Shutdown can be slow: teammates finish their current request before shutting down
- One team per session
- Split-pane mode not supported in VS Code's integrated terminal, Windows Terminal, or Ghostty
- A recently-filed issue (`anthropics/claude-code#23615`) reports that agents spawn by splitting the current pane rather than creating new windows, breaking layouts when multiple agents start simultaneously

---

## In-Category Comparison Tables

### TUI Session Managers

| Feature | claude-squad | ccmanager | agent-deck | agent-of-empires |
|---|:---:|:---:|:---:|:---:|
| **Language** | Go | TypeScript | Go (Bubble Tea) | Rust |
| **Requires tmux** | ✅ | ❌ | ✅ | ✅ |
| **Agent count** | 5 | 8+ | 5+ | 5 |
| **Session forking** | ❌ | ❌ | ✅ | ❌ |
| **MCP management** | ❌ | ❌ | ✅ (with socket pooling) | ❌ |
| **Docker sandboxing** | ❌ | ❌ | ❌ | ✅ |
| **Devcontainer support** | ❌ | ✅ | ❌ | ❌ |
| **Auto-approval (AI)** | ❌ | ✅ (Haiku-based) | ❌ | ❌ |
| **Session data copying** | ❌ | ✅ | ✅ (via fork) | ❌ |
| **Multi-project** | ❌ | ✅ | ❌ | ❌ |
| **State hooks** | ❌ | ✅ | ❌ | ❌ |
| **Worktree hooks** | ❌ | ✅ | ❌ | ❌ |
| **Tmux status bar** | ❌ | ❌ | ✅ | ❌ |
| **WSL support** | ❓ | ❓ | ✅ | ❓ |
| **Install method** | curl script | npx | installer script | brew / curl / cargo |
| **Auto-yes mode** | ✅ | ✅ (AI-based) | ❌ | ❌ |
| **Diff view** | ❌ | ❌ | ❌ | ✅ |

### Opinionated Workflow Tools

| Feature | workmux | dmux | barrel | forestui | maw |
|---|:---:|:---:|:---:|:---:|:---:|
| **Language** | TypeScript | TypeScript | (See repo) | Python | Shell |
| **tmux model** | Windows in existing session | Session per project | Configurable layout | Named windows | Split panes |
| **Agent status display** | ✅ (icons in window list) | ❌ | ❌ | ✅ (session tracking) | ❌ |
| **Merge + cleanup** | ✅ (`workmux merge`) | ✅ (smart merge) | ❌ | ❌ | ❌ |
| **Config file** | `.workmux.yaml` | ❌ | `barrel.yaml` | Settings JSON | `agents.yaml` |
| **File symlinks** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Lifecycle hooks** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Monorepo support** | ✅ (nested configs) | ❌ | ❌ | ✅ (multi-forest) | ❌ |
| **Agent portability** | ❌ | ❌ | ✅ (core feature) | ❌ | ❌ |
| **Inter-agent messaging** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Slash command** | ✅ (`/worktree`) | ❌ | ❌ | ❌ | ❌ |
| **AI branch naming** | ❌ | ✅ (OpenRouter) | ❌ | ❌ | ❌ |
| **WezTerm support** | ✅ (experimental) | ❌ | ❌ | ❌ | ❌ |

---

## Cross-Category Comparison Matrix

This matrix compares all tools across the most commonly needed capabilities.

| Tool | Category | Tmux Required | Worktrees | Multi-Agent | Lifecycle Mgmt | Status Monitoring | Sandboxing | Language |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|---|
| claude-squad | TUI Manager | ✅ | ✅ | 5 agents | Basic | Basic | ❌ | Go |
| ccmanager | TUI Manager | ❌ | ✅ | 8+ agents | Hooks | Hooks-based | ✅ (Devcontainer) | TypeScript |
| agent-deck | TUI Manager | ✅ | ✅ | 5+ agents | Basic | ✅ (smart) | ❌ | Go |
| agent-of-empires | TUI Manager | ✅ | ✅ | 5 agents | Basic | ✅ | Docker | Rust |
| workmux | Workflow | ✅ | ✅ | Any CLI | Full (merge+cleanup) | ✅ (status icons) | ❌ | TypeScript |
| dmux | Workflow | ✅ | ✅ | 2 agents | Full (smart merge) | Basic | ❌ | TypeScript |
| barrel | Workflow | ✅ | ✅ | 4+ agents | Session only | ❌ | ❌ | — |
| forestui | Workflow | ✅ | ✅ | Claude only | Session tracking | ✅ | ❌ | Python |
| maw | Workflow | ✅ | ✅ | Any CLI | Basic | ❌ | ❌ | Shell |
| claude-tmux | Tmux | ✅ | ✅ | Claude only | Session + PR | ✅ (live preview) | ❌ | Rust |
| vscode-ext | Tmux | ✅ | ✅ | Any | Basic | ❌ | ❌ | TypeScript |
| agenttools/worktree | Worktree | ✅ | ✅ | Claude (multi-worker) | Issue-based | Overseer | ❌ | — |
| worktree-manager-skill | Worktree | ❌ | ✅ | Claude only | Conversational | ❌ | ❌ | Skill |
| andynu scripts | Scripts | ✅ | ✅ | Claude only | Manual | ❌ | ❌ | Shell |
| /delegate | Scripts | ✅ | ✅ | Claude only | Task-based | ❌ | ❌ | Slash cmd |
| Agent Teams | First-party | Optional | ❌* | Claude only | Built-in | Built-in | ❌ | — |

*Agent Teams uses tmux for split-pane mode but does not use git worktrees for isolation — teammates work in their own context windows but share the filesystem.

---

## Architecture and Design Analysis

### The Core Pattern

Every tool in this survey implements some variation of the same fundamental pattern:

```
┌─────────────────────────────────────────────┐
│                 tmux session                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  │ Window 1  │ │ Window 2  │ │ Window 3  │   │
│  │ feature-a │ │ bugfix-b  │ │ review-c  │   │
│  │           │ │           │ │           │   │
│  │ ┌──────┐ │ │ ┌──────┐ │ │ ┌──────┐ │   │
│  │ │claude│ │ │ │claude│ │ │ │claude│ │   │
│  │ └──────┘ │ │ └──────┘ │ │ └──────┘ │   │
│  │ ┌──────┐ │ │ ┌──────┐ │ │ ┌──────┐ │   │
│  │ │ shell│ │ │ │ shell│ │ │ │ shell│ │   │
│  │ └──────┘ │ │ └──────┘ │ │ └──────┘ │   │
│  └──────────┘ └──────────┘ └──────────┘    │
│       ↓              ↓             ↓        │
│  worktree/       worktree/     worktree/    │
│  feature-a       bugfix-b      review-c     │
└─────────────────────────────────────────────┘
```

The variation comes in three key design decisions:

**1. Session topology: windows vs. sessions.** workmux puts each worktree in a tmux *window* within your existing session (context switching = tab switching). claude-squad and agent-deck create separate tmux *sessions* per agent (more isolated but requires a TUI to navigate). dmux creates a tmux *session per project* with worktrees as *panes* within that session. Each choice trades off between integration with existing workflows and isolation between tasks.

**2. Lifecycle scope.** Some tools manage only the "create session" phase (claude-squad, agent-of-empires), while others manage the full lifecycle through merge and cleanup (workmux, dmux). The lightweight scripts don't manage lifecycle at all — they're fire-and-forget launchers.

**3. Agent awareness.** The most sophisticated tools (agent-deck, workmux, ccmanager) can detect what state an agent is in — working, waiting for input, done — and surface this information in the tmux chrome. This is implemented by monitoring the terminal output for patterns specific to each agent (e.g., Claude Code's prompt patterns). agent-deck calls this "AI-specific intelligence on top of tmux."

### The Containerization Gap

Running AI coding agents with broad filesystem and shell access is inherently risky, yet most tools in this survey provide no isolation beyond git worktrees (which only isolate the filesystem view of the repo, not the agent's ability to execute arbitrary commands). Only three approaches in the ecosystem address this:

| Approach | Tool | Isolation Granularity | What's Containerized | What's on the Host |
|---|---|---|---|---|
| **Per-session Docker** | agent-of-empires | Individual agent sessions | Agent + its worktree | TUI manager, other sessions |
| **Devcontainer** | ccmanager | Development environment | Agent sessions + dependencies | ccmanager TUI, notifications |
| **Full-environment** | Snapp's workflow (DIY) | Everything | tmux, agents, editors, worktrees | SSH client only |

The per-session model (agent-of-empires) provides the finest-grained isolation — one container per agent — making it ideal for running `--dangerously-skip-permissions` with less actual danger. The devcontainer model (ccmanager) is coarser but more familiar to teams already using devcontainers, and it preserves host-level automation (notifications, hooks) that would be lost inside a container. The full-environment model provides the strongest isolation but requires the most setup and means your entire development workflow lives inside Docker.

This remains one of the most significant gaps in the ecosystem. As agents gain more autonomy (auto-approval, background execution, `--dangerously-skip-permissions`), the argument for sandboxing grows stronger. Anthropic's own Claude Code devcontainer exists but is separate from the agent teams feature, leaving the integration to the user.

### The Worktree Convention Question

One of the most interesting design divergences is where worktrees get created:

| Convention | Used By | Example |
|---|---|---|
| Sibling to project root | workmux, agent-deck (default), claude-squad | `project__worktrees/feature-a/` |
| Under repo in `.worktrees/` | agent-deck (option), agenttools/worktree | `project/.worktrees/feature-a/` |
| Global base directory | worktree-manager-skill | `~/tmp/worktrees/feature-a/` |
| Next to `main/` dir | dmux | `project/main/`, `project/feature-a/` |
| In `~/work/src/<project>-<branch>/` | andynu scripts | `~/work/src/myapp-feature-auth/` |
| Under `agents/` directory | maw | `repo/agents/1/`, `repo/agents/2/` |

The richsnapp.com article advocates strongly for the sibling-directory convention with a `default/` directory as the main worktree, arguing that bare repo approaches create "various gotchas" and the `default/` structure is simpler.

---

## Community Commentary and Discussion

### Hacker News

**"Worktrees and Tmux and Claude, Oh My Zsh" (Dec 2025)** — [news.ycombinator.com/item?id=46316943](https://news.ycombinator.com/item?id=46316943)

The author (Rich Snapp) wrote this because "I often heard about people using git worktrees to work with multiple agents but I never saw someone document exactly how they do it." The discussion validated the pattern as production-ready. Key themes: the importance of tmux's terminal bell support for notifications when agents finish, the value of Docker containers for sandboxing, and the debate over bare repos vs. default-directory worktree layouts.

**"Forestui: A tmux-powered worktree manager for Claude Code" (Feb 2026)** — [news.ycombinator.com/item?id=46864999](https://news.ycombinator.com/item?id=46864999)

Recent thread from the past week. forestui's author describes it as a TUI that "orchestrates your worktrees, editors, terminals, and Claude Code sessions across organized tmux windows."

**"Claude Code's Hidden Multi-Agent System" (Feb 2026)** — Covered on [paddo.dev/blog/claude-code-hidden-swarm/](https://paddo.dev/blog/claude-code-hidden-swarm/)

This post documents the discovery of TeammateTool inside Claude Code's binary before it was officially released, and contextualizes it within the ecosystem of community tools: "Before TeammateTool existed, developers built their own multi-agent solutions: claude-flow for swarm orchestration, ccswarm for git worktree isolation, oh-my-claudecode with five execution modes. The community proved the demand. Anthropic absorbed the pattern." The article also references a Hacker News commenter who described a nine-agent system with a Manager, Architect, Dev pair using TDD, and a CAB (Change Advisory Board) agent as quality gatekeeper.

### Claude Code GitHub Issues

**`anthropics/claude-code#23615` — "Agent teams should spawn in new tmux window, not split current pane"** — [github.com/anthropics/claude-code/issues/23615](https://github.com/anthropics/claude-code/issues/23615)

Filed just days ago (Feb 2026), this issue captures a fundamental tension in the agent teams implementation: agents spawn by splitting the current pane, which "breaks the user's existing layout and causes command corruption when multiple agents start simultaneously." The proposed fix is to create a new tmux window for the team instead. The author notes the workaround is to "dispatch fewer agents (2 instead of 4), or handle crashed agents' tasks in the main session." This issue is tagged with `platform:macos` and `has repro`.

### awesome-claude-code

The [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) repository tracks resources in this space. Tools like claude-tmux have been submitted and validated through its issue process.

---

## Blog Posts and Long-Form Writeups

### "Worktrees & Tmux & Claude, Oh My... Zsh" — Rich Snapp (Dec 2025)

**URL:** [richsnapp.com/article/2025/12-14-git-worktrees-tmux-claude-code-oh-my-zsh](https://www.richsnapp.com/article/2025/12-14-git-worktrees-tmux-claude-code-oh-my-zsh)

The most comprehensive single writeup of a complete workflow. Snapp describes a Docker container-based approach where tmux provides the persistent session layer, worktrees provide filesystem isolation, and a custom `gwtmux` script ties everything together. Key insights:

> "Just as git worktrees give you multiple workspaces in the codebase within which to work, tmux gives you multiple workspaces for the agents, shells, IDE instances, and various tools that you will be using during development."

> "tmux also has the ability to listen to terminal bells and highlight the appropriate tab/window when an agent has finished a task or is asking you questions. This is an important feature for being truly efficient with your concurrently running agents."

> "In Claude Code, sending notifications as terminal bells needs to be configured in `/config`."

He advocates for the `default/` directory convention for worktrees: "The directory structure often proposed for worktrees is using a bare repo and working around the various gotchas that come up with using a bare repo in a way it was not intended. I've found using a simple default directory structure with a regular git repo is a lot easier to work with."

### "workmux: git worktrees + tmux for parallel AI agents" — Raine Virta (Dec 2025)

**URL:** [raine.dev/blog/introduction-to-workmux/](https://raine.dev/blog/introduction-to-workmux/)

The design philosophy behind workmux. Virta articulates why this is the right level of abstraction: "While tools like Vibe Kanban and Claude Squad offer alternative UIs, workmux fits naturally into existing tmux workflows." He describes the evolution from curiosity about worktrees to a concrete tool:

> "Recently, with AI-assisted coding gaining momentum, a long-standing and not that well-known git feature is finding new relevance. The feature is git worktrees... Running multiple agents in parallel is becoming increasingly common, and agents benefit from the isolation worktrees provide."

> "After a few weeks of dogfooding, workmux has changed my standard git workflow. Now, instead of plain branches, I tend to go for a dedicated worktree for each task, be it a feature or reviewing someone else's code."

### "Ruby on Rails, Claude Code and Worktrees" — Hans Schnedlitz (Jul 2025)

**URL:** [hansschnedlitz.com/writing/2025/07/10/rails-claude-code-and-worktrees](https://www.hansschnedlitz.com/writing/2025/07/10/rails-claude-code-and-worktrees)

A practical Rails-specific workflow using tmuxinator for worktree review sessions. Schnedlitz uses a local `.tmuxinator` file per project to define review environments that automatically install dependencies, start the dev server (with port collision avoidance), and open an editor. The approach is notably low-tech compared to the TUI managers — just tmuxinator + a shell script for creating worktrees with Claude.

### "Note to a Friend — How I Run Claude Code Agents in Parallel" — WorksForNow

**URL:** [worksfornow.pika.page](https://worksfornow.pika.page/posts/note-to-a-friend-how-i-run-claude-code-agents-in-parallel)

The blog post behind the `/delegate` slash command. Framed as an actual message to a friend, it's refreshingly honest about the experience:

> "Context switching can be a lot. My current workflow is to spin up the agents, let them run, and then open the worktrees in an IDE to review changes."

> "If you're on the Pro plan, you'll likely hit usage limits quickly. You can upgrade to Max ($100/month), or have a second Pro account ready to switch over."

> "With the standard IDE<>Claude workflow, you're a micromanager, confined to one session. With this setup, you get to delegate and let work happen in parallel."

### "Practical Parallelism With Claude Code" — claydon.co (May 2025)

**URL:** [claydon.co/code/practical-parallelism-with-claude-code](https://claydon.co/code/practical-parallelism-with-claude-code/)

One of the earliest writeups of the pattern, predating most of the tools above. Describes an "ensemble" approach where Claude Opus generates three alternative implementations for the same task, each in its own worktree, and you pick the best one. Also demonstrates the simpler "divide and conquer" pattern of assigning different tasks to different agents.

---

## Recommendations by Use Case

### "I live in tmux and want this to feel native"

**→ workmux.** It maps worktrees to tmux windows and shows agent status in your window list. No separate TUI to manage — it's just tmux, enhanced. The `/worktree` slash command for Claude Code is a nice bonus for delegation from within a session. raine's ecosystem of related tools (tmux-file-picker, git-surgeon, etc.) adds further value if you're building out a complete tmux-based development environment.

### "I want a proper mission control TUI for many concurrent sessions"

**→ agent-deck** if you want the most polished TUI with session forking, MCP management, and tmux status bar integration. **→ ccmanager** if you need the broadest agent support (8+ agents), devcontainer sandboxing, or AI-powered auto-approval. **→ claude-squad** if you want the most battle-tested option with the largest community.

### "I need Docker/container isolation for security"

There are three tiers of containerization in this ecosystem, each drawing the isolation boundary at a different level:

**Per-session sandboxing → agent-of-empires.** Each agent session can run in its own Docker container with `aoe add --sandbox .`, giving you the most granular isolation. A misbehaving agent can't damage your host filesystem or affect other sessions. Shared auth credentials are passed through so the agent can still access APIs.

**Devcontainer-based environment isolation → ccmanager.** Rather than wrapping individual sessions, ccmanager can run agent sessions inside VS Code-style devcontainers while keeping the manager itself on the host machine. This enables sandboxed development environments with restricted network access while maintaining host-level notifications and automation. It's a coarser level of isolation than per-session sandboxing, but aligns well with teams that already use devcontainers for development and want to add `--dangerously-skip-permissions` inside a controlled environment.

**Full-environment containerization → build it yourself.** Rich Snapp's workflow (documented in the "Worktrees & Tmux & Claude, Oh My... Zsh" article) runs everything — tmux, worktrees, agents, editors — inside a persistent Docker container. A startup script provisions the container, and tmux inside it provides session persistence across reconnections. This is the most comprehensive approach, but it's a hand-rolled workflow rather than a feature of any particular tool.

Most of the other tools in this survey do not address sandboxing at all, which is a notable gap given the security implications of running AI agents with broad filesystem and shell access.

### "I want agent portability across Claude Code, Codex, OpenCode, etc."

**→ barrel.** Its core feature is writing agent definitions once and symlinking them into each tool's expected location. If you're regularly switching between LLM providers or want to compare outputs, this is the tool designed for that.

### "I want to tie agent work to GitHub issues"

**→ agenttools/worktree.** It creates a direct pipeline from GitHub issues to Claude Code sessions with auto-generated context. The archetype system for multi-worker setups is unique in this space.

### "I just want a simple script I can understand and modify"

**→ andynu's tmx-claude / tmx-worktree gist.** Shell scripts you can read in five minutes, understand completely, and customize to your workflow. The convention of worktrees at `~/work/src/<project>-<branch>/` is clean and predictable.

### "I use VS Code but want tmux persistence"

**→ vscode-ext-tmux-worktree.** It bridges the two worlds: VS Code's sidebar shows your worktrees, and tmux sessions persist in the background.

### "I want to try Anthropic's official approach"

**→ Claude Code Agent Teams.** Set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` and give it a task complex enough to warrant a team. Be aware of the known limitations around session resumption and the tmux pane-splitting issue (`#23615`). It's the least mature option but the only one with first-party support and development resources behind it.

---

## Appendix: Quick Reference Links

| Tool | Repository |
|---|---|
| claude-squad | [github.com/smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad) |
| ccmanager | [github.com/kbwo/ccmanager](https://github.com/kbwo/ccmanager) |
| agent-deck | [github.com/asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck) |
| agent-of-empires | [github.com/njbrake/agent-of-empires](https://github.com/njbrake/agent-of-empires) |
| workmux | [github.com/raine/workmux](https://github.com/raine/workmux) |
| dmux | [github.com/formkit/dmux](https://github.com/formkit/dmux) |
| barrel | [github.com/txtx/barrel](https://github.com/txtx/barrel) |
| forestui | [github.com/flipbit03/forestui](https://github.com/flipbit03/forestui) |
| multi-agent-workflow-kit | [github.com/laris-co/multi-agent-workflow-kit](https://github.com/laris-co/multi-agent-workflow-kit) |
| claude-tmux | [github.com/nielsgroen/claude-tmux](https://github.com/nielsgroen/claude-tmux) |
| vscode-ext-tmux-worktree | [github.com/kargnas/vscode-ext-tmux-worktree](https://github.com/kargnas/vscode-ext-tmux-worktree) |
| agenttools/worktree | [github.com/agenttools/worktree](https://github.com/agenttools/worktree) |
| worktree-manager-skill | [github.com/Wirasm/worktree-manager-skill](https://github.com/Wirasm/worktree-manager-skill) |
| andynu's scripts | [gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a](https://gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a) |
| /delegate command | [gist on GitHub](https://gist.github.com/worksfornow/df0cb6c4f6fd4966cd17133bc711fd20) |
| Claude Code Agent Teams | [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams) |
| awesome-claude-code | [github.com/hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) |
| awesome-opencode | [github.com/awesome-opencode/awesome-opencode](https://github.com/awesome-opencode/awesome-opencode) |
