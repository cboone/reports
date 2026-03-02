---
created: 2026-02-10
---

# LLM coding agents for the terminal in early 2026

_February 10, 2026_

**The CLI/TUI AI coding tool landscape has expanded from a handful of experiments to roughly 90 tools with recent activity in under two years.** The category barely existed before mid-2024; today, many major AI labs ship a terminal coding agent, and model-agnostic open-source alternatives have attracted tens of thousands of GitHub stars. Three tools appear most frequently in practitioner discussions, Claude Code, Aider, and OpenAI's Codex CLI, but a vibrant ecosystem of specialized, multi-agent, and local-first tools has emerged around them. MCP (Model Context Protocol) has become a widely adopted extensibility standard, local model support via Ollama is common, and multi-agent orchestration is a fast-growing sub-category.

This document catalogs a broad set of CLI and TUI coding tools that use large language models to assist with software development, organized into logical categories with comparison tables and landscape analysis.

---

## Full-featured agentic coding assistants

These tools can autonomously read codebases, edit files across multiple directories, execute shell commands, manage git workflows, and perform multi-step software engineering tasks with minimal human intervention. They represent the core of the CLI coding agent movement.

### Claude Code (Anthropic)

An agentic coding tool that lives in your terminal, understands your codebase, and codes through natural language commands. **~65,000 GitHub stars at time of writing.** Anthropic has publicly said a large share of its own code is written with the tool.

- **GitHub:** github.com/anthropics/claude-code
- **Models:** Anthropic models natively — Claude Opus 4.6, Sonnet 4.5, Haiku 4.5. Enterprise routing via Amazon Bedrock or Google Vertex AI. Community proxy support (OpenRouter, LiteLLM)
- **Features:** File reading/editing, shell execution, git integration (commits, PRs, branches), sub-agents (Plan/Explore/Task), CLAUDE.md project instructions, MCP support, context compaction, web search, background agents, GitHub Actions integration, SDK for headless automation
- **Interface:** CLI with TUI-like interactive session; also VS Code/JetBrains extensions
- **Language:** TypeScript (distributed as compiled binary)
- **License:** Proprietary (repo is for plugins/issue tracking)
- **Agent type:** Multi-agent (built-in sub-agents)
- **Status:** Extremely active, multiple releases per week
- **Distinguishing:** Often cited for strong reasoning on complex refactors and architectural changes. Plan mode enables deliberate multi-step work. Locked to Anthropic models natively. Community discussions often frame it as an "escalation path when other tools fail." Frequently praised in HN and Reddit discussions. Main downside is cost ($15+ per complex session).

### Aider

The original open-source AI pair programming tool for the terminal. **~40,500 GitHub stars**, 4.1M+ installs, and vendor-reported billions of tokens processed weekly.

- **GitHub:** github.com/Aider-AI/aider
- **Models:** Very broad model coverage — OpenAI, Anthropic, Google Gemini, DeepSeek, Ollama, LM Studio, OpenRouter, Azure, GitHub Copilot API, and any OpenAI-compatible endpoint
- **Features:** Git-native workflow (auto-commits with sensible messages), repository map via tree-sitter, multi-file editing, architect mode (plan then code), voice input, image/URL support, linter/test integration, multiple edit formats, scripting/automation
- **Interface:** CLI (interactive chat)
- **Language:** Python
- **License:** Apache 2.0
- **Agent type:** Single-agent with architect+editor dual-model mode
- **Status:** Very active, 230+ contributors
- **Distinguishing:** One of the broadest model support sets in this category. Git-first philosophy means AI changes are usually captured as clean commits. Strong SWE-Bench results in published materials. Battle-tested and mature. Repo map feature gives strong codebase understanding. Steep learning curve but powerful once mastered.

### OpenAI Codex CLI

Lightweight coding agent from OpenAI with built-in sandboxed execution. **~50,500 GitHub stars.** Rewritten from TypeScript to Rust for performance.

- **GitHub:** github.com/openai/codex
- **Models:** Primarily OpenAI (via Responses API); community fork (open-codex) adds Gemini, OpenRouter, Ollama
- **Features:** Three approval modes (suggest/auto-edit/full-auto), sandboxed execution (network-disabled, directory-confined), MCP support, AGENTS.md rules, multimodal (screenshots/diagrams), code review agent, GitHub Action, SDK, non-interactive mode for CI/CD
- **Interface:** CLI; also macOS app and IDE extensions
- **Language:** Rust (codex-rs)
- **License:** Apache 2.0
- **Agent type:** Single-agent with code review sub-agent
- **Status:** Very active, $1M open-source grant initiative
- **Distinguishing:** **A strong sandboxing/security model** among CLI agents, including network isolation in full-auto mode. Rust performance. Works with ChatGPT subscription. HN consensus: better for greenfield solo projects and parallel agents; weaker on documentation tasks where it can hallucinate.

### OpenCode (formerly SST project)

Model-agnostic terminal-first AI coding agent supporting **75+ LLM providers**. Originally launched by the SST team and now maintained under the Anomaly organization.

- **GitHub:** github.com/anomalyco/opencode
- **Models:** 75+ providers via Models.dev — Anthropic, OpenAI, Google, AWS Bedrock, Ollama, plus authentication via existing Copilot/ChatGPT subscriptions
- **Features:** LSP auto-configuration, multi-session parallel agents, session sharing via links, client/server architecture, privacy-first (no code storage), vim-like editor, SQLite storage, MCP support
- **Interface:** TUI (Bubble Tea, Neovim-inspired)
- **Language:** Go/TypeScript
- **License:** Apache 2.0
- **Agent type:** Multi-session parallel agents
- **Status:** Rapidly growing, very active
- **Distinguishing:** Can reuse existing GitHub Copilot or ChatGPT Plus subscriptions for authentication, a notable cost advantage. One of the broadest LLM provider catalogs in this category. Community opinion is divided, praised for flexibility but often considered less polished than Aider.

### Gemini CLI (Google)

