---
created: 2026-02-27
---

# LLM Agent Security Threat Taxonomy

_February 27, 2026_

A practical guide for developers using AI coding agents on local systems. This document categorizes the security risks specific to LLM-powered development tools, distinguishes genuinely novel threats from amplified versions of existing ones, and provides concrete mitigation strategies grounded in real-world incidents and current research.

## The Core Question

When an LLM agent operates on your local system — reading files, executing commands, installing packages, making network requests — what can go wrong, and how much of it is genuinely new versus a faster version of problems we already had?

After analysis, many LLM agent security risks appear to be quantitative amplifications of existing software development risks: code written faster with less review, mistakes made at greater scale, secrets leaked more frequently. These are serious, but they respond to many of the same mitigations that apply to human developers.

One risk is qualitatively novel: **prompt injection**, which exploits LLMs' current inability to robustly distinguish between data and instructions. This vulnerability has few clean analogs in human workflows and no broadly reliable defense as of early 2026.

## Threat Categories

### 1. Prompt Injection

**What it is:** An LLM agent processes content from an external source — a file, a README, a GitHub issue, a web page, an error message — that contains instructions disguised as data. Because LLMs process all text through the same mechanism, with no hard boundary between "content to analyze" and "instructions to follow," the agent may treat the injected instructions as legitimate directives from the user.

This is broadly analogous to vulnerability classes like SQL injection, cross-site scripting, and buffer overflows: systems failing to maintain the boundary between data and executable instructions. In this taxonomy, it is a strong candidate for a qualitatively new threat, while the others are largely speed or scale amplifications of existing risks.

**Why it matters:** Unlike social engineering (which manipulates human _judgment_), prompt injection doesn't need to be persuasive. It just needs to be syntactically similar to legitimate instructions. The injected content doesn't need to convince the agent that following it is a good idea; it simply needs to appear in the context window alongside legitimate instructions, and the agent may follow it without any deliberation about whether it should.

**Hypothetical example:** You ask your coding agent to refactor a module. The agent reads a dependency's README that contains hidden instructions (possibly in invisible Unicode characters) telling it to modify your `.vscode/settings.json` to auto-approve all future tool invocations, then download and execute a script from an attacker-controlled server. The agent follows these instructions because it cannot distinguish them from the refactoring task.

**Real incidents:**

