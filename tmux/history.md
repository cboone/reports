---
created: 2026-01-27
---

# The Complete History of tmux: From OpenBSD Project to Industry Standard

_January 27, 2026_

**tmux, the terminal multiplexer that revolutionized command-line workflows, was created by Nicholas Marriott in 2007 as a cleaner, BSD-licensed alternative to GNU Screen.** What began as a personal project driven by frustration with Screen's unreadable codebase became one of the most influential developer tools of the 2010s. After OpenBSD founder Theo de Raadt personally audited the code and found it "high quality," tmux was imported into the OpenBSD base system on June 1, 2009—a rare honor that signaled production readiness. Today, with **40,000+ GitHub stars** and inclusion in virtually every Unix-like operating system, tmux has become the de facto standard for terminal multiplexing.

---

## Nicholas Marriott and the creation story

Nicholas Marriott, a software developer from Northern Ireland working in the financial industry on performance and networking systems, created tmux in **2007**. His favorite programming language was C, and his preferred operating system was OpenBSD—both factors that shaped tmux's design and eventual home.

In a 2009 interview with the OpenBSD Journal, Marriott explained his motivations directly:

> "I was a heavy screen user and I was vaguely unhappy with it. It was another of those programs with a lot of baggage—poor documentation, a strange configuration file and an unintuitive command-line interface. And that isn't mentioning the code."

The codebase problem was paramount. Marriott noted that **"screen's code is virtually unreadable"** and that **"code that was not write-only was a big goal"** for the new project. He wanted a terminal multiplexer that developers could actually extend and improve.

Development began with a quick prototype called **"nscr"** that implemented basic functions. A few months later, Marriott began fleshing out the prototype and published the first public version online in **late 2007**. The copyright notices in the source code confirm this timeline, showing `Copyright (c) 2007 Nicholas Marriott`.

### Key advantages over GNU Screen

Marriott designed tmux with several specific improvements over Screen:

- **Clearly-defined client-server model** where windows are independent entities attachable to multiple sessions
- **Consistent, well-documented command interface** with unified syntax for interactive use, key bindings, and shell scripting
- **Native vertical splitting** (Screen required special patches for this functionality)
- **Multiple paste buffers** and flexible window linking across sessions
- **BSD license** rather than GPL, making it suitable for OpenBSD's base system
- **Clean, modern, extensible codebase** that invited contributions

---

## The road to OpenBSD and version 1.0

tmux's path to mainstream adoption accelerated dramatically when it caught the attention of the OpenBSD community.

**May 31, 2008**: tmux 0.2 was imported into the OpenBSD ports system by brynet@, who had discovered the project on SourceForge. The CVS commit message read: *"import tmux 0.2 - tmux is a 'terminal multiplexer'... intended to be a simple, modern, BSD-licensed alternative to programs such as GNU screen."*

**June 1, 2009**: In a remarkable vote of confidence, tmux was imported directly into the **OpenBSD base system**—not as a port, but as part of the core operating system. This was a rare honor that reflected exceptional code quality. Theo de Raadt, OpenBSD's founder, personally audited the code:

> "The most impressive thing about tmux, in my view, is how frustrating the code audit was. In 2 hours, I found only one or two nits that had very minor security consequences. It was not accepted into the tree based on license alone. It is high quality code."

Marriott later reflected that he hadn't expected the inclusion: *"Paul Irofti brought tmux up at the last hackathon and they decided to import it. I felt tmux would be improved by being part of OpenBSD—it would add many more expert users and developers, a larger userbase, and a more rigorous development schedule."*

**September 20, 2009**: **Version 1.0** was released, marking tmux's first major stable milestone with customizable mode keys, UTF-8 support, alternate screen handling, copy mode searching, and a system-wide configuration file at `/etc/tmux.conf`.

**October 2009**: OpenBSD 4.6 became the first OpenBSD release to ship with tmux in its base system.

---

## Complete version history and milestones

The following timeline documents every major tmux release from inception through 2025:

### The foundational years (2007-2012)

| Version | Date | Significance |
|---------|------|--------------|
| 0.1 | November 17, 2007 | First public release |
| 0.8 | April 21, 2009 | Major pre-1.0 release |
| 0.9 | July 1, 2009 | UTF-8 improvements, switch to libevent |
| **1.0** | September 20, 2009 | First stable release; 256-color support |
| 1.1 | November 5, 2009 | `run-shell` command, `synchronize-panes` |
| 1.2 | March 10, 2010 | Rectangle copy, `capture-pane` command |
| 1.3 | July 18, 2010 | New input parser rewrite, tiled layout |
| 1.4 | December 27, 2010 | UTF-8 line drawing, AIX support |
| 1.5 | July 9, 2011 | xterm mouse modes 1002/1003, HP-UX port |
| 1.6 | January 23, 2012 | **Format strings framework** (`#{...}` syntax) |
| 1.7 | October 13, 2012 | Line continuation in config, bracketed-paste |