Google's open-source terminal AI agent with **1M token context** and a comparatively generous free tier.

- **GitHub:** github.com/google-gemini/gemini-cli
- **Models:** Google Gemini models; **free tier: 60 requests/min, 1,000/day**
- **Features:** 1M token context, multimodal, built-in Google Search grounding, shell commands, web fetching, MCP extensibility, conversation checkpointing, three auth tiers
- **Interface:** CLI
- **Language:** TypeScript
- **License:** Apache 2.0
- **Agent type:** Single-agent
- **Status:** Very active
- **Distinguishing:** **A generous free tier** among major CLI agents. Web search grounding means the agent can verify its own answers against Google Search. Privacy concern: Google uses prompts and code for model training. ~27,000 stars.

### Crush (Charmbracelet)

The "glamorous" agentic coding TUI from the Charm ecosystem. Originally created as "OpenCode" by Kujtim Shala before he joined Charm.

- **GitHub:** github.com/charmbracelet/crush
- **Models:** OpenAI, Anthropic, Gemini, Groq, OpenRouter, AWS Bedrock, Azure, Vertex AI, local models via Ollama/LM Studio
- **Features:** Mid-session model switching preserving context, LSP integration, MCP extensibility, session-based architecture, Agent Skills standard, granular tool permissions, customizable commit attribution, Catwalk model database
- **Interface:** TUI (split-pane view, diff viewer, keyboard navigation)
- **Language:** Go
- **License:** Charm License (proprietary — not open source despite appearances)
- **Agent type:** Single-agent
- **Status:** Very active, ~12,000–19,000 stars
- **Distinguishing:** **Widely regarded as highly polished for terminal UX.** Broad platform support including Android, FreeBSD, OpenBSD, NetBSD. Part of the Charm ecosystem (Bubble Tea, Lip Gloss, Glow). Not truly open-source — uses Charm's custom license.

### Goose (Block)

Open-source, extensible AI agent from Block (Square/Cash App) that goes beyond coding. **~20,000–30,000 GitHub stars.**

- **GitHub:** github.com/block/goose
- **Models:** Any LLM with tool calling — multiple model configs simultaneously, BYOK
- **Features:** MCP-native (co-developed with Anthropic), recipes (reusable workflows), code mode, desktop app + CLI, multi-model cost optimization, extensible via MCP servers
- **Interface:** CLI + desktop app (Electron) + VS Code extension
- **Language:** Rust (core), TypeScript (UI)
- **License:** Apache 2.0
- **Agent type:** Multi-agent (sub-agents, parallel sessions)
- **Status:** Very active, 296 contributors
- **Distinguishing:** One of the earliest MCP-native agents. Goes beyond coding — extensible for non-engineering workflows. Enterprise credibility from Block backing. Recipes system for reusable task automation.

### GitHub Copilot CLI

GitHub's terminal agent with native repository, issue, and PR integration. Public preview since September 2025.

- **GitHub:** github.com/github/copilot-cli
- **Models:** Claude Sonnet 4.5 (default), Claude Sonnet 4, GPT-5, GPT-5.1-Codex, Grok Code Fast 1
- **Features:** Native GitHub integration (reference issues, browse PRs, manage repos), MCP support, model selection, session persistence, trust/permission model, skills system, interactive and programmatic modes
- **Interface:** CLI
- **Language:** Closed-source binary
- **License:** Proprietary (requires Copilot subscription)
- **Stars:** ~5,700
- **Agent type:** Single-agent (subagents on roadmap)
- **Status:** Active, public preview
- **Distinguishing:** **One of the few CLI agents with deep native GitHub integration** — reference issues, PRs, and repos through natural conversation. No separate API billing; it uses an existing Copilot subscription. Multi-model access.

### Cursor CLI

AI-powered code editor that added a CLI component for terminal-based agentic coding.

- **Homepage:** cursor.com/cli
- **Models:** GPT-5, GPT-5.2, Claude Opus 4.1/4.6, Claude Sonnet 4/4.5, Grok
- **Features:** Plan mode, Ask mode, file editing with diffs, shell execution, sub-agents (research, terminal, parallel work), MCP support, rules system, cloud agents
- **Interface:** IDE (primary) + CLI tool
- **Language:** TypeScript (closed-source)
- **License:** Proprietary (subscription required)
- **Agent type:** Multi-agent
- **Status:** Very active
- **Distinguishing:** Powerful IDE + CLI combination. Cloud agents push work to cloud for background processing. CLI is a newer addition — IDE remains primary focus.

### Amp (Sourcegraph)

Agentic coding tool from Sourcegraph with unconstrained token usage and team collaboration features.

- **Homepage:** ampcode.com
- **Models:** Claude Opus (smart mode), GPT-5.2-Codex (deep mode), Claude Haiku 4.5 (rush mode), Grok, Gemini — auto-selects a model per task
- **Features:** Three agent modes (smart/rush/deep), Librarian sub-agent (cross-repo search), Oracle sub-agent (code review/architecture), Thread Map visualization, thread sharing/leaderboards, skills system, MCP support
- **Interface:** CLI + VS Code extension
- **Language:** TypeScript (closed-source)
- **License:** Proprietary (ad-supported free tier up to $10/day, then pay-as-you-go)
- **Agent type:** Multi-agent (Librarian, Oracle, Smart)
- **Status:** Active, research preview
- **Distinguishing:** **Ad-supported free tier** is a novel business model. Thread Map visualizes how coding conversations connect over time. Librarian reads code across any GitHub repo. Automatic model selection rather than user choice.

### Plandex

Terminal-based AI coding engine designed for large, complex, multi-file projects. Written in Go.