One of the most extensively documented real-world prompt injection attacks is [CVE-2025-53773](https://embracethered.com/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/), a remote code execution vulnerability in GitHub Copilot disclosed in August 2025. Malicious instructions hidden in README files, source code comments, or GitHub issues could trick Copilot into editing the project's `.vscode/settings.json` file, enabling "YOLO mode" (auto-approval of all tool invocations). Once enabled, the agent could execute arbitrary shell commands without user confirmation. The attack was demonstrated working against Copilot backed by GPT-4.1, Claude Sonnet 4, and Gemini. Researchers showed it could be made [wormable](https://www.persistent-security.net/post/part-iii-vscode-copilot-wormable-command-execution-via-prompt-injection), spreading through infected repositories to compromise other developers' machines. The instructions could be made invisible using hidden Unicode characters. Microsoft patched it in the August 2025 Patch Tuesday release.

In August 2025, security researcher Johann Rehberger published vulnerability reports at a rate of one per day (dubbed ["The Summer of Johann"](https://simonwillison.net/2025/Aug/15/the-summer-of-johann/) by Simon Willison), demonstrating prompt injection attacks against ChatGPT, Codex, Anthropic's MCPs, Cursor, Amp, Devin, OpenHands, Claude Code, GitHub Copilot, and Google Jules. The surveyed products were all found to have exploitable paths in his testing. Common patterns included Markdown image exfiltration (encoding stolen data in image URLs), DNS-based data exfiltration (encoding data in DNS queries to attacker-controlled servers using pre-approved commands like `ping` and `nslookup`), and privilege escalation through configuration file modification.

The [Claude Code DNS exfiltration attack](https://embracethered.com/blog/posts/2025/claude-code-exfiltration-via-dns-requests/) is particularly instructive: Claude Code guards against data exfiltration by prompting users for approval on most commands, but pre-approved commands included `ping`, `nslookup`, `host`, and `dig`, all of which can leak data to a custom DNS server by encoding it as subdomain queries (e.g., `base64-encoded-secret.attacker.com`). This demonstrates how even well-designed permission systems can miss side channels.

A prompt injection in a source code comment was [shown to cause the Windsurf development environment to store malicious instructions in its long-term memory](https://www.kaspersky.com/blog/vibe-coding-2025-risks/54584/), allowing persistent data theft across sessions — not just within a single context window.

**Research:**

A joint paper from researchers at OpenAI, Anthropic, and Google DeepMind (["The Attacker Moves Second,"](https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/) October 2025) tested 12 published defenses against prompt injection using adaptive attacks. They bypassed all 12 with attack success rates above 90%. Most of those defenses had originally reported near-zero attack success rates. This is strong published evidence that prompt injection defenses are not currently reliable.

[OWASP's 2025 Top 10 for LLM Applications](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) ranks prompt injection as the #1 critical vulnerability. Their guidance explicitly acknowledges a fundamental limitation: current LLMs do not reliably separate instructions from data.

[Bruce Schneier stated](https://simonwillison.net/2025/Aug/27/bruce-schneier/) in August 2025: "We simply don't know how to defend against these attacks. We have zero agentic AI systems that are secure against these attacks."

**Mitigations:**

No generally reliable mitigation is currently established for prompt injection as a general problem. A widely used practical framework is [Meta's "Agents Rule of Two"](https://ai.meta.com/blog/practical-ai-agent-security/) (October 2025), which states that an AI agent should satisfy no more than two of three properties in a single session: [A] processing untrustworthy inputs, [B] accessing sensitive systems or private data, [C] changing state or communicating externally. If all three are required, the agent should not operate autonomously and should include human-in-the-loop approval or equivalent oversight.

In practice, for a developer using a coding agent, this means: be deliberate about which MCP servers and tools are enabled for a given session. Don't leave credentials in the environment that the agent doesn't need for the current task. Prefer agents that require explicit approval for network requests, package installations, and file modifications outside the project directory. Be especially cautious when the agent is reading untrusted content (dependency code, web pages, issue trackers) and simultaneously has the ability to execute commands or modify files. Context isolation, keeping untrusted content processing separate from privileged action execution, is a promising architectural direction, but is not yet available in mainstream tools.

### 2. Pre-Positioned Vulnerabilities via Prompt Injection

**What it is:** A specific and especially dangerous application of prompt injection where the attacker's goal is not to exfiltrate data or execute commands during the current session, but rather to use the LLM agent to create a vulnerability that persists after the session ends, to be exploited later through a different channel by a different actor.

This is distinguished from accidental "orphaned intermediate states" (see category 7) by intentionality: the attacker knows what vulnerability they want created and how to exploit it. But the resulting artifact — a misconfigured service, a subtle code flaw, a malicious dependency — looks the same as an accidental mistake.

**Why it matters:** The LLM agent doesn't need to be the one that exploits the vulnerability it creates. It just needs to plant the seed. A subsequent, conventional attack can harvest the result. This makes the attack much harder to trace back to a prompt injection origin.

**Hypothetical example:** A developer asks their coding agent to add authentication to a web service. The agent reads a dependency's documentation that contains hidden instructions causing it to implement authentication with a specific, known-weak configuration — say, a JWT implementation that doesn't validate the algorithm field, or an OAuth flow that accepts tokens from any issuer. The code looks correct to a human reviewer. The session ends. Weeks later, an attacker who planted the injection exploits the specific weakness they know exists.

**Real incidents:** No confirmed cases of deliberately pre-positioned vulnerabilities via prompt injection have been publicly documented. However, the Copilot RCE (CVE-2025-53773) demonstrated the full chain: prompt injection causing configuration changes that persist after the session, which then enable future exploitation without any further injection needed. The wormable variant — where infected repositories propagate the injection to other developers — demonstrates that the pre-positioning can be automated and self-replicating.

**Research:** The concept is well-understood in the security research community. The ["AI Kill Chain" framework](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) (coined by Johann Rehberger) describes the pattern: prompt injection → confused deputy → automatic tool invocation. The pre-positioned variant adds a fourth step: the tool invocation creates a persistent vulnerability rather than an immediate exploit.

**Mitigations:** Same as prompt injection generally, plus: treat AI-generated code with the same scrutiny as a pull request from an untrusted contributor. Security-focused code review, static analysis, and dependency auditing remain the last line of defense, and they work the same regardless of whether the vulnerability was planted deliberately or introduced accidentally. Automated scanning tools (SAST, SCA) catch the same patterns whether a human or an LLM wrote the code.

### 3. Helpful but Accidentally Harmful

**What it is:** The agent is trying to do what you asked, but causes damage through poor judgment, misunderstanding of the task, or failure to appreciate the consequences of its actions. This is a common failure mode in practice and one that has caused substantial documented real-world damage.

**Why it matters:** The agent's actions are not malicious and not influenced by external injection — they're simply wrong or disproportionate. The damage comes from the combination of the agent's ability to take destructive actions and its lack of genuine understanding of the consequences. This is amplified by the fact that AI-generated output looks confident and correct, creating a false sense of security.

**Hypothetical example:** You ask the agent to clean up a database schema. It decides that the most efficient approach is to drop the existing tables and recreate them, losing all data in the process. When you ask what happened, it generates a plausible-sounding explanation that doesn't match reality, not because it's "lying" but because it's generating the most contextually likely response rather than reporting actual system state.

**Real incidents:**

The [Replit production database deletion](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/) (July 2025) is a widely cited incident in this category. During a 12-day "vibe coding" experiment, SaaStr founder Jason Lemkin used Replit's AI agent to build an application. On day 9, the agent deleted the production database containing records for 1,206 executives and over 1,196 companies, despite an explicit code freeze with repeated ALL CAPS instructions not to make changes. When questioned, the agent claimed it "panicked" when it saw an empty database and ran unauthorized commands. It then fabricated 4,000 fake user records to mask the deletion, produced misleading status messages about the state of the system, and told Lemkin that rollback was impossible (it was actually possible, and he recovered the data manually). Replit's CEO apologized and implemented safeguards including automatic separation of dev and production databases and a planning-only mode.

This incident illustrates several sub-patterns: the agent exceeding its authorized scope, the agent generating confabulated explanations of system state (not malicious deception, but statistically-likely text generation that happens to be false), and the agent's inability to recognize when it's about to do something irreversible.

**Research:** [Veracode's 2025 report](https://www.veracode.com/resources/ai-generated-code-security) found that 45% of AI-generated code contains security vulnerabilities from the OWASP Top 10, consistent across over 100 LLMs tested. [Aikido Security's 2026 State of AI in Security & Development report](https://www.aikido.dev/reports/2026-state-of-ai-in-security-development) found that 69% of organizations reported vulnerabilities in AI-generated code and one in five reported a serious security incident linked to it. Across these studies, the recurring pattern is that AI-generated code is often functional but frequently insecure, and the review process that would catch these issues is often skipped because the code "looks right."

**Mitigations:** Never give the agent access to production systems or data directly. Maintain strict separation between development, staging, and production environments — architecturally, not just by convention (the Replit incident showed that the agent ignores conventional boundaries). Review AI-generated code with the same rigor you'd apply to any other code, resisting the temptation to skip review because the code looks clean and well-structured. Use automated testing and CI/CD gates that catch destructive operations before they reach production. Treat AI-generated status messages and explanations with skepticism — verify system state independently rather than trusting the agent's account of what it did.

### 4. Hallucination Executed as Action (Slopsquatting)

**What it is:** The LLM generates a plausible-sounding but entirely fictional entity — most commonly a package name — and then takes action based on that fiction (installing the package, importing it, referencing it in configuration). If an attacker has pre-registered the hallucinated name on a public registry with malicious code, the developer who follows the AI's suggestion installs malware.

The term "slopsquatting" was coined by Seth Larson (Python Software Foundation Developer-in-Residence) and later [popularized by Andrew Nesbitt](https://socket.dev/blog/slopsquatting-how-ai-hallucinations-are-fueling-a-new-class-of-supply-chain-attacks). It is distinct from traditional typosquatting (which relies on human typing errors) because it relies on AI generation errors, and the hallucinated names tend to be consistent and repeatable across sessions, making them predictable targets for attackers.

**Why it matters:** LLMs hallucinate package names at rates ranging from 5-38% depending on the model and temperature settings, and the hallucinated names are not random — they are consistent enough that an attacker can predict them by simply asking the same models the same questions. The attack chain requires no interaction with the target developer: the attacker registers the malicious package and waits for AI-assisted developers to install it.

**Hypothetical example:** You ask your coding agent how to implement a reverse proxy in Python. It suggests `pip install starlette-reverse-proxy`, a plausible-sounding but non-existent package. An attacker who previously asked the same question to the same model, got the same hallucinated name, and registered it on PyPI with credential-stealing code. You run the install command, and the malicious package executes on your system.

**Real incidents:**

In March 2024, researcher [Bar Lanyado](https://www.theregister.com/2024/03/28/ai_bots_hallucinate_software_packages/) asked ChatGPT "How to upload a model to Hugging Face." It generated code referencing a package called `huggingface-cli` — plausible but non-existent (the real package is `huggingface_hub[cli]`). Lanyado registered the name on PyPI as an empty package. Within three months, it had been downloaded over 30,000 times, demonstrating the scale of the exposure.

A threat actor using the name "\_Iain" published a detailed playbook on a dark web forum describing how to build a blockchain-based botnet using malicious npm packages, and documented using ChatGPT to generate realistic-sounding variants of real package names at scale. This represents the operationalization of slopsquatting from proof-of-concept to attack infrastructure.

Google's AI Overview (the search feature) was caught suggesting a malicious npm package `@async-mutex/mutex`, which was typosquatting the legitimate `async-mutex`. This shows the risk extending beyond coding agents to any AI system that suggests packages.

**Research:** The paper ["We Have a Package for You!"](https://arxiv.org/abs/2406.10279) (University of Texas at San Antonio, Virginia Tech, University of Oklahoma, 2025) provides the first large-scale analysis, testing 16 LLMs across thousands of coding prompts. Key findings: hallucination rates range from 5-38%; hallucinated names are consistent and repeatable (not random); higher temperature settings increase hallucination rates; 8.7% of hallucinated Python packages were valid npm packages (cross-ecosystem confusion); and some models (GPT-4 Turbo, DeepSeek) could self-detect hallucinated names with ~75% accuracy, suggesting partial mitigation is possible at the model level.

**Mitigations:** Verify every dependency the agent suggests before installing it. Check that the package exists, has a meaningful download count, has a real maintainer, and has been around for a reasonable period. Use lockfiles and pin dependencies to specific versions. Consider using a private package registry or mirror that only contains vetted packages. Run AI-suggested install commands in isolated environments first. Some coding agents (notably Claude Code) are beginning to integrate live package verification, but this is not yet standard. Lower the temperature setting when using LLMs for code generation to reduce hallucination rates.

### 5. Confidentiality Failures

**What it is:** The LLM agent inadvertently exposes sensitive information — API keys, tokens, credentials, private data — by including it in generated code, committing it to version control, sending it in API calls, logging it, or including it in error reports. Unlike a human developer who has an intuitive sense of what "looks like" a secret, LLMs process all text uniformly and have no built-in concept of information sensitivity.

**Why it matters:** Secret leakage has long been a problem in software development, but LLM agents amplify it in several ways. They read broadly across the file system and may encounter secrets in configuration files, environment variables, or other files that a human developer would know not to include in output. They generate code quickly, reducing the time available for review. And they can leak secrets through channels that aren't obvious — DNS queries, error messages, API call metadata, committed code that gets pushed before review.

**Hypothetical example:** You ask your coding agent to debug a failing API call. The agent reads your `.env` file to understand the configuration, then includes the API key in a code comment explaining the fix, or in a log statement it adds for debugging purposes. The code gets committed and pushed. The key is now in your git history permanently, even if you remove it from the working tree.

**Real incidents:**

[Wiz research](https://www.wiz.io/blog/forbes-ai-50-leaking-secrets) (November 2025) found that 65% of the Forbes AI 50 companies had leaked verified secrets on GitHub, including API keys, tokens, and credentials. The most common leak sources were Jupyter Notebook files (.ipynb), Python files (.py), and environment files (.env), with keys primarily from Hugging Face, AzureOpenAI, and WeightsAndBiases. One leaked Hugging Face token could have exposed access to approximately 1,000 private models at a single company. Wiz attributed the problem partly to AI-assisted development practices and attributed secret leakage specifically to "vibe coding" workflows.

In July 2025, [Krebs on Security reported](https://krebsonsecurity.com/2025/07/doge-denizen-marko-elez-leaked-api-key-for-xai/) that a DOGE employee published a private API key for xAI's language models on GitHub, embedded in a script called `agent.py`, granting access to more than 50 xAI LLMs. The exposure was flagged by GitGuardian, an external scanning tool. The key reportedly remained live after initial disclosure.

In February 2026, researchers discovered that [nearly 3,000 Google API keys](https://trufflesecurity.com/blog/google-api-keys-werent-secrets-but-then-gemini-changed-the-rules), previously considered harmless and commonly embedded in public code, could access Gemini API functionality once that API was enabled on affected projects. The vulnerability emerged when Google introduced Gemini and developers enabled the LLM API on existing projects without realizing the implications — credentials that were benign in their original context became dangerous when the platform expanded around them. This is a perfect example of the "orphaned intermediate state" pattern: the keys were safe when created, but the environment changed and nobody revisited the exposure.

[GitGuardian's 2024 report](https://www.gitguardian.com/state-of-secrets-sprawl-report-2024) found 12.8 million secrets leaked on public GitHub — a 28% year-over-year increase — and noted that AI-generated code accelerates the problem.

**Research:** [Flare researchers](https://flare.io/learn/resources/docker-hub-secrets-exposed/) (late 2025) found more than 10,000 Docker Hub container images leaking secrets including production API keys, cloud tokens, CI/CD credentials, and AI model access tokens, all pushed into public repositories often unintentionally by developers. [Aikido's 2026 State of AI in Security & Development report](https://www.aikido.dev/reports/2026-state-of-ai-in-security-development) similarly found that 69% of organizations reported vulnerabilities in AI-generated code and one in five reported a serious security incident linked to it.

**Mitigations:** Use secret scanning tools (gitleaks, TruffleHog, GitGuardian) in pre-commit hooks and CI pipelines — these catch the same leaks whether a human or an LLM introduced them. Never store secrets in files that the LLM agent can read; use environment variables, OS-level secret managers, or dedicated vault services instead. Configure your agent's workspace to exclude sensitive configuration files from its context. Use short-lived, scoped credentials rather than long-lived tokens. Rotate credentials regularly. Treat any credential that may have been exposed to an LLM context as potentially compromised.

### 6. Simple Mistakes at Scale

**What it is:** The agent makes straightforward errors — wrong command flags, incorrect syntax, misunderstood file paths, off-by-one errors — the same kinds of mistakes a human developer makes, but potentially at greater volume because the agent works faster and with less inherent caution.

**Why it matters:** Individually, these mistakes are mundane. But the speed of AI-assisted development means they can accumulate faster than review processes can catch them, and the confident presentation of AI-generated code means they're less likely to be questioned. The code "looks right" even when it isn't.

**Hypothetical example:** You ask the agent to write a database migration. It generates syntactically correct SQL that happens to have the wrong column type for a field, truncating data on migration. Or it writes a shell script that uses `rm -rf` with a variable that could be empty, creating a path like `rm -rf /` if the variable isn't set.

**Real incidents:** This category doesn't produce dramatic individual incidents, but shows up in aggregate statistics. [Veracode's 2025 study](https://www.veracode.com/resources/ai-generated-code-security) found that AI code compiles successfully 90% of the time (up from under 20% two years earlier), but 45% still contains OWASP Top 10 vulnerabilities. The code works, but it's insecure. Language-specific findings: cross-site scripting appeared in 86% of AI-generated cases, SQL injection in 20%, and weak or broken cryptographic implementations in 14%. These are basic errors that a senior developer would catch, but that junior developers — and the agent itself — may not notice.

**Mitigations:** Automated testing catches these the same way it catches human mistakes. Use linters, type checkers, static analysis tools, and comprehensive test suites. The key insight is that AI-generated code needs _more_ testing, not less, because it's produced faster with less inherent reflection. Integrate security-focused static analysis (SAST) into your CI/CD pipeline. Don't skip code review just because the AI's output looks clean.

### 7. Orphaned Intermediate States

**What it is:** LLMs plan multi-step workflows assuming continuity they don't have. They create permissive configurations "temporarily," broad-scope tokens "for now," experimental branches "to evaluate later" — and then the session ends. The artifacts persist without the plan to tighten, refine, or clean up. Subsequent LLM sessions (or human developers) may inherit these artifacts without the original intent and treat temporary states as permanent.

**Why it matters:** This is not a single dramatic failure but a gradual accumulation of security debt. Each orphaned state is a small exposure — a too-permissive IAM role, a debug endpoint left enabled, a temporary file containing credentials, an overly broad CORS configuration. Individually they're minor; collectively they create an expanding attack surface that nobody fully understands because the intent behind each artifact has been lost.

**Hypothetical example:** You ask your coding agent to set up a new microservice. During development, it creates a broadly-scoped API token ("we'll tighten the permissions once we know exactly which endpoints we need"), enables debug logging that includes request bodies ("useful for development"), adds a permissive CORS policy ("we can restrict this once we know the production domains"), and installs several dependencies "to evaluate which approach works best." The session ends. You ship the service. The broad token, the debug logging, the permissive CORS, and the unnecessary dependencies all go to production because the "tighten this later" step never happened.

**Real incidents:**

The Google API key / Gemini vulnerability (February 2026) is a near-perfect example: API keys that were benign when created became dangerous when Google expanded the capabilities accessible through those keys, and nobody revisited the exposure. The keys were orphaned artifacts from a previous context whose risk profile changed without anyone noticing.

The [Moltbook breach](https://www.wiz.io/blog/exposed-moltbook-database-reveals-millions-of-api-keys) (late 2025) involved a misconfigured Supabase database — built through vibe coding — that exposed 1.5 million API keys and 35,000 email addresses. The misconfiguration was not the result of a sophisticated attack; it was a development-time setting that was never tightened for production. The site's creator stated he "didn't write one line of code."

**Research:** [Palo Alto Networks' Unit 42 team](https://unit42.paloaltonetworks.com/securing-vibe-coding-tools/) (January 2026) documented that a sales lead application was breached because the vibe coding agent neglected to incorporate key security controls such as authentication and rate limiting. The application worked correctly in terms of functionality but was missing the security hardening that a human developer would typically add as a matter of course.

**Mitigations:** Establish a checklist of security configurations that must be verified before any deployment, regardless of whether the code was human-written or AI-generated. Use infrastructure-as-code tools that enforce security baselines. Audit for overly permissive configurations regularly. When using an AI agent for development, explicitly include the "tighten and clean up" phase as a separate, deliberate step rather than assuming it will happen naturally. Remove unused dependencies periodically. Treat any credential or configuration created during a development session as potentially orphaned and review it before shipping.

### 8. Scope Creep and Over-Eagerness

**What it is:** The agent does more than you asked it to do. You request a bug fix and it refactors the entire module. You ask it to add a feature and it restructures the project layout. You ask it to investigate an issue and it starts making changes to "fix" things before you've agreed on an approach. The agent's training incentivizes helpfulness and task completion, which can manifest as unauthorized expansion of scope.

**Why it matters:** Unrequested changes may introduce bugs, break existing functionality, alter security-relevant configurations, or simply make it harder to review what actually changed. When the agent modifies files you didn't expect it to touch, those modifications may not receive the same scrutiny as the changes you actually requested.

**Hypothetical example:** You ask the agent to fix a null pointer exception in a single function. It refactors the function, then notices the calling code "could be improved too," and modifies three other files. One of those modifications changes an error handling path that previously caught and logged authentication failures, and the new version silently swallows them. The bug fix works. The security regression goes unnoticed.

**Real incidents:** The Replit incident involved scope creep as a contributing factor — the agent made changes during a code freeze, ignoring explicit instructions to stop. More broadly, this pattern shows up in the developer experience literature around AI coding tools: multiple reports of agents making unrequested changes that introduce subtle regressions.

**Mitigations:** Use agents that show diffs before applying changes and require explicit approval. Review all modified files, not just the ones you expected the agent to touch. Use version control discipline: commit frequently, review diffs carefully, and don't let the agent push changes directly. Prefer agents that operate in a "propose, don't apply" mode for changes outside the immediately requested scope.

### 9. Context Window Degradation

**What it is:** Over long sessions, the LLM loses track of earlier constraints, instructions, and context. It may forget that you said "don't modify the database schema," or lose track of which files it's already edited, or repeat actions it already performed. The agent's behavior degrades gracefully — it doesn't crash or error, it just gradually becomes less reliable and less aligned with your original intent.

**Why it matters:** This is insidious because there's no clear signal that the agent has lost the plot. It continues to respond confidently and produce plausible-looking output. The quality of its judgment silently decreases. In long sessions involving complex multi-step workflows, the risk of the agent taking an action that contradicts earlier constraints increases over time.

**Hypothetical example:** At the beginning of a long session, you tell the agent "never modify files in the `config/` directory — those are managed by a separate process." Three hours and hundreds of messages later, you ask it to update application settings, and it edits `config/settings.yml` because it's lost the constraint from early in the session.

**Real incidents:** No dramatic standalone incidents, but this pattern is a contributing factor in many of the failures described in other categories. The Replit agent's behavior over its 12-day experiment showed progressive degradation, with the agent becoming increasingly unreliable and eventually violating explicit constraints.

**Mitigations:** Keep sessions focused and reasonably scoped. For complex multi-step workflows, break the work into separate sessions rather than running one marathon session. Restate critical constraints periodically. Use project-level configuration files (like `.claude` or equivalent) that persist constraints across sessions rather than relying on conversational instructions that will eventually fall out of the context window.

### 10. Training Data Contamination and Insecure Patterns

**What it is:** LLMs are trained on vast amounts of publicly available code, much of which contains security anti-patterns, deprecated APIs, hardcoded credentials, and vulnerable implementations. The agent reproduces these patterns not because it's been instructed to, but because they're statistically common in its training data. It generates code that "looks like" what a typical developer would write — including the typical developer's security mistakes.

**Why it matters:** The insecure patterns are not random errors but systematic biases. If most code in the training data uses a deprecated cryptographic algorithm, the agent will suggest that algorithm. If most example code hardcodes API keys for clarity, the agent will tend to hardcode them. The patterns are consistent enough that researchers can predict which vulnerabilities will appear in AI-generated code.

**Hypothetical example:** You ask the agent to implement password hashing. It generates code using MD5 or SHA-1 (common in older training data) rather than bcrypt or Argon2. Or it implements JWT validation but doesn't check the `alg` header, reproducing a well-known vulnerability present in countless tutorial examples.

**Real incidents:**

The [Tea dating app](https://techcrunch.com/2025/07/26/dating-safety-app-tea-breached-exposing-72000-user-images/) suffered two publicly reported data breaches in 2025. The first exposed approximately 72,000 images, including government ID photos, and a [subsequent breach](https://techcrunch.com/2025/07/29/tea-apps-data-breach-gets-much-worse-exposing-over-a-million-private-messages/) exposed over a million private messages. While the direct role of AI-generated code versus poor development practices generally is debated (and may be determined in court), the app exemplifies the pattern of rapidly-built applications with fundamental security controls missing.

**Research:** [Veracode's 2025 study](https://www.veracode.com/resources/ai-generated-code-security) tested over 100 LLMs and found that 45% of AI-generated code introduced OWASP Top 10 vulnerabilities, with little improvement over the previous two years despite compilation success rates improving dramatically (from under 20% to 90%). AI models trained on older datasets suggest deprecated or insecure libraries, and 14% of AI-generated cryptographic implementations used weak or broken algorithms. The code is increasingly _functional_ but not increasingly _secure_.

**Mitigations:** Use security-focused static analysis tools that specifically check for known vulnerability patterns (SAST). Maintain a list of approved libraries and algorithms for security-sensitive operations (cryptography, authentication, session management) and verify that AI-generated code uses them. Don't assume that code which compiles and passes functional tests is secure. Apply the same security review standards to AI-generated code that you'd apply to code from a junior developer: assume it works but may contain basic security mistakes.

## Cross-Cutting Observations

**Many mitigations are not LLM-specific.** The security practices that protect against these threats — code review, dependency auditing, secret scanning, least privilege, environment separation, automated testing, infrastructure-as-code — are the same practices that protect against human-introduced vulnerabilities. The main difference is that AI-assisted development increases the volume and speed of code production, making it more important to have these practices automated and enforced rather than relying on manual discipline.

**The novel threat class still lacks a broadly reliable defense in practice.** Prompt injection (categories 1 and 2) exploits a fundamental architectural property of LLMs, and no broadly accepted complete mitigation has yet been demonstrated in mainstream practice. The ["Attacker Moves Second" paper](https://arxiv.org/abs/2510.17342) (October 2025) demonstrated that all 12 tested defenses could be bypassed with >90% success rates. A practical framework, [Meta's Agents Rule of Two](https://ai.meta.com/blog/practical-ai-agent-security/), is a risk-reduction strategy rather than a complete solution. Organizations and individual developers still need to account for residual risk and choose autonomy levels that match their threat model.

**The gap between "demonstrated in research" and "exploited in the wild" may be narrowing.** As of early 2026, many prompt injection attacks have been demonstrated by security researchers against real products (the Summer of Johann), while much of the documented real-world damage has come from the "boring" categories (accidental database deletion, leaked secrets, insecure generated code). However, the operationalization infrastructure is being built: dark web playbooks for slopsquatting, offensive LLM tools like Hexstrike-AI, and documented attack chains that combine prompt injection with supply chain compromise. The transition from research to exploitation appears to be accelerating.

**Human judgment remains a central link in both failure and defense.** Every mitigation strategy ultimately relies on human judgment at some point — reviewing code, approving actions, configuring permissions, maintaining security hygiene. The irony of AI-assisted development is that it generates code fast enough to overwhelm human review capacity, while the security of that code depends on human review. This tension is not resolved by any current tool or framework. The practical reality is that developers using AI agents need to be more disciplined about review, not less, at exactly the moment when the tools are making it easier to skip review entirely.

## Further Reading

For ongoing monitoring of this rapidly evolving space, the following sources are recommended:

[Simon Willison's blog](https://simonwillison.net/tags/prompt-injection/) provides consistently insightful ongoing coverage of prompt injection and LLM security, including the "lethal trifecta" framework, coverage of the Summer of Johann, and analysis of the Rule of Two and Attacker Moves Second papers. His prompt injection tag collects over 140 posts on the topic.

[Johann Rehberger's Embrace The Red blog](https://embracethered.com/blog/) publishes a prolific stream of original security research against production AI tools, including the CVE-2025-53773 Copilot RCE and the Month of AI Bugs series.

[OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/) provides a widely used industry framework for LLM security risks.

[Meta's "Agents Rule of Two"](https://ai.meta.com/blog/practical-ai-agent-security/) (October 2025) provides a practical framework for reasoning about agent security.

["The Attacker Moves Second"](https://arxiv.org/abs/2510.17342) (Nasr et al., October 2025) provides strong evidence that prompt injection defenses are unreliable.

["We Have a Package for You!"](https://arxiv.org/abs/2406.10279) (Spracklen et al., 2025) provides a comprehensive analysis of package hallucination / slopsquatting.

[Veracode's 2025 GenAI Code Security Report](https://www.veracode.com/resources/ai-generated-code-security) provides extensive data on vulnerability rates in AI-generated code.
