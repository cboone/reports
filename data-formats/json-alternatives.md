---
created: 2026-01-25
---

# Human-readable alternatives to JSON: performance, syntax, and adoption in 2025

_January 25, 2026_

**JSON's dominance remains unchallenged for APIs, but TOML has emerged as the clear winner for configuration files** in modern toolchains. Performance benchmarks reveal that JSON parsers are consistently **5-650x faster** than human-readable alternatives—yet for config files parsed once at startup, this gap is irrelevant. The real driver of adoption is reduced syntax friction: TOML's explicit typing avoids YAML's infamous gotchas, while newer formats like **KDL** and **Pkl** (Apple, 2024) are gaining traction for document-heavy and type-safe configurations respectively.

The ecosystem has settled into distinct niches rather than crowning a single successor. YAML dominates DevOps (Kubernetes, GitHub Actions), TOML owns package management (Cargo.toml, pyproject.toml), and JSON holds firm for machine-to-machine APIs. Emerging formats target specialized needs: KDL for human-friendly documents, RON for Rust type safety, and Dhall for programmable configs that can never crash or hang.

## Performance benchmarks show JSON still reigns for speed

The performance gap between JSON and its alternatives is substantial but context-dependent. Native JSON parsers benefit from decades of optimization and, in JavaScript/Python, C-level implementations.

**Parsing speed comparison (Python):**
| Format | Time | vs. JSON |
|--------|------|----------|
| JSON (stdlib) | 1.5ms | baseline |
| YAML (CLoader) | 58ms | **38x slower** |
| TOML (tomli) | 163ms | **106x slower** |
| YAML (pure Python) | 995ms | **650x slower** |

**Rust serialization benchmarks** tell a similar story: serde_json achieves ~170ns serialize / ~240ns deserialize, while RON manages ~500ns / ~2,000ns (3-8x slower) and YAML crawls at ~3,000ns / ~7,000ns (18-30x slower). The Go ecosystem shows JSON parsing at **16,829 ns/op** with 1.3KB memory versus TOML at **91,145 ns/op** using 12.6KB—JSON is 5.4x faster with 9.5x less memory.

For Node.js, native `JSON.parse()` achieves **5,940 ops/sec** while the JSON5 library manages just 327 ops/sec—an **18x penalty** for human-friendly syntax. The JSON5 maintainers explicitly acknowledge that "extreme performance isn't a priority."

**Practical implication:** For config files under 1,000 lines, even a 100x slowdown means <10ms parsing time. Performance matters for high-throughput APIs (stick with JSON) but not for startup-time configuration loading.

## TOML and YAML lead the established alternatives

### TOML has captured the packaging ecosystem