- **GitHub:** github.com/plandex-ai/plandex
- **Models:** Anthropic Claude (default), OpenAI, Gemini, open-source via OpenRouter; supports Claude Pro/Max subscriptions
- **Features:** **2M token context window** with tree-sitter project maps, cumulative diff sandbox (changes separate from project files), full auto + step-by-step control, REPL mode with fuzzy auto-complete, built-in version control with branches, multi-provider model mixing
- **Interface:** CLI + TUI (REPL mode)
- **Language:** Go
- **License:** MIT (originally AGPL-3.0)
- **Stars:** ~14,700
- **Agent type:** Single-agent with multi-step planning
- **Status:** Self-hosted mode active; cloud service winding down
- **Distinguishing:** **Very large context handling** (2M tokens). Diff sandbox keeps AI changes separate until approved. Plan versioning with branches. Single Go binary with no dependencies.

### Continue

Open-source AI code assistant with CLI, TUI, headless modes, and CI/CD agent workflows.

- **GitHub:** github.com/continuedev/continue
- **Models:** Any model — OpenAI, Anthropic, Gemini, Ollama, LM Studio, Azure, and more
- **Features:** CLI with TUI mode and headless mode, VS Code and JetBrains extensions, autocomplete, chat, code editing, CI/CD agent checks on PRs, workflows triggered on events, context providers
- **Interface:** CLI (TUI + headless) + IDE extensions
- **Language:** TypeScript
- **License:** Apache 2.0
- **Stars:** ~22,000
- **Agent type:** Multi-agent (CI/CD workflow agents)
- **Status:** Very active, recently pivoted to emphasize CLI
- **Distinguishing:** Recently expanded from IDE-only to CLI + CI/CD agents. Can run agents as automated CI checks on pull requests. Solid local model support.

### Droid (Factory AI)

Enterprise-grade terminal coding agent with specialized sub-agents. **Vendor-reported leading Terminal-Bench result (58.75%).**

- **Homepage:** factory.ai
- **Models:** BYOK — Claude Opus, Sonnet, GPT-5, any frontier model, local via Ollama
- **Features:** Specialized Droids (Code, Knowledge, Reliability for incident triage, Product for PM work), headless exec mode for CI/CD, spec mode for auto-planning, Jira/Notion/Slack/GitHub integrations, web browsing
- **Interface:** TUI + headless exec CLI
- **Language:** Closed-source
- **License:** Proprietary (commercial)
- **Agent type:** Multi-agent (specialized Droids)
- **Status:** Active
- **Distinguishing:** **Vendor-reported leading Terminal-Bench performer.** Unique architecture, not a single generalist but a system of domain-specific Droids. Enterprise-focused with incident response and PM capabilities.

### Kiro CLI (AWS)

AWS's spec-driven development AI coding tool, successor to Amazon Q Developer CLI.

- **Homepage:** kiro.dev
- **Models:** Claude Sonnet 4.5, "Auto" mode (frontier model mix with intent detection)
- **Features:** Spec-driven development (EARS-notation requirements → architecture → implementation plan), property-based testing, checkpoint/rewind, agent hooks, "Powers" plugin ecosystem (Figma, Supabase, Stripe integrations), autonomous multi-day agent
- **Interface:** CLI + IDE (Code OSS-based)
- **Language:** Closed-source
- **License:** Proprietary (free during preview)
- **Agent type:** Multi-agent (autonomous agent can dispatch sub-tasks)
- **Status:** Active, preview phase
- **Distinguishing:** **One of the few CLI tools with spec-driven development** — generates formal requirements before writing code. Autonomous agent can work independently for days. Successor to Amazon Q Developer CLI (which is now deprecated).

### Rovo Dev CLI (Atlassian)

Enterprise CLI agent with native Jira, Confluence, and Bitbucket integration. Atlassian has reported a high SWE-bench full leaderboard result (41.98%).

- **Homepage:** atlassian.com/blog/announcements/rovo-dev-command-line-interface
- **Models:** Frontier models (details not fully public)
- **Features:** Jira/Confluence/Bitbucket integration, enterprise security, Teamwork Graph for organizational context
- **Interface:** CLI
- **License:** Proprietary (enterprise)
- **Agent type:** Single-agent
- **Status:** Active
- **Distinguishing:** **One of the few CLI agents with native Atlassian integration.** Teamwork Graph provides organizational context beyond just code.

### Qwen Code CLI

Open-source CLI coding agent optimized for Qwen3-Coder, forked from Gemini CLI. **1,000 free requests/day.**

- **GitHub:** github.com/QwenLM/qwen-code
- **Models:** Qwen3-Coder (480B MoE, 256K-1M context), any OpenAI-compatible API
- **Features:** Enhanced parser for Qwen models, vision mode, MCP support, plan mode, sub-agents, headless mode, IDE integration
- **Interface:** CLI (interactive + headless)
- **Language:** TypeScript
- **License:** Apache 2.0
- **Status:** Active (launched July 2025)
- **Distinguishing:** One of the stronger free tiers from a major lab (1,000 req/day). Open-source model weights. Qwen3-Coder-480B posts competitive coding benchmark results against frontier models.

### Kimi Code CLI (Moonshot AI)

Terminal agent with a distinctive dual shell/agent mode and Agent Swarm capability.

- **GitHub:** github.com/MoonshotAI/kimi-cli
- **Models:** Kimi K2/K2.5 (1T parameter MoE, 256K context)
- **Features:** Ctrl-X switches between shell mode and agent mode, ACP support for Zed/Neovim/Emacs, Zsh plugin, MCP support, Agent Swarm for parallel sub-agents (up to 100)
- **Interface:** CLI + TUI (dual mode)
- **Language:** Python
- **License:** Modified MIT
- **Status:** Technical preview
- **Distinguishing:** **Distinctive dual-mode shell** — seamlessly switch between regular terminal and AI agent. Agent Swarm coordinates up to 100 parallel sub-agents.

### Additional full-featured agents

