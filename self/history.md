---
created: 2026-01-23
---

# Self: The invisible language that powers modern computing

_January 23, 2026_

Self is often cited as an influential programming language that many developers have not heard of. Designed in **1986** by **David Ungar** and **Randall B. Smith** at Xerox PARC, this prototype-based language pioneered several optimization techniques now central to JavaScript engines and Java virtual machines. Though Self did not achieve mainstream adoption, its DNA runs through V8, HotSpot, SpiderMonkey, and JavaScript's object model, making it a consequential "failure" in computing history.

The language emerged from frustration with Smalltalk's complexity. Ungar, fresh from his award-winning Berkeley dissertation on Smalltalk performance, joined Smith at PARC to explore a radical simplification: what if object-oriented programming didn't need classes at all?

## From PARC to Stanford to Sun: a research odyssey

Self's journey through three institutions shaped both the language and the engineers who would later transform the industry.  The **design phase at Xerox PARC (1985-1986)** established the conceptual foundations, with Smith bringing insights from his Alternate Reality Kit prototype system and Ungar contributing deep expertise in dynamic language implementation.

When Ungar moved to Stanford as an assistant professor, **implementation began in earnest (1987-1990)**. His graduate students, including **Craig Chambers** and **Urs Hölzle**, built the first working compiler in 1987  and published "Self: The Power of Simplicity" at OOPSLA that year. The paper was later recognized as an influential OOPSLA paper of that era. The team publicly released Self in 1990, demonstrating that a prototype-based language could achieve remarkable performance.

The **Sun Microsystems era (1991-1995)** brought additional talent, including **Lars Bak**  and Mario Wolczko. The team developed the Morphic UI framework  and released Version 4.0 in 1995 with a completely redesigned programming environment. But 1995 also marked Java's launch, and Sun redirected resources toward its new commercial priority.  The Self project officially ended that year.

## The technical shift: eliminating classes, influencing modern JIT

Self's core insight was that classes are not strictly necessary for object-oriented programming.  Objects inherit directly from other objects through **prototype delegation**—when a message is sent, the system searches the receiver for a matching slot, then recursively searches parent objects.  Creating new objects requires only **cloning** existing ones, not instantiating from abstract class descriptions. 

This simplicity created severe performance challenges that drove groundbreaking innovations. A key one was **maps** (now called "hidden classes" in V8): an implementation-level structure that transparently groups objects with identical slot layouts. Since objects cloned from the same prototype typically share structure, maps enabled class-like optimization without language-level classes. V8's documentation explicitly acknowledges: "This basic idea is not new—the prototype-based programming language Self used maps to do something similar." 

**Polymorphic inline caches (PICs)**, introduced by Hölzle, Chambers, and Ungar in 1991, solved the problem of call sites that encounter multiple receiver types. Rather than falling back to slow dictionary lookups, PICs generate stub routines that test receiver types and branch directly to cached methods. Published results showed clear median speedups on representative workloads, and the technique became standard in modern JavaScript engines.

The team's work on **adaptive optimization** proved highly influential. Urs Hölzle's 1994 PhD thesis demonstrated that compilers could use runtime type information collected by PICs to guide aggressive speculative optimization, then safely **deoptimize** when assumptions failed. This approach, compiling quickly at first and then recompiling hot code with full optimization, became part of the foundation for Java's HotSpot VM and major JavaScript engines.  

## Why Self remained a research curiosity

Despite its technical strengths, Self faced formidable adoption barriers. The system required approximately **32MB of RAM**—extraordinary when typical PCs had 8-16MB.   Its image-based development model, where programs existed as "snapshots" of memory rather than source files, confused developers trained in file-based workflows.  The GUI-intensive environment demanded substantial hardware and made text-based development awkward.

More fundamentally, prototype-based programming was simply unfamiliar. Industry and academia had converged on class-based object orientation, and Self's alternative paradigm required conceptual reorientation that few were willing to undertake.  When Sun cancelled the project in favor of Java, Self lost its institutional champion. By the time hardware caught up with Self's requirements, JavaScript and Java had captured mindshare.

The project didn't entirely disappear. Community members released Version 4.4 in 2010 with Linux support, and **Self 2024.1** shipped in August 2024  with FreeBSD and NetBSD support. The codebase remains active on GitHub, maintained by enthusiasts who recognize its historical significance.

## Legacy: powering modern runtimes invisibly

Self's influence flows through two channels: language design and virtual machine technology. **Brendan Eich** explicitly adopted Self's prototype model when creating JavaScript in 1995, later writing: "I'm not proud, but I'm happy that I chose Scheme-ish first-class functions and Self-ish prototypes as the main ingredients."   JavaScript's prototype-based inheritance model traces directly to Self.

**NewtonScript** (1993) adapted Self for Apple's Newton PDA, demonstrating that prototype-based programming could work on resource-constrained devices.   **Lua**'s metatable mechanism enables Self-style delegation.  **Io**, **Slate**, and numerous research languages carried forward the prototype paradigm.  

The VM influence runs even deeper. After Sun cancelled Self, Hölzle and Bak founded Animorphic to build Strongtalk, a high-performance Smalltalk using Self techniques. **Sun acquired Animorphic in 1997**, and the technology became the **Java HotSpot VM** released in 1999.  When Google hired Lars Bak in 2006 to build Chrome's JavaScript engine, he carried forward many Self-inspired optimizations into **V8**.  Hidden classes, inline caches, adaptive optimization, and deoptimization, the techniques that make JavaScript fast enough to power modern web applications, were all advanced significantly in Self research.

## Key academic papers and resources

The landmark paper "Self: The Power of Simplicity" by Ungar and Smith (OOPSLA 1987, revised 1991) is available at https://bibliography.selflanguage.org/_static/self-power.pdf. The maps concept appears in "An Efficient Implementation of Self" by Chambers, Ungar, and Lee (OOPSLA 1989) at https://bibliography.selflanguage.org/_static/implementation.pdf.

Polymorphic inline caches are detailed in "Optimizing Dynamically-Typed Object-Oriented Languages With Polymorphic Inline Caches" by Hölzle, Chambers, and Ungar (ECOOP 1991) at https://bibliography.selflanguage.org/_static/pics.pdf. Hölzle's comprehensive PhD thesis "Adaptive Optimization for Self" (Stanford 1994) covers type feedback and deoptimization at https://bibliography.selflanguage.org/_static/urs-thesis.pdf.

The official Self bibliography at https://bibliography.selflanguage.org/ provides the complete collection of papers with PDF links.

## Conclusion

Self represents a fascinating paradox: a language that "failed" commercially yet became highly influential in modern computing. The techniques Ungar, Smith, Chambers, Hölzle, and Bak developed at PARC, Stanford, and Sun are now embedded in mainstream JavaScript and JVM runtimes, as well as in modern JIT compiler design generally.  Self's vision of "simplicity, uniformity, concreteness, and liveness" lives on not in Self programs but in infrastructure that powers a large share of modern software.

The language also demonstrated that radical simplification can drive innovation. By eliminating classes, Self forced its implementers to invent new optimization strategies—strategies that proved more powerful and general than the class-based approaches they replaced. Self's lesson for language designers: constraints can catalyze creativity, and today's research curiosity may become tomorrow's invisible foundation.
