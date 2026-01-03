# Documentation Index

This directory contains comprehensive documentation for Solo RPG Toolkit development and features.

## 📚 Documentation Files

### [CODEBASE_SUMMARY.md](CODEBASE_SUMMARY.md)

**Start here!** High-level overview of the entire codebase.

**Contents:**

- Repository structure
- Architecture overview
- Three-layer inline element system
- Key implementation patterns
- Common development tasks
- FAQ and troubleshooting

**Audience:** New developers, contributors wanting to understand the project

---

### [STAT_DIE_TRACKER_DESIGN.md](STAT_DIE_TRACKER_DESIGN.md)

Complete feature specification for the proposed Stat Die Tracker inline element.

**Contents:**

- Use case analysis
- Syntax design and rationale
- Parsing rules and regex patterns
- Rendering approach and DOM structure
- State update strategy
- Edge case handling
- Compatibility analysis
- Implementation checklist
- Alternative approaches considered

**Audience:** Feature designers, implementers, code reviewers

**Status:** Design complete, ready for implementation

---

### [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)

Step-by-step implementation guide for the Stat Die Tracker.

**Contents:**

- Phased approach (8 PRs/commits)
- Detailed implementation steps per phase
- Testing checklists
- Success criteria for each phase
- Timeline estimates (5-12 hours total)
- Risk assessment and mitigation
- Rollback plan

**Audience:** Implementers, project managers, code reviewers

**Ready to implement:** Yes, follow Phase 1-8 sequentially

---

## 📖 Other Documentation

### [../CONTRIBUTING.md](../CONTRIBUTING.md)

Developer guide for contributing to the project.

**Contents:**

- Getting started (setup, build, deploy)
- Complete architecture explanation
- Step-by-step: "How to Add a New Inline Element"
- Development workflow
- Testing guidelines
- Code style and conventions

**Audience:** All contributors

---

### [../README.md](../README.md)

User-facing documentation for plugin features.

**Contents:**

- Feature overview
- Inline elements documentation
- Installation instructions
- Usage examples

**Audience:** Plugin users, feature discovery

---

## 🎯 Quick Navigation

### I want to

**...understand the codebase**
→ Start with [CODEBASE_SUMMARY.md](CODEBASE_SUMMARY.md)

**...contribute code**
→ Read [../CONTRIBUTING.md](../CONTRIBUTING.md)

**...implement Stat Die Tracker**
→ Follow [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)

**...understand the feature design**
→ Read [STAT_DIE_TRACKER_DESIGN.md](STAT_DIE_TRACKER_DESIGN.md)

**...use the plugin**
→ See [../README.md](../README.md)

**...add a new inline element**
→ See "How to Add a New Inline Element" in [../CONTRIBUTING.md](../CONTRIBUTING.md)

---

## 🏗️ Architecture at a Glance

```
┌─────────────────────────────────────────────────────────┐
│                    Solo RPG Toolkit                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Main Plugin (src/main.ts)                             │
│  ├── Registers views (dice, deck, oracle, word)       │
│  ├── Registers inline element processors               │
│  └── Settings UI                                       │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │     Inline Elements System                      │  │
│  ├─────────────────────────────────────────────────┤  │
│  │                                                 │  │
│  │  Base Layer (src/inline/base/)                 │  │
│  │  ├── Core widget logic                         │  │
│  │  ├── State management                          │  │
│  │  ├── DOM generation                            │  │
│  │  └── Serialization                             │  │
│  │                                                 │  │
│  │  ┌──────────────────┐  ┌──────────────────┐   │  │
│  │  │  Live Preview    │  │  Reading Mode    │   │  │
│  │  │  (live/)         │  │  (read/)         │   │  │
│  │  ├──────────────────┤  ├──────────────────┤   │  │
│  │  │ CodeMirror 6     │  │ Post-processor   │   │  │
│  │  │ WidgetType       │  │ replaceInFile()  │   │  │
│  │  └──────────────────┘  └──────────────────┘   │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  Sidebar Views (src/view/)                             │
│  ├── Dice roller                                       │
│  ├── Card decks                                        │
│  ├── Oracle                                            │
│  └── Random generators                                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## 📊 Inline Elements

| Element | Syntax | Base File | Description |
|---------|--------|-----------|-------------|
| Count | `` `5` `` | `count.ts` | Simple counter |
| CountLimit | `` `1/5` `` | `countlimit.ts` | Counter with max |
| Track | `` `boxes: 1/5` `` | `track.ts` | Progress boxes/circles |
| Clock | `` `clock: 1/6` `` | `clock.ts` | Progress clock |
| Dice | `` `d6` `` | `dice.ts` | Rollable dice |
| Space | `` ` ` `` | `space.ts` | Tab stop |
| **Stat Die** | `` `sd6: 3` `` | `statdie.ts` | **Proposed** |

## 🔧 Development Commands

```bash
npm install          # Install dependencies
npm run build        # Build plugin
npm run dev          # Watch mode
npm run lint         # Type check and lint
npm run deploy       # Prepare for installation
```

## 📝 Notes

- All inline elements disabled by default (user opt-in via settings)
- Master toggle: Settings → Inline elements → Enable inline elements
- Each element follows three-layer architecture (base, live, read)
- State persists back to markdown source
- Works in both Live Preview and Reading modes

## 🤝 Contributing

1. Read [../CONTRIBUTING.md](../CONTRIBUTING.md)
2. Follow the step-by-step guide for adding inline elements
3. Test thoroughly in both modes
4. Update documentation
5. Submit PR with clear description

## 📧 Contact

- **Issues:** [GitHub Issues](https://github.com/Newman5/solo-toolkit/issues)
- **Email:** <kurowski.dev@gmail.com> (original author)

---

**Last Updated:** 2026-01-02