| Tool | GitHub | Stars | Language | License | Models | Key differentiator |
|------|--------|-------|----------|---------|--------|--------------------|
| **Codel** | Referenced in awesome-agents | ~2,000 | Go | Open | Multiple | Fully autonomous with terminal, browser, and editor |
| **o1-engineer** | github.com/Doriandarko/o1-engineer | ~3,000 | Python | Open | OpenAI o1 | CLI focused on o1 reasoning model capabilities |
| **Claude Engineer** | Referenced in lists | ~2,000 | Python | Open | Anthropic | Interactive CLI leveraging Claude for development |
| **gptme** | github.com/gptme/gptme | ~4,000 | Python | MIT | Multiple | "Your agent in your terminal" — writes code, browses web, vision |
| **Nanocoder** | github.com/Nano-Collective/nanocoder | ~1,000 | TypeScript | Open | Local/OpenRouter | Local-first CLI coding agent for local models |
| **VT Code** | github.com/vinhnx/vtcode | ~310 | Rust | Open | Multiple | Semantic code understanding via tree-sitter and ast-grep |
| **SHAI** | github.com/ovh/shai | ~517 | Rust | Open | Multiple | OVH's pair programming buddy, written in Rust |
| **Codai** | github.com/meysamhadeli/codai | ~374 | Go | Open | Multiple | AI coding agent for terminal |
| **MyCoder** | github.com/drivecore/mycoder | ~500 | TypeScript | Open | Multiple | Simple CLI-based AI agent system |
| **CodeBuff** | github.com/CodebuffAI/codebuff | ~400 | TypeScript | Open | Multiple | Multi-agent AI coding assistant with SDK |
| **Grok CLI** | github.com/superagent-ai/grok-cli | ~500 | TypeScript | Open | xAI Grok, OpenAI-compat | Cheapest model ($0.20/1M tokens), 92 tok/sec |
| **Groq Code CLI** | github.com/build-with-groq/groq-code-cli | ~500 | TypeScript | Open | Groq-hosted models | Extensible AI-powered coding CLI |
| **OCode** | github.com/haasonsaas/ocode | ~200 | TypeScript | Open | Multiple | Deep codebase intelligence |
| **TunaCode** | github.com/alchemiststudiosDOTai/tunacode | ~200 | TypeScript | Open | Multiple | Safe git branches, multi-LLM |
| **Terra Code** | github.com/TerraAGI/terra-code-cli | ~200 | TypeScript | Open | Multiple | Persistent memory |
| **Mistral Vibe** | github.com/mistralai/mistral-vibe | New | Python | Open | Mistral | Mistral's own CLI coding assistant |
| **Every CODE** | github.com/just-every/code | ~200 | TypeScript | Open | Multiple | Fork of Codex CLI with multi-agent orchestration |
| **GenAIcode** | github.com/gtanczyk/genaicode | ~300 | TypeScript | Open | Multiple | Multi-modal coding assistant, CLI or GUI |
| **Gemini Engineer** | github.com/ozanunal0/gemini-engineer | ~200 | Python | Open | Gemini | CLI code generation via Gemini API |
| **p90** | github.com/Ichigo-Labs/p90-cli | ~100 | TypeScript | Open | Multiple | Minimal CLI coding agent for quick prototypes |
| **Cliq** | github.com/kpritam/cliq | ~100 | TypeScript | Open | Multiple | Functional Effect-TS-based coding CLI |
| **Codexa** | github.com/mikeoller82/codexa | ~200 | Python | Open | Multiple | AI coding assistant and development partner |
| **Augment CLI** | augmentcode.com | N/A | Closed | Proprietary | Frontier models | Enterprise context engine, MongoDB/Spotify customer |

---

## Code-aware chat and REPL tools

These tools provide interactive Q&A with code context but are oriented more toward conversation and query than autonomous file editing. They sit between chat interfaces and full agents.

### Open Interpreter

Natural language interface for your computer — lets LLMs run code locally with minimal restrictions in local mode. **~62,000 GitHub stars** — one of the higher-starred tools in this category.

- **GitHub:** github.com/openinterpreter/open-interpreter
- **Models:** OpenAI (default), Claude, all LiteLLM providers, Ollama, LM Studio
- **Features:** Executes Python/JS/Shell locally, file manipulation, data analysis, image generation, GUI control, vision, profile system
- **Interface:** CLI (interactive chat)
- **Language:** Python
- **License:** AGPL-3.0
- **Distinguishing:** Essentially a local, unrestricted Code Interpreter. In typical local mode, there are no hard file-size or timeout caps, and internet use is broadly available. Can control your computer's GUI.

### AIChat

All-in-one LLM CLI featuring shell assistant, chat REPL, RAG, AI tools, and agents. **Written in Rust** for performance.

- **GitHub:** github.com/sigoden/aichat
- **Models:** **20+ providers natively** — OpenAI, Claude, Gemini, Ollama, Groq, Azure, Bedrock, Mistral, Deepseek, Grok, Cohere, Perplexity, and more
- **Features:** Shell assistant, Chat-REPL with tab completion, RAG support, AI agents, built-in HTTP server with LLM playground/arena, shell integration for all major shells
- **Interface:** CLI + interactive REPL
- **Language:** Rust
- **License:** MIT/Apache 2.0 (dual)
- **Stars:** ~6,000
- **Distinguishing:** Very comprehensive multi-provider support in a single binary. Built-in LLM arena for model comparison. RAG support built in. Single Rust binary.

### LLM (Simon Willison)

The Swiss-army knife for LLMs on the command line. **~11,100 GitHub stars.**

- **GitHub:** github.com/simonw/llm
- **Models:** OpenAI (default), 100+ via plugins — Anthropic, Gemini, Ollama, Mistral, Cohere, and many more
- **Features:** Plugin architecture, conversation logging to SQLite, embeddings, tool use (v0.26+), multimodal, templates, system prompts, pipe-friendly, --functions for inline Python tools
- **Interface:** CLI (with interactive chat mode)
- **Language:** Python
- **License:** Apache 2.0
- **Distinguishing:** One of the biggest plugin ecosystems (100+ plugins). Unix philosophy — composable, pipe-friendly. SQLite-backed conversation storage. Created by prominent open-source developer Simon Willison. Tool use support makes it increasingly agent-capable.

### Elia

Snappy, keyboard-centric TUI for interacting with LLMs. Beautiful Textual-based interface.

