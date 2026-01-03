# Solo RPG Toolkit - Codebase Analysis Summary

This document provides a high-level overview of the Solo RPG Toolkit codebase and serves as an entry point for understanding the plugin architecture.

## Quick Links

- **[README.md](../README.md)** - User documentation, features, and usage
- **[CONTRIBUTING.md](../CONTRIBUTING.md)** - Developer guide, architecture, how to add features
- **[STAT_DIE_TRACKER_DESIGN.md](STAT_DIE_TRACKER_DESIGN.md)** - Complete feature specification for proposed Stat Die Tracker
- **[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)** - Step-by-step implementation guide

## What is Solo RPG Toolkit?

An Obsidian plugin that provides tools for solo TTRPG gameplay, including:

- Dice roller with inline dice buttons
- Card decks (standard and custom)
- Oracle systems
- Random word generators
- Custom random tables
- **Inline elements** for tracking stats and progress in notes

## Repository Structure

```text
solo-toolkit/
├── src/
│   ├── main.ts                    # Plugin entry point
│   ├── settings/                  # Plugin settings and UI
│   ├── inline/                    # Inline element system ⭐
│   │   ├── base/                 # Core widget logic (shared)
│   │   ├── live/                 # Live Preview mode (CodeMirror 6)
│   │   └── read/                 # Reading mode (post-processor)
│   ├── view/                      # Sidebar panel views
│   │   ├── dice/                 # Dice roller view
│   │   ├── deck/                 # Card deck view
│   │   ├── oracle/               # Oracle view
│   │   └── word/                 # Random generator view
│   ├── utils/                     # Utility functions
│   ├── icons/                     # Custom icon registration
│   └── styles/                    # SCSS stylesheets
├── docs/                          # Documentation
│   ├── STAT_DIE_TRACKER_DESIGN.md
│   └── IMPLEMENTATION_PLAN.md
├── README.md                      # User documentation
├── CONTRIBUTING.md                # Developer guide
├── manifest.json                  # Plugin metadata
└── package.json                   # Dependencies and scripts
```

## Core Architecture: Inline Elements System

The inline elements are the most distinctive feature of this plugin. They transform code blocks in markdown into interactive widgets.

### Three-Layer Architecture

```text
┌─────────────────────────────────────────────────────┐
│  Base Layer (src/inline/base/)                      │
│  - Core logic, parsing, state management            │
│  - DOM generation                                    │
│  - Serialization (getText())                        │
└──────────────────┬──────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
┌───────▼──────────┐  ┌──────▼──────────┐
│ Live Layer       │  │ Read Layer      │
│ (live/)          │  │ (read/)         │
│                  │  │                 │
│ CodeMirror 6     │  │ Post-processor  │
│ WidgetType       │  │ replaceInFile() │
│ view.dispatch()  │  │                 │
└──────────────────┘  └─────────────────┘
```

### How Inline Elements Work

1. **User types code block:** `` `d6` ``
2. **Plugin detects pattern:** Regex matches in live or read mode
3. **Widget created:** Wraps base logic with view-specific behavior
4. **DOM rendered:** Interactive element replaces code block
5. **User interacts:** Click, adjust values
6. **State updated:** Widget updates internal state
7. **Document persisted:** Markdown source updated with new value

### Existing Inline Elements

| Type | Syntax | Description |
|------|--------|-------------|
| **Count** | `` `5` `` | Simple counter with +/- buttons |
| **CountLimit** | `` `1/5` `` | Counter with maximum value |
| **Track** | `` `boxes: 1/5` `` | Clickable boxes for progress |
| **Clock** | `` `clock: 1/6` `` | Circular clock for progress |
| **Dice** | `` `d6` `` | Rollable dice button |
| **Space** | `` ` ` `` | Adjustable tab stop |

All elements support:

- Colors (red, orange, yellow, green, cyan, blue, purple, pink)
- Custom sizes (small, large, or pixel values)
- Alternative separators (comma or pipe)

### Master Toggle

All inline elements are controlled by a single setting:

- `Settings → Inline elements → Enable inline elements`
- Default: **OFF** (user must opt-in)
- When disabled, code blocks render as plain code

## Key Implementation Patterns

### Pattern 1: Regex Matching

Each inline element has two regex patterns:

```typescript
// Exact match (for validation)
export const COUNT_REGEX = /^`[+-]?\d+`$/;

// Global match (for finding all instances)
export const COUNT_REGEX_G = /`[+-]?\d+`/g;
```

### Pattern 2: Base Widget

All base widgets implement the `BaseWidget` interface:

```typescript
export interface BaseWidget {
  getText: (wrap: string) => string;    // Serialize to markdown
  generateDOM: (opts: DomOptions) => void;  // Create DOM elements
}
```

### Pattern 3: Live Preview Widget

Wraps base widget with CodeMirror 6 integration:

```typescript
export class CountWidget extends WidgetType {
  base: CountWidgetBase;
  node: SyntaxNode;
  
  toDOM(view: EditorView): HTMLElement {
    this.base.generateDOM({
      onChange: () => this.updateDoc(view)
    });
    return this.base.el;
  }
  
  updateDoc(view: EditorView) {
    view.dispatch({
      changes: [{
        from: this.node.from,
        to: this.node.to,
        insert: this.base.getText("`")
      }]
    });
  }
}
```

### Pattern 4: Reading Mode Widget

Wraps base widget with post-processor:

```typescript
export class CountWidget {
  base: CountWidgetBase;
  // ... file and position info
  