TOML (Tom's Obvious Minimal Language) uses INI-style syntax with explicit types and **first-class datetime support**. Its simplicity—the spec fits on a few pages versus YAML's 80+—translates to faster parsing and fewer surprises.

```toml
# pyproject.toml example
[project]
name = "mypackage"
version = "1.2.3"
dependencies = ["requests >= 2.28"]

[tool.ruff]
line-length = 88
```

**Ecosystem adoption:** Python selected TOML over JSON ("not human-usable"), YAML ("overly complex"), and INI ("lacks standardization") for PEP 518/621. The `tomllib` module is now in Python's stdlib (3.11+). Rust's entire ecosystem runs on `Cargo.toml`. Hugo, Poetry, Black, Ruff, and Mypy all use TOML configs.

**Limitations:** Deeply nested data becomes verbose (requires explicit `[a.b.c.d]` section headers), inline tables can't span lines, and no `null` type exists. TOML is deliberately unsuitable for general data serialization—it's purpose-built for configuration.

### YAML remains dominant in DevOps despite its dangers

YAML's indentation-based syntax and powerful features (anchors, aliases, multiline strings) make it expressive but treacherous:

```yaml
# The infamous Norway problem (YAML 1.1)
countries: [DK, NO, SE]  # NO becomes boolean false!

# Version strings lose precision
version: 1.10  # Becomes float 1.1
```

The "Norway problem" alone has caused production incidents. Other gotchas include `~` parsing as null, tabs being forbidden, and arbitrary code execution via `!python/object` tags (always use `safe_load()`).

**Despite these issues, YAML is entrenched in DevOps**: Kubernetes manifests, Docker Compose, GitHub Actions, Ansible, Helm charts, AWS CloudFormation. The ecosystem lock-in is total—no format is displacing YAML in cloud-native tooling.

**Library quality varies dramatically:** Python's `PyYAML` with CLoader is 17x faster than pure Python. In Node.js, `js-yaml` takes 504ms where native JSON takes 94ms—but both dwarf `yaml@2` at 2,108ms.

### JSON5 and HJSON: minimal friction for JSON users

**JSON5** adds comments, trailing commas, unquoted keys, and single-quoted strings while remaining a strict superset of JSON:

```json5
{
  // Comments allowed
  unquoted: 'single quotes work',
  trailing: ['commas', 'ok',],
  hex: 0xDEADBEEF,
}
```

Adoption includes Chromium, Next.js, Babel, and Apple's native `JSONDecoder.allowsJSON5`. With **65M+ weekly npm downloads**, it's the safest choice for teams wanting "JSON but readable."

**HJSON** goes further with completely optional quotes and commas—maximally human-friendly but with a smaller ecosystem. It's ideal for files edited by non-developers but lacks formal standardization.

## Emerging formats target specialized needs

### KDL offers the most promising general-purpose syntax

KDL (KDL Document Language), finalized with v2.0 in December 2024, combines XML's document semantics with modern ergonomics:

```kdl
package {
  name my-pkg
  version "1.2.3"
  
  dependencies {
    lodash "^3.2.1" optional=#true
  }
}
```

**Key features:** Node-based structure, slashdash comments (`/-` comments out entire nodes), a CSS-inspired query language (KQL), and format-preserving parsing that roundtrips comments and whitespace.

**Adoption:** Zellij terminal multiplexer, Niri Wayland compositor, Microsoft's TypeScript DOM generator, the W3C/WHATWG Bikeshed spec processor. Library support spans **15+ languages** including Rust, JavaScript, Python, and Go with mature implementations.

### RON excels for Rust-native configuration

RON (Rusty Object Notation) mirrors Rust's type system exactly:

```ron
GameConfig(
    window_size: (800, 600),  // Tuple
    difficulty: Easy,          // Enum variant
    key_bindings: {           // Map (not struct)
        "up": Up,
    },
)
```

With **~2.8 million monthly downloads** on crates.io, RON dominates Rust game development (Bevy, Amethyst). However, it's deliberately Rust-centric—other language implementations are experimental at best.

### Dhall and Pkl bring programming to configuration

**Dhall** (2017) is "JSON + functions + types + imports" that's intentionally **not Turing-complete**—programs are guaranteed to terminate and cannot access the filesystem or network:

```dhall
let makeUser = \(name : Text) -> { home = "/home/${name}", shell = "/bin/bash" }
in [ makeUser "alice", makeUser "bob" ]
```

Use cases include dhall-kubernetes (typed K8s manifests) and complex infrastructure where DRY principles and static type checking prevent runtime surprises.

**Pkl** (Apple, February 2024) provides full IDE support with autocompletion, constraint validation, and **code generation to Swift, Go, Java, and Kotlin**. It outputs JSON/YAML/XML, making it a configuration transpiler rather than a standalone format. Early adoption signals are strong given Apple's backing.

## When to use each format in practice

**Configuration files in 2025:**
- **Rust projects:** TOML is mandatory (Cargo.toml)
- **Python projects:** TOML via pyproject.toml (PEP 518/621)
- **DevOps/Kubernetes:** YAML (ecosystem lock-in)
- **JavaScript:** JSON remains entrenched (package.json)
- **Human-edited with comments needed:** JSON5 for minimal learning curve, KDL for clean syntax
- **Complex configs with logic:** Dhall (safety guarantees) or Pkl (IDE tooling)

**APIs:**
- **Public REST APIs:** JSON (universal compatibility, browser native)
- **Internal microservices:** gRPC/Protocol Buffers for 30% size reduction and streaming
- **Mobile with complex queries:** GraphQL (still uses JSON transport)
- **Bandwidth-constrained:** MessagePack as a binary JSON drop-in

**Data storage:**
- **Structured logs:** JSON Lines (JSONL)
- **MongoDB:** BSON (transparent to users)
- **High-throughput interchange:** MessagePack (20-30% smaller than JSON)
- **Analytics pipelines:** Parquet/Avro with schema evolution

## Conclusion

The "JSON alternative" landscape has matured into stable niches rather than converging on a single winner. **TOML is the clear choice for new configuration file ecosystems**, backed by Python stdlib inclusion and Rust's success. **YAML's DevOps dominance is permanent** despite its footguns—the sunk cost of Kubernetes alone ensures its longevity. **JSON remains uncontested for APIs** where parsing speed and universal compatibility matter.

For teams frustrated with JSON's verbosity in human-edited files, **JSON5 offers the lowest-friction migration path**. Those willing to learn new syntax should evaluate **KDL for documents** and **TOML for structured configs**. Rust developers have an excellent native option in **RON**, while organizations managing complex infrastructure should consider **Dhall's safety guarantees** or **Pkl's Apple-backed IDE integration**.

The performance penalty of human-readable formats (5-650x slower than JSON) sounds alarming but is irrelevant for configuration files parsed once at startup. Choose based on **ecosystem fit** (what does your toolchain already use?), **syntax ergonomics** (will humans edit this?), and **failure modes** (can YAML's implicit typing break your deployments?).