- **GitHub:** github.com/darrenburns/elia
- **Models:** OpenAI, Anthropic, Gemini, Groq, Ollama, any LiteLLM provider
- **Features:** Full-screen TUI, vim-like keybindings, SQLite conversation storage, inline mode, visual select for copying code
- **Interface:** TUI
- **Language:** Python (Textual framework)
- **License:** Apache 2.0
- **Stars:** ~2,300
- **Distinguishing:** Purpose-built beautiful TUI for LLM chat. Keyboard-centric with vim keybindings.

### Devon (entropy-research)

Open-source AI pair programmer with both TUI and web interfaces.

- **GitHub:** github.com/entropy-research/Devon
- **Models:** Claude (primary), GPT-4, Groq, Ollama/DeepSeek Coder
- **Features:** Multi-file editing, codebase exploration, config/test writing, bug fixes
- **Interface:** TUI (devon-tui) + Web UI
- **Language:** Python/TypeScript
- **License:** AGPL-3.0
- **Stars:** ~3,500
- **Distinguishing:** Named after but distinct from Cognition's Devin. Community-driven, offers both terminal and web interfaces.

### Toad (Will McGugan / Batrachian AI)

Universal front-end for AI agents in the terminal, by the creator of Rich and Textual.

- **GitHub:** github.com/batrachianai/toad
- **Models:** Backend-agnostic (connects to OpenHands, Claude Code, Gemini CLI, etc.)
- **Features:** Agent Communication Protocol (ACP) for connecting to 12+ agent backends, beautiful Markdown rendering, no-flicker alternate buffer mode, @-convention fuzzy file search, session resume
- **Interface:** TUI (front-end only)
- **Language:** Python
- **Status:** New, actively developed
- **Distinguishing:** **Front-end/back-end separation** — not an agent itself but a universal TUI for any agent. ACP protocol enables connecting to any backend. From the creator of Rich/Textual.

---

## Shell and command assistants

Tools primarily focused on helping with shell commands, scripting, and terminal productivity rather than multi-file code editing.

| Tool | GitHub | Stars | Language | License | Models | Key feature |
|------|--------|-------|----------|---------|--------|-------------|
| **ShellGPT (sgpt)** | github.com/TheR1D/shell_gpt | ~10,000 | Python | MIT | OpenAI + LiteLLM (Ollama, Azure, etc.) | Shell command generation with `--shell`, REPL mode, Bash/ZSH hotkeys |
| **Fabric** | github.com/danielmiessler/fabric | ~28,000 | Go | MIT | OpenAI, Claude, Ollama, more | Crowdsourced "Patterns" (reusable prompts), pipe-friendly, YouTube transcripts |
| **Mods** | github.com/charmbracelet/mods | ~4,000 | Go | MIT | OpenAI, Ollama, Groq, Gemini | Pipeline-friendly AI, part of Charm ecosystem, beautiful output |
| **Butterfish** | github.com/bakks/butterfish | ~3,500 | Go | MIT | OpenAI + compatible APIs | Shell wrapper — history becomes AI context, Goal Mode for autonomy |
| **AI Shell (Builder.IO)** | github.com/BuilderIO/ai-shell | ~4,000 | TypeScript | MIT | OpenAI | Natural language to shell commands, inspired by Copilot X CLI |
| **AI Shell (Microsoft)** | github.com/PowerShell/AIShell | ~1,500 | C# | MIT | Azure OpenAI, Copilot | Split-pane in Windows Terminal, PowerShell-focused |
| **Gorilla CLI** | github.com/gorilla-llm/gorilla-cli | ~1,300 | Python | Apache 2.0 | Multi-LLM aggregation | UC Berkeley research; aggregates multiple LLM responses |
| **tgpt** | github.com/aandrew-me/tgpt | ~2,000 | Go | GPL-3.0 | Free providers (no API key needed) | Zero-config, cross-platform binary, image generation |
| **Amazon Q CLI** | github.com/aws/amazon-q-developer-cli | ~5,000 | Rust | MIT/Apache 2.0 | Amazon models | Autocomplete for 500+ CLIs, now deprecated → Kiro CLI |
| **OpenCommit** | github.com/di-sukharev/opencommit | ~8,000+ | TypeScript | Open | OpenAI, Ollama, Mistral | `oco` generates conventional commit messages |
| **AICommits** | github.com/Nutlope/aicommits | ~8,000 | TypeScript | Open | OpenAI | AI-powered git commit message generation |

**Fabric** deserves special mention — its "Pattern" system (crowdsourced reusable markdown prompts for tasks like `extract_wisdom`, `summarize`, `analyze_claims`) represents a distinct approach. Rather than an agent, it is an AI augmentation framework. **~28,000 stars** make it one of the more popular tools in the ecosystem.

---

## Multi-agent coding frameworks

These tools simulate software teams or use multiple specialized agents working together on development tasks. Most originated as research projects.

### MetaGPT

Multi-agent framework simulating a software company — "First AI Software Company." **~48,000 stars.** ICLR 2024 oral paper (top 1.2%).

- **GitHub:** github.com/FoundationAgents/MetaGPT
- **Models:** OpenAI GPT-4/3.5, Azure, Ollama, Groq
- **Features:** Product managers, architects, project managers, engineers as agents; SOP-driven workflows; generates PRDs, designs, APIs from one-line input
- **Interface:** CLI (`metagpt` command)
- **Language:** Python
- **License:** MIT
- **Distinguishing:** Highly structured multi-agent approach. "Code = SOP(Team)" philosophy. Strong academic pedigree.

### GPT-Pilot (Pythagora)

AI developer with 6+ specialized agents that writes scalable apps while the developer oversees. **~32,000 stars.**

- **GitHub:** github.com/Pythagora-io/gpt-pilot
- **Models:** GPT-4 (primary), configurable
- **Features:** Product Owner, Spec Writer, Architect, Tech Lead, Developer, Code Monkey agents; step-by-step with human review; context rewinding; TDD approach
- **Interface:** CLI + VS Code extension
- **Language:** Python
- **License:** Open source
- **Distinguishing:** Mimics real software company workflow. Developer stays in the loop as lead. Pythagora Pro is the commercial product.