  toDOM(): HTMLElement {
    this.base.generateDOM({
      onChange: () => this.updateDoc()
    });
    return this.base.el;
  }
  
  updateDoc() {
    replaceInFile({
      vault: this.app.vault,
      file: this.file,
      regex: COUNT_REGEX_G,
      lineStart: this.lineStart,
      lineEnd: this.lineEnd,
      newValue: this.base.getText("`"),
      replaceIndex: this.index
    });
  }
}
```

## Development Workflow

### Setup

```bash
# Clone and install
git clone https://github.com/Newman5/solo-toolkit.git
cd solo-toolkit
npm install

# Build
npm run build

# Deploy to test vault
npm run deploy
# Then copy dist/solo-rpg-toolkit to your vault's .obsidian/plugins/
```

### Development Mode

```bash
# Watch mode (auto-rebuild)
npm run dev
```

### Testing

1. Make changes to source files
2. Build: `npm run build`
3. Reload Obsidian (Ctrl+R / Cmd+R)
4. Test in a note
5. Check console for errors (Ctrl+Shift+I / Cmd+Opt+I)

## Common Tasks

### Adding a New Inline Element

See **[CONTRIBUTING.md](../CONTRIBUTING.md)** for complete step-by-step guide.

**Summary:**

1. Create base widget in `src/inline/base/[name].ts`
2. Create live widget in `src/inline/live/[name].ts`
3. Create read widget in `src/inline/read/[name].ts`
4. Register in `src/inline/live/index.ts` and `src/inline/read/index.ts`
5. Add styles to `src/styles.scss`
6. Update documentation

### Understanding a Feature

1. Start with the **base widget** to understand core logic
2. Look at **regex pattern** to see what it matches
3. Check **live widget** for Live Preview behavior
4. Check **read widget** for Reading mode behavior
5. Find **styles** in `src/styles.scss`

### Debugging Issues

1. Check browser console for errors
2. Verify regex matches expected pattern
3. Test in both Live Preview and Reading modes
4. Check if settings toggle is enabled
5. Try in fresh note to isolate issue

## Proposed Feature: Stat Die Tracker

A new inline element combining counters and dice rolling, designed for TTRPG systems like Cortex Plus and Savage Worlds.

**Syntax:** `sd6: 3` (stat die d6, current value 3)

**Rendering:** `[d6] [-] 3 [+]`

**See:**

- **[STAT_DIE_TRACKER_DESIGN.md](STAT_DIE_TRACKER_DESIGN.md)** for complete specification
- **[IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md)** for implementation guide

## Resources

### Official Documentation

- [Obsidian Plugin Developer Docs](https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- [CodeMirror 6 Documentation](https://codemirror.net/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

### Repository Files

- `manifest.json` - Plugin metadata (version, author, min Obsidian version)
- `package.json` - npm dependencies and build scripts
- `tsconfig.json` - TypeScript configuration
- `esbuild.config.mjs` - Build configuration

### Build Tools

- **esbuild** - Fast JavaScript bundler
- **esbuild-sass-plugin** - SCSS compilation
- **TypeScript** - Type checking and compilation
- **ESLint** - Code linting

## Frequently Asked Questions

### Q: Why are inline elements disabled by default?

**A:** To avoid surprising users. They opt-in through settings.

### Q: Can I use inline elements outside code blocks?

**A:** No, they must be in backtick code blocks (`` `like this` ``).

### Q: Do inline elements work on mobile?

**A:** The plugin claims mobile support (isDesktopOnly: false), but test thoroughly.

### Q: Can I add custom colors beyond the preset ones?

**A:** Yes, hex colors work: `` `d6,#fb464c: 4` ``

### Q: How do I prevent accidental edits?

**A:** Use the lock feature for dice: `` `!d6: 4` ``

### Q: Can I have multiple inline elements on the same line?

**A:** Yes! Example: `STR:`sd6: 4` DEX: `sd8: 5` CON: `sd6: 3``

### Q: What's the difference between `1/5` and `boxes: 1/5`?

**A:**

- `` `1/5` `` = Counter with limit, shows "+/- 1 / 5" buttons
- `` `boxes: 1/5` `` = Progress tracker, shows 5 clickable boxes

## Getting Help

- **Issues:** [GitHub Issues](https://github.com/Newman5/solo-toolkit/issues)
- **Discussions:** Use GitHub Discussions for questions and ideas
- **Email:** <kurowski.dev@gmail.com> (original author)

## Contributing

We welcome contributions! Please:

1. Read **[CONTRIBUTING.md](../CONTRIBUTING.md)**
2. Fork the repository
3. Create a feature branch
4. Make your changes
5. Test thoroughly
6. Submit a pull request

## License

MIT License - See LICENSE file

---

**Last Updated:** 2026-01-02

**Current Version:** 0.7.12

**Plugin ID:** solo-rpg-toolkit

**Minimum Obsidian Version:** 1.5.11
