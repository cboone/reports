# Reports

Detailed reports on various topics, created by LLMs, usually the latest version of Claude. Everything in here was written by an LLM (other than this intro). Often multiple LLMs, or at least multiple rounds without preserving context, were used to fact-check, increase accuracy, and fill in gaps. I did my best to encourage real references, citations, etc. But you know how this works. Double check anything important.

[AI Ethics](#ai-ethics)
∙ [Agile](#agile)
∙ [LLM Agents](#llm-coding-agents)
∙ [Programming Languages](#programming-languages)
∙ [Self](#self)

## AI Ethics

### [Moral philosophy confronts generative AI: mapping the ethical landscape](ai-ethics/moral-philosophy-generative-ai.md)

How virtue ethics, care ethics, Rawlsian justice, and non-Western frameworks like Ubuntu and Confucian ethics are competing to shape the evaluation of generative AI in early 2026. Covers the shift from principalism toward structural analysis, the four main impact domains (social, political, economic, environmental), key fault lines dividing the field, and the gap between philosophical sophistication and practical governance.

## Agile

### [The complete history of agile: from Toyota to transformation](agile/history.md)

How agile methodology emerged from three tributaries — Toyota's production system, early iterative experiments at NASA and IBM, and mounting evidence of waterfall's failures — and what happened after 17 practitioners met in Utah's Wasatch Mountains in February 2001. Covers the pre-history through lean manufacturing, the methodology wars of the 1990s, the Snowbird meeting, the explosion of frameworks (Scrum, XP, Kanban), enterprise adoption, and the ongoing debates about whether mainstream agile has preserved or diluted its original values.

### [Complete timeline of agile programming](agile/timeline.md)

Chronological timeline from Toyota's factory floors in the 1940s to today's global software industry. Traces the full arc: manufacturing philosophy, early iterative development at NASA and IBM, the methodology wars, the Agile Manifesto, the rise of Scrum and SAFe, and the current landscape of agile practice.

### [Agile methodology: sources and references](agile/bibliography.md)

Primary sources, foundational documents, and further reading for the history of agile, including the original Manifesto documents, key books by Beck, Schwaber, Cockburn, and others, and academic research on agile adoption and outcomes.

## LLM Coding Agents

### [Every LLM coding agent for the terminal in early 2026](llm-agents/coding-agent-clis.md)

Catalog of over 90 CLI and TUI coding tools that use large language models for software development. Organized into categories — full-featured agentic assistants, code-aware chat and REPL tools, shell and command assistants, multi-agent coding frameworks, orchestration and session management tools, research and SWE-bench tools, terminal editor plugins, and context preparation utilities — with cross-category comparison tables and landscape analysis covering market consolidation, key trends, and gaps.

### [Axes for reasoning about LLM agent tool use](llm-agents/agent-tool-axes.md)

A framework of eight independent axes for reasoning about the design space of agent-tool systems: interaction topology, degree of autonomy, security surface, state mutation, determinism and idempotency, error recovery and observability, context cost, and trust boundary. Each axis is defined with concrete examples, and the document maps how axes interact to create emergent risk (the mutation-autonomy-security triangle) or emergent capability.

### [LLM coding agent instruction files: comparing CLAUDE.md, AGENTS.md, copilot-instructions.md, and SKILL.md](llm-agents/agent-instruction-files.md)

How the four major instruction file formats work and compare across Claude Code, Codex, Copilot, and OpenCode. Covers file location, hierarchical loading, path-scoping, import mechanisms, and override semantics for each format, with concrete recommendations for cross-tool compatibility including the AGENTS.md-as-source-of-truth strategy with symlinks.

### [LLM coding agent configuration files: comparing Claude Code, Codex, Copilot, and OpenCode](llm-agents/agent-config-files.md)

Where every configuration file lives, what it controls, and which files should be committed to version control across the four major LLM coding agents. Covers permissions, sandboxing, MCP server configuration, hooks, model selection, named profiles, and admin enforcement, with a decision framework for what goes where and a dotfiles strategy for syncing preferences across tools and machines.

### [LLM coding agent monitoring, session management, and notification tools](llm-agents/llm-agent-monitoring-tools.md)

Every notable tool available as of mid-February 2026 for monitoring LLM coding agent activity, managing sessions, receiving notifications, and tracking usage and costs. Covers desktop session managers (macOS), terminal TUI session managers, desktop apps for parallel agent workflows, mobile and remote clients, notification hooks, usage and cost monitoring CLIs, IDE extensions, LLM observability platforms, and agent SDKs, with comparison tables across each category.

### [Parallel LLM coding agents with tmux and git worktrees](llm-agents/tmux-worktree-tools.md)

Comprehensive survey of tools at the intersection of LLM coding agents, tmux session management, and git worktree isolation. Profiles every significant tool from full-featured TUI session managers (claude-squad, ccmanager, agent-deck) through opinionated workflow tools (workmux, dmux, barrel) to lightweight shell scripts, with architectural analysis of the core pattern, the containerization gap, worktree conventions, and community discussion.

## Programming Languages

### [Three algorithms, dozens of languages: an overview](programming-languages/three-algorithms-overview.md)

Introduces and connects three companion studies that each implement a single algorithm across dozens of programming languages. Includes a summary table rating every language family across all three problems and analysis of cross-cutting themes: how data shape determines paradigm fit, the abstraction spectrum from APL to C, and how mainstream languages borrow from specialized paradigms over time.

### [Finding indices of elements above the mean: a cross-language study](programming-languages/indices-above-mean.md)

A single array-transformation algorithm implemented in over 30 languages organized by family: APL, ML, Lisp, BEAM, JVM, C, modern systems languages, scripting languages, proof assistants, and historical languages. Reveals how languages differ in explicitness vs. implicitness, static vs. dynamic typing, functional vs. imperative style, and memory management approaches.

### [Evaluating expression trees: a cross-language study](programming-languages/expression-tree-evaluation.md)

A recursive expression tree evaluator implemented across the same language families, showcasing algebraic data types, pattern matching, and the expression problem. The ML family excels here where array languages struggle, inverting the results of the first study and demonstrating that language strengths are fundamentally problem-dependent.

### [Concurrent producer-consumer pipeline: a cross-language study](programming-languages/concurrent-producer-consumer.md)

A concurrent producer-consumer pipeline with bounded buffering, backpressure, and shutdown coordination implemented across language families. Compares message passing (Erlang, Go channels), shared memory with locks (C, C++), and transactional memory (Haskell STM), revealing how each language's concurrency model reflects the problems it was designed to solve.

## Self

### [Self: the invisible language that powers modern computing](self/history.md)

How a radical simplification of Smalltalk — removing classes entirely in favor of prototypes — produced the optimization techniques that now power every major JavaScript engine and the Java Virtual Machine. Traces Self's journey from Xerox PARC through Stanford to Sun Microsystems, the breakthroughs in adaptive compilation and polymorphic inline caches, the direct lineage to Java HotSpot and V8, and Self's influence on JavaScript's prototype-based object model.

### [Timeline of Self and its legacy](self/timeline.md)

Chronological timeline from Smalltalk's creation at Xerox PARC in 1972 through Self's design, implementation, and optimization breakthroughs in the late 1980s and 1990s, to its lasting impact on JavaScript, Java HotSpot, V8, and modern JIT compilation techniques.

### [The Self programming language: key papers and resources](self/bibliography.md)

Key academic papers and resources on Self and its technical legacy, including the foundational OOPSLA and ECOOP papers by Ungar, Smith, Chambers, and Hölzle, doctoral theses, implementation guides, and the Self language website and community resources.