### ChatDev

Virtual software company using communicative agents. **~26,000 stars.** ACL 2024, NeurIPS 2025.

- **GitHub:** github.com/OpenBMB/ChatDev
- **Models:** OpenAI (primary); Ollama fork for local
- **Features:** CEO, CTO, CPO, Programmer, Reviewer, Tester, Art Designer roles; DAG-based collaboration; v2.0 with YAML workflows
- **Interface:** CLI + Web Console
- **Language:** Python
- **License:** Apache 2.0
- **Distinguishing:** Strong academic backing (Tsinghua/OpenBMB). Communicative dehallucination. Among the more role-diverse multi-agent systems in this survey.

### GPT-Engineer

Generates entire codebases from prompts. **~52,000 stars** — precursor to Lovable.dev.

- **GitHub:** github.com/AntonOsika/gpt-engineer
- **Models:** OpenAI GPT-4
- **Features:** One-prompt codebase generation, clarifying questions, benchmarking tools
- **Interface:** CLI
- **Language:** Python
- **License:** MIT
- **Status:** Winding down — succeeded by commercial Lovable.dev

### Devika

Agentic AI software engineer, open-source Devin alternative. **~19,500 stars.** Rebranded to "Opcode."

- **GitHub:** github.com/stitionai/devika
- **Models:** Claude 3, GPT-4, Gemini, Mistral, Groq, Ollama
- **Features:** Planning engine, web browsing, multi-language code generation, browser interaction
- **Interface:** Web UI (not strictly CLI)
- **Language:** Python
- **License:** MIT
- **Status:** Early/experimental, many unimplemented features

### smol developer

Minimal "junior developer" agent in **under 200 lines** of Python. ~12,200 stars.

- **GitHub:** github.com/smol-ai/developer
- **Models:** OpenAI GPT-4
- **Interface:** CLI
- **License:** MIT
- **Status:** Largely inactive — served as proof of concept

---

## Multi-agent orchestration and session management

A growing sub-category: tools that manage multiple AI coding agents running simultaneously, rather than being agents themselves.

| Tool | GitHub | Stars | Language | Description |
|------|--------|-------|----------|-------------|
| **Claude Squad** | github.com/smtg-ai/claude-squad | ~5,000 | Go | Manage multiple AI agents (Claude Code, Aider, Codex, OpenCode, Amp) in separate workspaces simultaneously |
| **Ruflo (formerly Claude-Flow)** | github.com/ruvnet/ruflo | ~3,000 | TypeScript | Agent orchestration platform for Claude — multi-agent swarms, distributed intelligence |
| **Agent Deck** | github.com/asheshgoplani/agent-deck | ~500 | Go | Terminal session manager TUI for Claude, Gemini, OpenCode, Codex with smart status detection |
| **Floki** | github.com/FinnaAI/floki | ~300 | TypeScript | Work with multiple AI coding agents in parallel |
| **Conduit** | github.com/lostintangent/conduit-release | ~186 | TypeScript | Terminal-centric workspace manager for task parallelization |
| **Metacoder** | github.com/ai4curation/metacoder | ~100 | Python | Unified interface for CLI AI coding assistants |
| **Emdash** | github.com/generalaction/emdash | ~894 | TypeScript | Coding agent orchestration layer |
| **Omnara** | github.com/omnara-ai/omnara | ~200 | TypeScript | Command center syncing sessions across terminal, web, mobile |
| **Warp** | warp.dev | ~25,000 | Rust | Terminal replacement that runs multiple agents simultaneously |

This category was much smaller six months ago. **Claude Squad** is a notable example; it lets you run Claude Code, Aider, Codex, and other agents in separate git worktrees simultaneously, monitoring their progress through a unified TUI. The emergence of orchestration tools signals that the "one agent per task" paradigm is shifting toward parallel multi-agent workflows.

---

## Research and academic SWE-bench tools

Tools designed primarily for software engineering research, particularly benchmarking against SWE-bench.

| Tool | GitHub | Stars | Language | License | SWE-bench Score | Key innovation |
|------|--------|-------|----------|---------|-----------------|----------------|
| **SWE-agent** | github.com/SWE-agent/SWE-agent | ~15,000 | Python | MIT | SOTA (open source) | Custom Agent-Computer Interface, Docker execution |
| **mini-swe-agent** | github.com/SWE-agent/mini-swe-agent | New | Python | MIT | >74% verified | **100 lines of code**, bash-only, Princeton/Stanford |
| **OpenHands** | github.com/OpenHands/OpenHands | ~67,000 | Python | MIT | Competitive | Full platform with web GUI, CLI, cloud; Docker-sandboxed |
| **AutoCodeRover** | github.com/AutoCodeRoverSG/auto-code-rover | ~2,500 | Python | GPL-3.0 | 46.2% verified | AST-aware code search, <$0.7/task |
| **Agentless** | github.com/OpenAutoCoder/Agentless | ~2,000 | Python | MIT | 50.8% (Claude) | No agent scaffolding — simple localize→repair→validate |
| **Open SWE** | github.com/langchain-ai/open-swe | ~2,000 | Python | Open | N/A | Async coding agent built with LangGraph |
| **kwaak** | github.com/bosun-ai/kwaak | ~800 | Rust | Open | N/A | Run a team of autonomous agents on code |

**mini-swe-agent** is notable, with reported >74% on SWE-bench verified in just 100 lines of bash. **Agentless** challenges the complexity assumption by showing that a simple three-phase pipeline (localize → repair → validate) without any agent scaffolding can match or beat elaborate agent systems.

---

## Lightweight and specialized utilities

Smaller, focused tools for specific use cases rather than general-purpose coding.