### The maturation period (2013-2016)

| Version | Date | Significance |
|---------|------|--------------|
| **1.8** | March 26, 2013 | **Control mode** for iTerm2 integration, pane zooming |
| 1.9 | February 20, 2014 | Major style option changes, Cygwin support |
| **2.0** | March 6, 2015 | utmp support, function keys F13-F63 |
| 2.1 | October 18, 2015 | **Mouse mode rewrite** (single `mouse` option) |
| 2.2 | April 10, 2016 | **True color (24-bit RGB)** support, hooks system |

### The modern era (2017-present)

| Version | Date | Significance |
|---------|------|--------------|
| 2.3 | September 29, 2016 | `pane-border-status`, command hooks |
| 2.4 | April 20, 2017 | **Major key table rewrite**, search highlighting |
| 2.6 | October 5, 2017 | Choose mode rewrite with preview/sorting |
| 2.9 | May 1, 2019 | **Multi-line status bar** (2-5 lines), `window-size` option |
| **3.0** | November 2019 | **yacc-based config parser**, menus (`display-menu`), pane options |
| 3.1 | April 26, 2020 | Regex search in copy mode, `~/.config/tmux/tmux.conf` |
| 3.2 | April 14, 2021 | **Popups** (`display-popup`), extended keys, customize mode |
| 3.3 | June 2022 | ACL for socket users, OSC 7/8 support, systemd socket activation |
| 3.4 | February 13, 2024 | **SIXEL graphics support**, OSC 133 shell prompts, hyperlinks |
| 3.5 | September 27, 2024 | **Scrollbars**, extended keys revamp, mirrored layouts |
| 3.6 | November 26, 2024 | Emoji improvements, clock seconds, codepoint-widths |
| 3.6a | December 5, 2024 | Latest patch release |

### Operating system adoption timeline

- **June 2009**: OpenBSD base system
- **October 2009**: OpenBSD 4.6 (first release with tmux)
- **October 2009**: FreeBSD ports (sysutils/tmux)
- **2010+**: Debian, Ubuntu, Fedora, Arch Linux repositories
- **Ongoing**: Available via Homebrew for macOS, pkgsrc for NetBSD

---

## Technical evolution and major features

tmux's architecture has remained fundamentally stable—a **client-server model** where the server manages sessions/windows/panes and clients connect via Unix sockets—but the feature set has evolved dramatically.

### Color support progression

Color capabilities expanded across three generations. **256-color support** arrived in version 1.0 (September 2009) via the `-2` flag and appropriate `default-terminal` settings. The transformative **true color (24-bit RGB)** support came in **tmux 2.2** (April 2016) through the `Tc` terminfo flag in `terminal-overrides`. Version 3.2 introduced the cleaner `terminal-features` option for specifying RGB capability.

### Mouse support transformation

Early tmux versions (1.0-2.0) required multiple options: `mouse-resize-pane`, `mouse-select-pane`, `mouse-select-window`, and `mode-mouse`. The **2.1 release** (October 2015) consolidated these into a single `mouse` option—an incompatible but welcomed simplification. Subsequent versions added double/triple-click support (2.4), improved pane resizing (2.8), and right-click context menus (3.0).

### The format strings revolution

Version 1.6 (January 2012) introduced the **format strings framework**, enabling the `#{variable}` syntax that transformed tmux scripting. This system evolved continuously:

- **2.2**: Modifiers like `#{t:}` (time), `#{b:}` (basename)
- **2.4**: Comparisons (`#{==:a,b}`)
- **3.0**: Numeric operators, regex support
- **3.2**: Math operators (+, -, *, /)

### Control mode for IDE integration

**Version 1.8** (March 2013) introduced **control mode** (`tmux -C`), enabling terminal emulators like iTerm2 to render tmux sessions using native GUI elements. This created seamless integration where users could benefit from tmux's session persistence while enjoying native tabs, scrolling, and copy/paste.

### Configuration parsing overhaul

The **3.0 release** (November 2019) represented a major architectural change: configuration file parsing was rewritten using **yacc(1)**, unifying the parser for both config files and runtime commands. This enabled the `{}` syntax for multi-line strings and more consistent command handling.

### Recent innovations

The most recent versions brought visual capabilities previously thought impossible in terminals:

- **3.2** (2021): Popup windows via `display-popup`
- **3.4** (2024): **SIXEL graphics support** for inline images
- **3.5** (2024): **Pane scrollbars** via `pane-scrollbars` option
- **3.6** (2024): Improved emoji and regional indicator handling

---

## Community adoption and the plugin ecosystem

tmux's rise from OpenBSD utility to industry standard occurred through several key inflection points.

### The Thoughtbot era (2011-2015)

The **"A tmux Crash Course"** blog post by Thoughtbot became arguably the most influential tmux tutorial ever written, appearing on Hacker News multiple times between 2011 and 2018. This single article introduced thousands of developers to tmux's capabilities.

