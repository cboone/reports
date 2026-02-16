---
created: 2026-02-15
updated: 2026-02-15
---

# LLM coding agent monitoring, session management, and notification tools

_February 15, 2026_

This document catalogs every notable tool available (as of mid-February 2026) for monitoring LLM coding agent activity, managing sessions, receiving notifications, and tracking usage/costs. Tools are grouped by category, with links, platform support, and key features.

---

## Table of Contents

1. [Desktop Session Managers (macOS)](#1-desktop-session-managers-macos)
2. [Terminal TUI Session Managers](#2-terminal-tui-session-managers)
3. [Desktop Apps for Parallel Agent Workflows](#3-desktop-apps-for-parallel-agent-workflows)
4. [Mobile & Remote Agent Clients](#4-mobile--remote-agent-clients)
5. [Notification Tools & Hooks](#5-notification-tools--hooks)
6. [Usage & Cost Monitoring CLIs](#6-usage--cost-monitoring-clis)
7. [IDE Extensions for Agent Visibility](#7-ide-extensions-for-agent-visibility)
8. [LLM Observability Platforms (Production / SaaS)](#8-llm-observability-platforms-production--saas)
9. [Agent SDKs & Developer Observability](#9-agent-sdks--developer-observability)
10. [Comparison Tables](#10-comparison-tables)

---

## 1. Desktop Session Managers (macOS)

These are native or near-native desktop apps that give you a floating/dockable window to see all running coding agent sessions at a glance.

### Agent Manager X

- **GitHub:** [maddada/agent-manager-x](https://github.com/maddada/agent-manager-x)
- **Platform:** macOS only (native Swift/SwiftUI)
- **Install:** `brew install --cask maddada/tap/agent-manager-x-swift`
- **Agents:** Claude Code, Codex, OpenCode
- **Key Features:**
  - Real-time status detection (Thinking, Processing, Waiting, Idle)
  - Mini floating display mode with spinning indicators
  - Global hotkey toggle (configurable, default Ctrl+Space)
  - CPU and RAM usage per session — kill stale sessions with one click
  - Voice (TTS) and bell audio notifications on completion
  - Click on session to jump to its terminal/editor
  - Groups sessions per project
- **Notes:** Built as native Swift app for performance. Inspired by Agent Sessions (ozankasikci).

### Agent Sessions (ozankasikci)

- **GitHub:** [ozankasikci/agent-sessions](https://github.com/ozankasikci/agent-sessions)
- **HN Discussion:** [news.ycombinator.com/item?id=46126741](https://news.ycombinator.com/item?id=46126741) (Dec 2025)
- **Platform:** macOS only (iTerm2 and Terminal.app)
- **Install:** `brew tap ozankasikci/tap && brew install --cask agent-sessions`
- **Tech:** Tauri 2.x + React + TypeScript (Rust backend)
- **Agents:** Claude Code, OpenCode
- **Key Features:**
  - View all active sessions in one place
  - Real-time status detection (Thinking, Processing, Waiting, Idle)
  - Global hotkey to toggle visibility
  - Click to focus on a specific session's terminal
  - Custom session names and quick-access URLs

### Agent Sessions (jazzyalex) — Session Browser & Analytics

- **GitHub:** [jazzyalex/agent-sessions](https://github.com/jazzyalex/agent-sessions)
- **Platform:** macOS native app
- **Install:** `brew tap jazzyalex/agent-sessions && brew install --cask agent-sessions`
- **Agents:** Codex CLI, Claude Code, OpenCode, Gemini CLI, Factory Droid, GitHub Copilot CLI
- **Key Features:**
  - Unified session browser across ALL supported agents
  - Full-text search across all past sessions
  - Rate limits tracker in real-time
  - Agents analytics dashboard
  - Filter by folder/repo, resume sessions instantly
  - Read-only, local-only, zero telemetry
  - Image browser for inline thumbnails
  - Supports deleted session visibility (OpenClaw)

---

## 2. Terminal TUI Session Managers

Terminal-based tools that wrap tmux (or equivalent) to provide a unified dashboard for managing multiple coding agent sessions.

### Agent Deck

- **GitHub:** [asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck)
- **Website:** [asheshgoplani.github.io/agent-deck](https://asheshgoplani.github.io/agent-deck/)
- **Platform:** macOS, Linux, Windows (WSL)
- **Install:** `curl -fsSL https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/install.sh | bash` or `brew install asheshgoplani/tap/agent-deck`
- **Tech:** Go + Bubble Tea TUI framework, built on tmux
- **Agents:** Claude Code (+ fork support), Gemini CLI, OpenCode, Codex, Aider, any shell
- **Key Features:**
  - Smart status detection (running, waiting, idle, error)
  - Fuzzy search across all sessions
  - Session forking (Claude Code conversations)
  - On-demand MCP server attachment per session or globally
  - MCP Socket Pool — reduces MCP memory usage by 85-90%
  - Git worktree support for parallel agents
  - Conductors — persistent Claude sessions that orchestrate/monitor other sessions
  - Telegram & Slack bridges for remote monitoring
  - Tmux status bar integration showing waiting sessions
  - Groups, profiles, configurable hotkeys
- **Notes:** One of the most feature-rich terminal-based session managers. The "conductor" feature is unique — lets a Claude Code instance supervise other sessions.

### Agent of Empires (AoE)

- **GitHub:** [njbrake/agent-of-empires](https://github.com/njbrake/agent-of-empires)
- **Platform:** macOS, Linux
- **Install:** `curl -fsSL https://raw.githubusercontent.com/njbrake/agent-of-empires/main/scripts/install.sh | bash` or `brew install njbrake/aoe/aoe`
- **Tech:** Rust, built on tmux
- **Agents:** Claude Code, OpenCode, Mistral Vibe, Codex CLI, Gemini CLI
- **Key Features:**
  - TUI dashboard to create, monitor, and manage sessions
  - Toggle between agent and paired shell terminal views
  - Status detection (running, waiting, idle)
  - Git worktrees for parallel branches
  - Docker sandboxing with shared auth volumes
  - Diff view — review git changes without leaving TUI
  - Per-repo config via `.aoe/config.toml`
  - Profiles for separate workspaces
- **Notes:** Inspired by Agent Deck. Rust-native for performance. Docker sandboxing is a differentiator.

### Claude Squad

- **GitHub:** [smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad)
- **Website:** [smtg-ai.github.io/claude-squad](https://smtg-ai.github.io/claude-squad/)
- **Platform:** macOS, Linux
- **Agents:** Claude Code, Aider, Codex, OpenCode, Amp
- **Key Features:**
  - Manages multiple AI terminal agents in separate workspaces
  - Git worktrees for codebase isolation
  - Session-per-branch model
- **Notes:** One of the earlier tools in this space; well-known in the community.

### CCManager

- **GitHub:** [kbwo/ccmanager](https://github.com/kbwo/ccmanager)
- **Platform:** macOS, Linux
- **Agents:** Claude Code, Gemini CLI, Codex CLI, Cursor Agent, Copilot CLI, Cline CLI, OpenCode, Kimi CLI
- **Key Features:**
  - Self-contained — no tmux dependency
  - Visual status indicators (busy, waiting, idle)
  - Multi-project support from a single interface
  - Copy Claude Code session data between worktrees (context transfer)
  - Auto-approval feature using Claude Haiku to analyze prompts
  - Worktree hooks for post-creation automation
  - Auto-generated worktree directory paths
  - Devcontainer support for sandboxed environments
- **Notes:** Supports the widest range of agents. The auto-approval and context-transfer features are unique.

### Agent Viewer

- **GitHub:** [hallucinogen/agent-viewer](https://github.com/hallucinogen/agent-viewer)
- **Platform:** macOS, Linux (web UI accessible from any device)
- **Key Features:**
  - Kanban board for managing Claude Code agents in tmux
  - Spawn, monitor, interact with agents from a single web UI
  - Access from mobile phone via Tailscale
  - Auto-discovery of existing tmux Claude sessions
  - Live output with ANSI color rendering
  - Send messages and upload files to agent cards
  - Columns: Running, Idle, Completed
- **Notes:** Web-based UI is a differentiator — great for remote/mobile access.

### WezTerm Agent Deck (Plugin)

- **GitHub:** [Eric162/wezterm-agent-deck](https://github.com/Eric162/wezterm-agent-deck)
- **Platform:** Any platform running WezTerm
- **Agents:** Claude Code, OpenCode, Gemini, Codex, Aider
- **Key Features:**
  - WezTerm plugin — shows status dots in tabs
  - Notifications when agents need attention
- **Notes:** Lightweight integration for WezTerm users. No separate app needed.

---

## 3. Desktop Apps for Parallel Agent Workflows

Full desktop applications focused on running multiple agent sessions with visual management, worktree isolation, and diff review.

### Crystal

- **GitHub:** [stravu/crystal](https://github.com/stravu/crystal)
- **Website:** [stravu.com/crystal](https://stravu.com/crystal)
- **Platform:** macOS (Homebrew: `brew install --cask stravu-crystal`), Linux (AppImage), Windows
- **Tech:** Electron + React + TypeScript
- **Agents:** Claude Code, Codex
- **Key Features:**
  - Run multiple agent sessions in parallel git worktrees
  - Each iteration auto-commits for easy rollback
  - Built-in diff view and manual edit capability
  - Squash commits and merge to main branch
  - Status tracking with visual indicators
  - Prompt history and session templates
  - Sub-agent output visualization
  - Terminal/Logs separation with filter and search
  - Auto model selection for Claude Code
  - Desktop notifications when sessions need input
  - Multiple agents per worktree support
  - Nimbalyst integration
- **Notes:** Self-described as the first "IVE" (Integrated Vibe Environment). Very actively developed with frequent releases.

### Verdent Deck

- **Website:** [verdent.ai](https://verdent.ai) / [claudelog.com/verdent-ai](https://claudelog.com/verdent-ai/)
- **Platform:** macOS (Apple M-series)
- **Pricing:** Starts at $19/month
- **Key Features:**
  - Multi-agent deployment and monitoring in isolated git worktrees
  - DiffLens for code diffs, timelines, and causal flows
  - Plan-first alignment with verification
  - Asynchronous execution — delegate tasks and walk away
  - Comprehensive dashboards and task tracking
  - Also has a VS Code extension variant
- **Notes:** Commercial product. The only paid standalone desktop app in this space. Claims highest SWE-bench Verified results among production agents.

---

## 4. Mobile & Remote Agent Clients

Tools for monitoring and interacting with agents from your phone or remotely.

### Happy Coder

- **GitHub:** [slopus/happy](https://github.com/slopus/happy)
- **Website:** [happy.engineering](https://happy.engineering/)
- **App Store:** [iOS](https://apps.apple.com/us/app/happy-codex-claude-code-app/id6748571505) / [Google Play](https://play.google.com/store/apps/details?id=com.ex3ndr.happy)
- **Platform:** iOS, Android, Web, macOS desktop
- **Install CLI:** `npm install -g happy-coder`
- **Agents:** Claude Code, Codex
- **Key Features:**
  - Continue agent sessions seamlessly between desktop and mobile
  - Push notifications for permission requests, task completion, errors
  - Spawn and control multiple agents in parallel
  - Voice coding with voice-to-action
  - End-to-end encryption (TweetNaCl, same as Signal)
  - Zero-knowledge architecture
  - Access conversation history offline
  - File mentions, slash commands, custom agents on mobile
  - Open source (MIT)
- **Notes:** The leading mobile companion for coding agents. Genuinely useful for monitoring long-running tasks on the go. Free and open source.

### Agent Viewer (via Tailscale)

- See [Agent Viewer](#agent-viewer) above — its web UI is accessible from mobile via Tailscale.

### Agent Deck (via Telegram/Slack bridges)

- See [Agent Deck](#agent-deck) above — conductor sessions can be monitored and controlled via Telegram bots and Slack slash commands.

---

## 5. Notification Tools & Hooks

Lightweight tools and configurations specifically for getting desktop or system notifications when agents complete tasks or need input.

### Claude Code Built-in Hooks

- **Docs:** [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)
- **Platform:** macOS, Linux, Windows
- **Features:**
  - Native hook system with events: `Notification`, `Stop`, `PreToolUse`, `PostToolUse`, `SessionStart`, `SessionEnd`
  - Notification event fires when Claude is waiting for input/permission
  - Configure via `~/.claude/settings.json` or `/hooks` interactive menu
  - macOS: `osascript -e 'display notification...'`
  - Linux: `notify-send`
  - Windows: PowerShell MessageBox
- **Notes:** First-party solution. Zero dependencies. Should be your starting point.

### CCNotify

- **GitHub:** [dazuiba/CCNotify](https://github.com/dazuiba/CCNotify)
- **Platform:** macOS only
- **Features:**
  - Desktop notifications via terminal-notifier
  - Task duration tracking (started time, how long it took)
  - Click notification to jump back to project in VS Code
  - Session data stored locally in SQLite
  - Activity logging to `~/.claude/ccnotify/`

### code-notify

- **GitHub:** [mylee04/code-notify](https://github.com/mylee04/code-notify)
- **Platform:** macOS, Linux, Windows
- **Install (macOS):** `brew tap mylee04/tools && brew install code-notify && cn on`
- **Agents:** Claude Code, Codex, Gemini
- **Features:**
  - Cross-platform native notifications
  - Sound notifications with custom sounds
  - Voice announcements (macOS, Windows)
  - Tool-specific messages ("Claude completed the task")
  - Project-specific settings
  - Quick aliases (`cn`, `cnp`)
  - Configurable alert types (idle_prompt, permission_prompt)

### ai-agents-notifier (claude-code-notifier)

- **GitHub:** [hta218/ai-agents-notifier](https://github.com/hta218/claude-code-notifier)
- **Platform:** macOS, Linux, Windows
- **Agents:** Claude Code, GitHub Copilot CLI
- **Features:**
  - Cross-platform notification system
  - Session start/end, completion, and permission notifications
  - Shell script-based, minimal setup

### terminal-notifier (macOS utility)

- **Guide:** [andreagrandi.it/posts/using-terminal-notifier-claude-code-custom-notifications](https://www.andreagrandi.it/posts/using-terminal-notifier-claude-code-custom-notifications/)
- **Install:** `brew install terminal-notifier`
- **Notes:** General-purpose macOS CLI for sending notifications to Notification Center. Used as a building block by many of the above tools.

### OSC Escape Sequences (for remote/SSH workflows)

- **Guide:** [kane.mx/posts/2025/claude-code-notification-hooks](https://kane.mx/posts/2025/claude-code-notification-hooks/)
- **Features:**
  - OSC 777 (VSCode, rxvt-unicode) and OSC 9 (iTerm2, Windows Terminal) format notifications
  - Works over SSH tunnels — remote EC2 agent sends notification to local desktop
  - Multi-window UUID-based terminal mapping
  - Task description extraction for context-rich notifications
- **Notes:** Best solution for remote development environments.

---

## 6. Usage & Cost Monitoring CLIs

Terminal tools for tracking token consumption, costs, and billing windows for coding agent subscriptions.

### ccusage

- **GitHub:** [ryoppippi/ccusage](https://github.com/ryoppippi/ccusage)
- **Website:** [ccusage.com](https://ccusage.com/)
- **Platform:** macOS, Linux, Windows (CLI)
- **Features:**
  - Analyzes Claude Code usage from local JSONL files
  - 5-hour billing block tracking with active block monitoring
  - Statusline integration for Claude Code status bar hooks
  - Model tracking and per-model cost breakdown
  - Date filtering (`--since`, `--until`)
  - Beautiful colorful table output
  - Smart compact mode for narrow terminals
  - Custom Claude data directory support

### Claude Code Usage Monitor

- **GitHub:** [Maciek-roboblog/Claude-Code-Usage-Monitor](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor)
- **Platform:** Terminal (macOS, Linux)
- **Features:**
  - Real-time token usage monitoring
  - Burn rate calculation
  - Predictions for token depletion
  - Visual progress bars
  - Session-aware analytics
  - Multiple subscription plan support

### simple-usage-monitor

- **GitHub:** [SrivathsanSivakumar/simple-usage-monitor](https://github.com/SrivathsanSivakumar/simple-usage-monitor)
- **Platform:** Terminal
- **Features:**
  - Real-time token and cost estimator
  - Lives in your terminal alongside Claude Code

### claude-code-usage

- **GitHub:** [evanlong-me/claude-code-usage](https://github.com/evanlong-me/claude-code-usage)
- **Platform:** Terminal
- **Features:**
  - Track costs and token consumption
  - Analyze spending locally

---

## 7. IDE Extensions for Agent Visibility

### Agent Lens (VS Code)

- **GitHub:** [23min/agent-lens](https://github.com/23min/agent-lens)
- **Platform:** VS Code
- **Agents:** GitHub Copilot, Claude Code
- **Features:**
  - Interactive DAG visualization of agents, skills, and handoff connections
  - Token usage, model distribution, agent activity at a glance
  - Session timeline replay with agent switches and model changes
  - Claude Code: cache read/creation tokens, cache hit ratio, input token breakdown
  - Sidebar browser for agents and skills
  - Filter by provider (Copilot, Claude, or both)
  - All data stays local

### VS Code Agent Sessions (Built-in)

- **Docs:** [code.visualstudio.com/docs/copilot/agents/overview](https://code.visualstudio.com/docs/copilot/agents/overview)
- **Blog:** [VS Code Multi-Agent Development (Feb 2026)](https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development)
- **Platform:** VS Code 1.109+
- **Agents:** GitHub Copilot, Claude (Anthropic), Codex (OpenAI)
- **Features:**
  - Unified sidebar view for ALL agent sessions (local, cloud, background)
  - Real-time status tracking
  - Chat editor tabs for monitoring/course-correcting agents mid-run
  - Delegate tasks between agent types
  - Compact and side-by-side view modes
  - Session filtering, archiving, and PR checkout
- **Notes:** First-party VS Code feature. As of February 2026, supports Claude and Codex alongside Copilot under a single Copilot Pro+ subscription.

### GitHub Agent HQ

- **Docs:** [github.com blog / Agent HQ announcement](https://www.helpnetsecurity.com/2026/02/05/github-enables-coding-agents/)
- **Platform:** GitHub.com, GitHub Mobile, VS Code
- **Agents:** GitHub Copilot, Claude, Codex
- **Features:**
  - Assign agent sessions directly to issues and PRs
  - Mission Control dashboard for all active sessions
  - Live session logs with mid-run steering
  - Multi-agent comparison on same task
  - Mobile access via GitHub Mobile
- **Notes:** Announced February 2026. Requires Copilot Pro+ ($39/mo) or Enterprise.

---

## 8. LLM Observability Platforms (Production / SaaS)

Enterprise-grade platforms for monitoring LLM applications in production. These focus on API-level observability, tracing, evaluation, and cost management — not on individual coding agent sessions.

### Langfuse (Open Source)

- **GitHub:** [langfuse/langfuse](https://github.com/langfuse/langfuse)
- **Website:** [langfuse.com](https://langfuse.com)
- **Pricing:** Free (self-hosted), Cloud from $29/mo
- **Key Strengths:** Open source, self-hostable, strong community, prompt versioning, session-based analysis, human annotation queues
- **Best For:** Teams wanting full data control with self-hosting

### LangSmith

- **Website:** [smith.langchain.com](https://smith.langchain.com)
- **Key Strengths:** Deep LangChain integration, virtually zero performance overhead, excellent tracing
- **Best For:** Teams building with LangChain/LangGraph

### LangWatch

- **Website:** [langwatch.ai](https://langwatch.ai)
- **Key Strengths:** Full monitoring + evaluation + experimentation in one platform, fast setup (5 min)
- **Best For:** Teams wanting comprehensive all-in-one solution

### Braintrust

- **Website:** [braintrust.dev](https://www.braintrust.dev)
- **Key Strengths:** CI/CD integration for evals, dataset versioning, webhook alerts, strong cost attribution
- **Best For:** Teams focused on continuous quality and evaluation

### Helicone (Open Source)

- **Website:** [helicone.ai](https://helicone.ai)
- **Pricing:** Free tier (10K requests/mo), Pro from $79/mo
- **Key Strengths:** Proxy-based (just change base URL), zero code changes, fast setup
- **Best For:** Quick visibility with minimal integration effort

### Maxim AI

- **Website:** [getmaxim.ai](https://www.getmaxim.ai)
- **Key Strengths:** End-to-end platform (simulation → evaluation → observability), AI-powered simulations, flexible evals, real-time alerts (Slack/PagerDuty/OpsGenie)
- **Best For:** Enterprise teams needing comprehensive agent quality management

### Arize AI (Phoenix)

- **Website:** [arize.com](https://arize.com)
- **Key Strengths:** Advanced drift detection for embeddings/LLM outputs, strong ML monitoring pedigree, enterprise-grade
- **Best For:** Enterprises with existing ML infrastructure

### Datadog LLM Observability

- **Website:** [datadoghq.com/product/llm-observability](https://www.datadoghq.com/product/llm-observability/)
- **Key Strengths:** End-to-end infrastructure monitoring with LLM layer, enterprise features, broad integrations
- **Best For:** Teams already using Datadog for infrastructure

### Grafana (with OpenTelemetry)

- **Website:** [grafana.com](https://grafana.com)
- **Key Strengths:** Open source visualization, integrates with Prometheus/OTEL/Datadog, alert routing
- **Best For:** Teams with existing Grafana/Prometheus infrastructure wanting LLM dashboards

### Other Notable Platforms

| Platform | Focus | Notes |
|----------|-------|-------|
| **Lunary** | Prompt management + monitoring | Self-hostable, user-friendly |
| **Agenta** | Prompt experimentation | Finding best prompt×model combos |
| **TruLens** (Snowflake) | Evaluation + tracing | Open-source library, acquired by Snowflake |
| **Evidently AI** | Testing + monitoring | Built on open-source Evidently library |

---

## 9. Agent SDKs & Developer Observability

Python/TypeScript SDKs for instrumenting your own agents with monitoring and observability.

### AgentOps

- **GitHub:** [AgentOps-AI/agentops](https://github.com/AgentOps-AI/agentops)
- **Website:** [agentops.ai](https://www.agentops.ai/)
- **Docs:** [docs.agentops.ai](https://docs.agentops.ai/v1/introduction)
- **Platform:** Python SDK, TypeScript SDK, self-hostable dashboard (open source, MIT)
- **Integrations:** CrewAI, Agno, OpenAI Agents SDK, LangChain, AutoGen/AG2, CamelAI, LlamaIndex, Llama Stack, SwarmZero, Google ADK, LiteLLM, Haystack
- **Key Features:**
  - 2-line integration for full observability
  - Session replay with waterfall visualization
  - LLM call tracing (prompts, completions, tokens, costs)
  - Multi-agent interaction tracking
  - Tool usage statistics
  - Failure detection and error tracking
  - Cost control and analytics
  - Self-hostable via Docker (ClickHouse + OTEL collector backend)
- **Notes:** One of the most popular agent-focused observability SDKs. Open source dashboard was a recent addition.

---

## 10. Comparison Tables

### Session Managers — Quick Comparison

| Tool | Platform | Interface | Agents Supported | Worktrees | Notifications | Install |
|------|----------|-----------|-----------------|-----------|---------------|---------|
| **Agent Deck** | macOS/Linux/WSL | Terminal TUI | Claude, Gemini, OpenCode, Codex, Aider | ✅ | Telegram, Slack | `curl` / Homebrew |
| **Agent of Empires** | macOS/Linux | Terminal TUI | Claude, OpenCode, Mistral, Codex, Gemini | ✅ | — | `curl` / Homebrew |
| **Claude Squad** | macOS/Linux | Terminal TUI | Claude, Aider, Codex, OpenCode, Amp | ✅ | — | Homebrew |
| **CCManager** | macOS/Linux | Terminal TUI | Claude, Gemini, Codex, Cursor, Copilot, Cline, OpenCode, Kimi | ✅ | — | Cargo |
| **Agent Manager X** | macOS | Desktop (Swift) | Claude, Codex, OpenCode | — | Voice, Bell | Homebrew |
| **Agent Sessions (ozan)** | macOS | Desktop (Tauri) | Claude, OpenCode | — | — | Homebrew |
| **Agent Sessions (jazzy)** | macOS | Desktop (native) | Claude, Codex, Gemini, Copilot, Droid, OpenCode | — | — | Homebrew |
| **Agent Viewer** | Any (web) | Web Kanban | Claude Code (tmux) | — | — | npm |
| **Crystal** | macOS/Linux/Win | Desktop (Electron) | Claude, Codex | ✅ | Desktop | Homebrew / DMG |
| **Happy Coder** | iOS/Android/Web | Mobile + CLI | Claude, Codex | — | Push | npm |

### Notification Tools — Quick Comparison

| Tool | Platform | Method | Agents | Unique Feature |
|------|----------|--------|--------|----------------|
| **Claude Code Hooks** | All | Native OS | Claude Code | Built-in, zero-dep |
| **CCNotify** | macOS | terminal-notifier | Claude Code | Duration tracking, VS Code jump |
| **code-notify** | All | Native OS | Claude, Codex, Gemini | Voice announcements |
| **ai-agents-notifier** | All | Shell scripts | Claude, Copilot CLI | Minimal setup |
| **Happy Coder** | iOS/Android | Push | Claude, Codex | Full mobile client |
| **Agent Deck conductors** | All | Telegram/Slack | All supported | AI-supervised monitoring |

### LLM Observability Platforms — Quick Comparison

| Platform | Open Source | Self-Host | Proxy-Based | Alerts | Evals | Starting Price |
|----------|-----------|-----------|-------------|--------|-------|----------------|
| **Langfuse** | ✅ | ✅ | — | ✅ | ✅ | Free / $29/mo |
| **LangSmith** | — | — | — | ✅ | ✅ | Free tier |
| **LangWatch** | — | — | — | ✅ | ✅ | Free tier |
| **Helicone** | ✅ | ✅ | ✅ | ✅ | — | Free / $79/mo |
| **Braintrust** | — | — | — | ✅ | ✅ | Free tier |
| **Maxim AI** | — | ✅ (VPC) | — | ✅ | ✅ | Contact |
| **AgentOps** | ✅ | ✅ | — | ✅ | ✅ | Free tier |
| **Arize Phoenix** | ✅ | ✅ | — | ✅ | ✅ | Free / Enterprise |
| **Datadog** | — | — | — | ✅ | ✅ | Usage-based |

---

## Key Discussions & Community Resources

- **Hacker News — Agent Sessions:** [news.ycombinator.com/item?id=46126741](https://news.ycombinator.com/item?id=46126741) — "I built a macOS app to monitor all my Claude Code sessions at once"
- **Awesome Claude Code:** [github.com/hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — Curated list of skills, hooks, orchestrators, and apps
- **Git Worktrees for AI Agents (comprehensive guide):** [Upsun Developer Center](https://devcenter.upsun.com/posts/git-worktrees-for-parallel-ai-coding-agents/) — Compares agentree, git-worktree-runner, worktree-cli, gwq, ccswarm, Crystal
- **VS Code Multi-Agent Development:** [VS Code blog (Feb 2026)](https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development)
- **GitHub Agent HQ Announcement:** [helpnetsecurity.com](https://www.helpnetsecurity.com/2026/02/05/github-enables-coding-agents/)
- **Claude Code Hooks Guide (official):** [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)
- **AIMultiple — 15 AI Agent Observability Tools (benchmark):** [research.aimultiple.com/agentic-monitoring](https://research.aimultiple.com/agentic-monitoring/)

---

## Summary & Recommendations

**For individual developers on macOS wanting to monitor multiple coding agents:**
- Start with **Agent Deck** (terminal TUI, most feature-rich) or **Crystal** (desktop GUI) depending on whether you prefer terminal or graphical workflows.
- Add **Happy Coder** if you want mobile push notifications.
- Use **ccusage** for cost tracking.

**For teams running production LLM applications:**
- **Langfuse** (open source, self-hosted) or **LangSmith** (if using LangChain) for observability.
- **AgentOps** for agent-specific monitoring with broad framework support.
- **Datadog** or **Grafana** if you already have infrastructure monitoring in place.

**For simple notifications (just tell me when Claude is done):**
- Claude Code's built-in hooks are sufficient for most cases. Takes 2 minutes to configure.
- **code-notify** if you want voice announcements and cross-agent support.

**The ecosystem is evolving rapidly.** Most of these tools appeared in mid-to-late 2025 and are being actively developed. VS Code's Agent Sessions and GitHub's Agent HQ (both February 2026) suggest that agent monitoring is becoming a first-party concern for platform vendors, which may consolidate some of this fragmented tooling over time.

---

*Last updated: February 15, 2026*