| Tool | GitHub | Stars | Language | License | Purpose |
|------|--------|-------|----------|---------|---------|
| **Rawdog** | github.com/granawkins/rawdog | ~3,000 | Python | Permissive | Generates and auto-executes Python scripts |
| **Pieces CLI** | github.com/pieces-app/cli-agent | ~200 | Python | MIT | AI-powered code snippet manager |
| **agent-cli** | github.com/basnijholt/agent-cli | ~200 | Python | Open | Local AI suite (chat, autocorrect, transcribe, RAG) |
| **Vibe Compiler (vibec)** | Referenced | ~200 | Rust | Open | Self-compiling markdown prompt stacks → code |
| **wcgw** | github.com/rusiaaman/wcgw | ~626 | Python | Open | Shell/coding agent on MCP clients |
| **octogen** | github.com/dbpunk-labs/octogen | ~257 | Python | Open | Open-source code interpreter framework |
| **oi** | github.com/oi-overide/oi | ~300 | Python | Open | CLI using CodeLlama, generates code in any editor |
| **DuetGPT** | Referenced | ~500 | TypeScript | Open | Conversational semi-autonomous pair programmer |

---

## Terminal editor AI plugins

Neovim and Emacs plugins that bring AI coding to terminal-based editors.

**Neovim plugins:**

| Plugin | GitHub | Stars | Description |
|--------|--------|-------|-------------|
| **avante.nvim** | github.com/yetone/avante.nvim | ~8,000 | Emulates Cursor AI behavior in Neovim |
| **codecompanion.nvim** | github.com/olimorris/codecompanion.nvim | ~5,000 | AI coding, Vim style, multi-model |
| **copilot.vim** | github.com/github/copilot.vim | ~11,000 | Official GitHub Copilot for Vim/Neovim |
| **chatgpt.nvim** | github.com/jackmort/chatgpt.nvim | ~1,000 | ChatGPT in Neovim |
| **gp.nvim** | github.com/Robitx/gp.nvim | ~1,000 | GPT prompt plugin with chat sessions |
| **gen.nvim** | github.com/David-Kunz/gen.nvim | ~1,000 | Generate text with LLMs and custom prompts |
| **VimLM** | github.com/JosefAlbers/Vim-LM | ~500 | Copilot/Cursor-inspired local LLM companion |

**Emacs plugins:**

| Plugin | GitHub | Stars | Description |
|--------|--------|-------|-------------|
| **gptel** | github.com/karthink/gptel | ~2,000 | Simple LLM chat client, multiple backends |
| **aidermacs** | github.com/MatthewZMD/aidermacs | ~1,000 | Aider integration for Emacs |
| **Emigo** | github.com/MatthewZMD/emigo | ~500 | Intelligent agentic Emacs-native AI assistant |
| **copilot.el** | github.com/copilot-emacs/copilot.el | ~2,000 | Unofficial Copilot for Emacs |

---

## Context and code preparation tools

Tools that prepare codebases for LLM consumption rather than being agents themselves, but critical infrastructure in the ecosystem.

| Tool | GitHub | Stars | Description |
|------|--------|-------|-------------|
| **Repomix** | github.com/yamadashy/repomix | ~10,000 | Packs entire repo into single AI-friendly file |
| **claude-context** | github.com/zilliztech/claude-context | ~2,000 | Code search MCP for coding agents |
| **Jinni** | github.com/smat-dev/jinni | ~200 | Bring your project into LLM context |
| **talk-codebase** | Referenced | ~500 | CLI chatbot with repository as context |

---

## Cross-category comparison of major tools

### Model support comparison

| Tool | OpenAI | Anthropic | Google | Local (Ollama) | Other providers |
|------|--------|-----------|--------|----------------|-----------------|
| Claude Code | ✗ | ✅ (only) | ✗ | ✗ | Via proxy only |
| Aider | ✅ | ✅ | ✅ | ✅ | OpenRouter, Azure, 50+ |
| Codex CLI | ✅ (only) | ✗ | ✗ | ✗ | Via community fork |
| OpenCode | ✅ | ✅ | ✅ | ✅ | 75+ providers |
| Gemini CLI | ✗ | ✗ | ✅ (only) | ✗ | ✗ |
| Crush | ✅ | ✅ | ✅ | ✅ | Groq, OpenRouter, Bedrock |
| Goose | ✅ | ✅ | ✅ | ✅ | Any with tool calling |
| Copilot CLI | ✅ | ✅ | ✗ | ✗ | Grok |
| Amp | ✅ | ✅ | ✅ | ✗ | Grok, auto-selected |
| Plandex | ✅ | ✅ | ✅ | ✅ | OpenRouter |
| AIChat | ✅ | ✅ | ✅ | ✅ | 20+ natively |
| LLM | ✅ | ✅ | ✅ | ✅ | 100+ via plugins |
| Open Interpreter | ✅ | ✅ | ✗ | ✅ | Via LiteLLM |

### Capability comparison

| Tool | File edit | Shell exec | Git | MCP | Sandbox | Multi-agent | Voice | Browser |
|------|-----------|-----------|-----|-----|---------|-------------|-------|---------|
| Claude Code | ✅ | ✅ | ✅ | ✅ | ✗ | ✅ | ✗ | ✗ |
| Aider | ✅ | ✗ | ✅✅ | ✗ | ✗ | Partial | ✅ | ✗ |
| Codex CLI | ✅ | ✅ | ✅ | ✅ | ✅✅ | Partial | ✗ | ✗ |
| OpenCode | ✅ | ✅ | ✅ | ✅ | ✗ | ✅ | ✗ | ✗ |
| Gemini CLI | ✅ | ✅ | ✗ | ✅ | ✗ | ✗ | ✗ | ✅ |
| Crush | ✅ | ✅ | ✅ | ✅ | ✗ | ✗ | ✗ | ✗ |
| Goose | ✅ | ✅ | ✅ | ✅✅ | ✗ | ✅ | ✗ | ✗ |
| Roo Code | ✅ | ✅ | ✅ | ✅ | ✗ | ✅ | ✗ | ✅ |
| Open Interpreter | ✅ | ✅ | ✗ | ✗ | ✗ | ✗ | ✅ | ✗ |