In **February 2012**, Brian P. Hogan published **"tmux: Productive Mouse-Free Development"** through Pragmatic Programmers—the first book dedicated to tmux. A second edition covering tmux 2.3 followed in 2016, and a third edition covering modern features is in development.

Thoughtbot further cemented tmux's prominence by launching a **dedicated tmux course on Upcase** in 2015, taught by Chris Toomey, whose `vim-tmux-navigator` plugin had already become essential for vim users.

### The plugin ecosystem emerges

**Bruno Sutic** (GitHub: @bruno-) created the **Tmux Plugin Manager (TPM)** around 2014-2015, establishing a standardized plugin architecture. TPM now has **13,800+ GitHub stars** and supports tmux 1.9+.

The most popular plugins, all from the tmux-plugins organization:

- **tmux-resurrect** (~12,400 stars): Persist sessions across restarts
- **tmux-continuum** (~3,800 stars): Automatic saving/restoration
- **tmux-sensible**: Baseline settings everyone accepts
- **tmux-yank**: Clipboard integration
- **tmux-logging** (~1,200 stars): Session recording

### Editor integrations

The **vim-tmux-navigator** plugin by Chris Toomey (~5,900 GitHub stars), created around 2012-2013, enabled seamless navigation between vim splits and tmux panes using consistent Ctrl+h/j/k/l bindings. This pattern has been replicated for Neovim (nvim-tmux-navigation) and conceptually adapted for VS Code.

For Emacs users, **emamux** provides tmux manipulation capabilities, including sending commands, managing runner panes, and copying from tmux buffers.

Shell frameworks embraced tmux through plugins like the **oh-my-zsh tmux plugin**, which provides aliases and autostart features, and **tmuxinator**, a Ruby gem for defining complex session layouts in YAML.

---

## Primary sources and authoritative references

### Official repositories and documentation

- **GitHub repository**: https://github.com/tmux/tmux
- **Official website**: http://tmux.github.io/
- **GitHub Wiki**: https://github.com/tmux/tmux/wiki
- **OpenBSD CVS**: http://cvsweb.openbsd.org/cgi-bin/cvsweb/src/usr.bin/tmux/
- **OpenBSD man page**: https://man.openbsd.org/tmux
- **CHANGES file** (complete changelog): https://raw.githubusercontent.com/tmux/tmux/master/CHANGES

### Key interviews and historical documents

- **OpenBSD Journal interview with Nicholas Marriott** (July 12, 2009): https://undeadly.org/cgi?action=article;sid=20090712190402 — Primary source for creation motivations and early history
- **OpenBSD imports tmux announcement** (July 7, 2009): http://undeadly.org/cgi?action=article;sid=20090707041154 — Contains Theo de Raadt's code audit statement
- **LinuxTag 2012 speaker feature**: http://www.linuxtag.org/2012/en/program/speaker-features/featured/article/featured-nicholas-marriott-tmux.html

### Mailing lists

- **tmux-users Google Group**: https://groups.google.com/g/tmux-users
- **Mail Archive (searchable)**: https://www.mail-archive.com/tmux-users@googlegroups.com/

### Release information

- **GitHub Releases**: https://github.com/tmux/tmux/releases
- **FreeBSD ports**: https://www.freshports.org/sysutils/tmux
- **Debian package tracker**: https://tracker.debian.org/pkg/tmux

### Community resources

- **TPM (Tmux Plugin Manager)**: https://github.com/tmux-plugins/tpm
- **tmux-resurrect**: https://github.com/tmux-plugins/tmux-resurrect
- **vim-tmux-navigator**: https://github.com/christoomey/vim-tmux-navigator
- **awesome-tmux**: https://github.com/Morantron/awesome-tmux
- **Thoughtbot's tmux Crash Course**: https://thoughtbot.com/blog/a-tmux-crash-course

---

## Conclusion

tmux's trajectory from Nicholas Marriott's personal project in 2007 to the dominant terminal multiplexer reveals how exceptional code quality and clear design can overcome entrenched alternatives. The decision to import tmux into OpenBSD's base system—validated by Theo de Raadt's rigorous code audit—provided the credibility that accelerated adoption.

Three factors distinguished tmux's success: **technical superiority** over GNU Screen's aging codebase, **community investment** through influential tutorials and the plugin ecosystem, and **sustained development** with regular releases adding meaningful features without breaking compatibility. The recent additions of SIXEL graphics and scrollbars demonstrate that even after 17 years, tmux continues evolving to meet modern developer needs.

The original SourceForge repository is now historical; development has moved to GitHub where the project maintains an active ~6-month release cycle coordinated with OpenBSD releases. With control mode enabling native integration in tools like iTerm2, true color support for modern terminals, and a robust plugin ecosystem for customization, tmux has transcended its origins as "a better Screen" to become infrastructure that developers build their workflows upon.