### Technical implementation comparison

| Tool | Written in | Binary size | Startup | Architecture |
|------|-----------|-------------|---------|--------------|
| Claude Code | TypeScript | npm package | Medium | Client binary |
| Aider | Python | pip package | Slow | Python process |
| Codex CLI | Rust | Native binary | Fast | Sandboxed process |
| OpenCode | Go/TS | Native binary | Fast | Client/server |
| Gemini CLI | TypeScript | npm package | Medium | Client binary |
| Crush | Go | Native binary | Fast | Session-based TUI |
| Goose | Rust/TS | Native binary | Fast | MCP-native |
| AIChat | Rust | Native binary | Very fast | Single binary |
| Plandex | Go | Native binary | Fast | Client/server |

---

## Landscape analysis and trends

### The market has consolidated around three tiers

**Tier 1 — Lab-backed flagships** (Claude Code, Codex CLI, Gemini CLI, Copilot CLI): Most major AI labs now ship a terminal coding agent. These tools serve as both products and model showcases. Claude Code is often cited for reasoning quality, Codex CLI for sandboxing, Gemini CLI for free access, and Copilot CLI for GitHub ecosystem integration.

**Tier 2 — Model-agnostic open-source** (Aider, OpenCode, Goose, Crush, Continue): These tools deliberately avoid model lock-in. Aider is among the most mature, OpenCode among the more provider-diverse, and Goose highly extensible. This tier often benefits when new models launch; it can adopt quickly while lab tools are locked to their own models.

**Tier 3 — Specialized and emerging** (Plandex, AIChat, Droid, Kimi CLI, multi-agent frameworks): Tools that differentiate through unique architecture (Plandex's 2M token context, Droid's specialized sub-agents, Kimi's dual-mode shell) rather than trying to be general-purpose.

### Seven key trends shaping the landscape

**MCP as broadly adopted standard.** Model Context Protocol has become a leading extensibility mechanism. Many major tools now support MCP, creating a shared ecosystem of tools and integrations. This is a major structural development; it means tools are becoming more interoperable rather than fully siloed.

**Multi-agent orchestration is a fast-growing category.** Six months ago, Claude Squad didn't exist. Now there are many tools for managing multiple agents simultaneously. This reflects a workflow shift: developers increasingly run 2-3 agents on different tasks in parallel rather than using one agent sequentially.

**Rust and Go are displacing Python for new tools.** While Python dominates older tools (Aider, Open Interpreter, MetaGPT, SWE-agent), many tools launched in 2025-2026 are written in Rust, Go, or TypeScript. Codex CLI's rewrite from TypeScript to Rust and Goose's Rust core suggest that **startup performance and binary distribution** are increasingly prioritized over Python's ecosystem advantages for CLI tools.

**The free tier war.** Gemini CLI offers 60 req/min free, Qwen Code gives 1,000 req/day free, Amp provides $10/day ad-supported, and Grok CLI's model costs $0.20/1M tokens. The barrier to entry is collapsing. This commoditizes basic coding assistance and pushes differentiation toward reasoning quality, context management, and workflow integration.

**Local model support is now common but rarely primary.** Many tools support Ollama, but community discussions often report that local models underperform cloud models for complex coding tasks. Local support matters for privacy-sensitive environments and experimentation, but frontier cloud models remain the primary productivity drivers for many teams.

**Terminal-Bench is emerging as an important benchmark.** Just as SWE-bench standardized evaluation for research agents, Terminal-Bench (where Droid has a reported leading score at 58.75%) is becoming a key benchmark for CLI coding tools in real-world terminal scenarios.

**The "why CLI over IDE?" question is often answered in favor of CLI for certain workflows.** HN and Reddit discussions converge on recurring advantages: cost savings (no IDE subscription), better separation of concerns (agent in separate worktree), higher-quality interactions (deliberate prompting vs. autocomplete), SSH compatibility, and composability (terminal is the natural interface for tool use).

### Most mature tools

**Aider** is one of the more battle-tested open-source tools, with 40,000+ stars, 4.1M installs, vendor-reported very high weekly token volume, and 230+ contributors. **Claude Code** is widely viewed as a polished commercial tool for complex reasoning. **OpenHands** is among the more mature platforms for autonomous headless operation.

### Most promising tools

**Crush** (Charm ecosystem backing, beautiful UX, broad platform support), **OpenCode** (model-agnostic, growing rapidly), **Kimi Code CLI** (distinctive dual-mode shell, Agent Swarm), and **mini-swe-agent** (showing that compact implementations can post high SWE-bench scores) represent several notable new directions.

### Gaps in the market

**Few tools currently combine all three dimensions strongly: reasoning quality + model flexibility + strong sandboxing.** Claude Code is strong on reasoning but locked to Anthropic. Codex CLI is strong on sandboxing but tied closely to OpenAI. Aider has strong model flexibility but no built-in sandboxing. A tool that combines all three could become category-defining.

**Code review is underserved.** While several tools offer code review features, few CLI tools are purpose-built for AI-assisted code review workflows. Devin's `npx devin-review` and Codex's review agent are early entries but neither is comprehensive.

**Team collaboration is nascent.** Amp's thread sharing is distinctive. Many tools are still fundamentally single-developer. As organizations adopt CLI agents, tools that enable team workflows (shared sessions, review of agent output, audit trails) will have an advantage.

**Testing integration is weak across the board.** Most agents can *run* tests but few can *write* meaningful test suites or use test results to guide their approach. Kiro's property-based testing and GPT-Pilot's TDD approach are exceptions.

### The convergence pattern

Many tools are converging toward a similar feature set: file editing + shell execution + git integration + MCP extensibility + multi-model support + sub-agents. The differentiation is increasingly about **default model quality, UX polish, context management strategy, and ecosystem integrations** rather than raw capabilities. The tools that win will likely be those that make common workflows effortless while supporting power-user customization, much like the text editor wars that preceded them.
