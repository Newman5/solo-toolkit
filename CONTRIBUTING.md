# Contributing to Solo RPG Toolkit

Thank you for your interest in contributing to Solo RPG Toolkit! This guide will help you understand the codebase structure and how to add new features.

## Table of Contents

- [Getting Started](#getting-started)
- [Codebase Architecture](#codebase-architecture)
- [Key Files and Their Responsibilities](#key-files-and-their-responsibilities)
- [How to Add a New Inline Element](#how-to-add-a-new-inline-element)
- [Development Workflow](#development-workflow)
- [Testing Your Changes](#testing-your-changes)
- [Code Style Guidelines](#code-style-guidelines)

## Getting Started

### Prerequisites

- Node.js v18 or higher
- npm
- A test Obsidian vault for development

### Initial Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/Newman5/solo-toolkit.git
   cd solo-toolkit
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Build the plugin:
   ```bash
   npm run build
   ```

4. Deploy to your test vault:
   ```bash
   npm run deploy
   # Then copy dist/solo-rpg-toolkit to your vault's .obsidian/plugins/ folder
   ```

5. Enable the plugin in Obsidian Settings → Community Plugins

### Development Commands

- `npm run dev` - Watch mode: automatically rebuilds on file changes
- `npm run build` - Production build with type checking
- `npm run lint` - Run ESLint type checker
- `npm run deploy` - Build and prepare plugin folder for installation

## Codebase Architecture

The plugin is organized into several main modules:

```
src/
├── main.ts                  # Plugin entry point
├── settings/                # Plugin settings and UI
│   └── index.ts
├── inline/                  # Inline element system
│   ├── base/               # Base widget classes (shared logic)
│   ├── live/               # Live Preview mode (CodeMirror 6)
│   └── read/               # Reading mode
├── view/                    # Sidebar panel views
│   ├── dice/               # Dice roller view
│   ├── deck/               # Card deck view
│   ├── oracle/             # Oracle view
│   └── word/               # Random word generator view
├── utils/                   # Utility functions
│   ├── dice.ts             # Dice rolling logic
│   ├── plugin.ts           # File manipulation helpers
│   └── helpers/
├── icons/                   # Custom icon registration
└── styles/                  # SCSS stylesheets
```

### Module Responsibilities

#### Main Plugin (`src/main.ts`)
- Entry point for the plugin
- Registers views, commands, and settings
- Initializes inline element processors for Live Preview and Reading modes
- Manages plugin lifecycle (load/unload)

#### Settings (`src/settings/index.ts`)
- Defines `SoloToolkitSettings` interface
- Creates settings UI with toggles and dropdowns
- Key setting: `inlineCounters` - master toggle for all inline elements

#### Inline Elements System

The inline elements follow a three-layer architecture:

**1. Base Layer (`src/inline/base/`)** - Shared logic
- Contains base widget classes that handle:
  - Parsing code block text into structured data
  - Generating DOM elements
  - Managing internal state
  - Providing `getText()` method to serialize back to markdown
- Each inline element type has:
  - A regex pattern to match code blocks (e.g., `COUNT_REGEX`)
  - A base widget class (e.g., `CountWidgetBase`)
  - Common interface: `BaseWidget` with `getText()` and `generateDOM()`

**2. Live Preview Layer (`src/inline/live/`)** - CodeMirror 6 integration
- Wraps base widgets for Live Preview mode
- Uses CodeMirror 6 `WidgetType` API
- Handles document updates by dispatching changes to the editor
- Widgets are registered in `src/inline/live/index.ts`

**3. Reading Mode Layer (`src/inline/read/`)** - Post-processor
- Wraps base widgets for Reading mode
- Uses Obsidian's Markdown post-processor API
- Handles document updates using `replaceInFile` utility
- Registered as post-processor in `src/inline/read/index.ts`

#### View System (`src/view/`)
- Sidebar panel with tabs for dice, decks, oracle, and word generators
- Each view is independent and can be selected via settings

## Key Files and Their Responsibilities

### Plugin Core

| File | Purpose |
|------|---------|
| `src/main.ts` | Plugin initialization, view registration, command setup |
| `src/settings/index.ts` | Settings interface and UI configuration |

### Inline Elements

| File | Purpose |
|------|---------|
| `src/inline/base/types.ts` | Shared interfaces (`BaseWidget`, `DomOptions`) |
| `src/inline/base/count.ts` | Counter widget (`\`5\``) base logic |
| `src/inline/base/countlimit.ts` | Counter with limit (`\`1/5\``) base logic |
| `src/inline/base/track.ts` | Progress boxes/circles base logic |
| `src/inline/base/clock.ts` | Progress clock base logic |
| `src/inline/base/dice.ts` | Dice roller widget base logic |
| `src/inline/base/space.ts` | Tab stop widget base logic |
| `src/inline/base/shared.ts` | Shared utilities (menus, colors) |
| `src/inline/live/index.ts` | Live Preview registration and syntax tree traversal |
| `src/inline/read/index.ts` | Reading mode post-processor registration |

### Utilities

| File | Purpose |
|------|---------|
| `src/utils/plugin.ts` | `replaceInFile()` - updates markdown source |
| `src/utils/dice.ts` | Dice rolling algorithms |
| `src/utils/helpers/` | Various helper functions |
| `src/utils/index.ts` | Exports commonly used utilities |

### Styles

| File | Purpose |
|------|---------|
| `src/styles.scss` | Main stylesheet with component styles |

## How to Add a New Inline Element

This step-by-step guide shows how to create a new inline element. We'll use a hypothetical "stat die tracker" as an example.

### Step 1: Define the Regex Pattern

First, decide on your syntax. For example, `sd6: 2` for a d6 stat die with current value 2.

Create a regex that matches your pattern. Be specific to avoid conflicts with existing patterns.

### Step 2: Create the Base Widget Class

Create a new file in `src/inline/base/` (e.g., `statdie.ts`):

```typescript
import { setIcon } from "obsidian";
import { BaseWidget, DomOptions } from "./types";
import { nrandom } from "src/utils";

// Regex to match: `sd6: 2` or `sd10,red: 3`
export const STATDIE_REGEX = /^`sd(4|6|8|10|12|20)([|,]#?[\w\d]+)*: ?[+-]?\d+`$/;
export const STATDIE_REGEX_G = /`sd(4|6|8|10|12|20)([|,]#?[\w\d]+)*: ?[+-]?\d+`/g;

export class StatDieWidgetBase implements BaseWidget {
  dieSize: number;   // 4, 6, 8, 10, 12, 20
  value: number;     // Current value
  color: string;     // Optional color

  el: HTMLElement;
  dieEl: HTMLElement;
  minusEl: HTMLElement;
  valueEl: HTMLElement;
  plusEl: HTMLElement;

  constructor(opts: { originalText: string }) {
    this.parseValue(opts.originalText);
  }

  private parseValue(text: string) {
    // Remove backticks and split by colon
    const normalized = text.replace(/^`+|`+$/g, "");
    const [control, value] = normalized.split(":");
    
    // Parse die size
    const match = control.match(/sd(\d+)([|,]#?[\w\d]+)*/);
    this.dieSize = parseInt(match?.[1] || "6");
    
    // Parse color if present
    const colorMatch = control.match(/[|,](#?[\w\d]+)/);
    this.color = colorMatch?.[1] || "";
    
    // Parse current value
    this.value = parseInt(value?.trim() || "1");
    
    // Clamp value to valid range
    if (this.value < 1) this.value = 1;
    if (this.value > this.dieSize) this.value = this.dieSize;
  }

  private roll() {
    // Roll 1d(dieSize)
    this.value = nrandom(1, 1, this.dieSize, this.value);
    this.valueEl.innerText = this.value.toString();
  }

  private addValue(delta: number) {
    this.value += delta;
    if (this.value < 1) this.value = 1;
    if (this.value > this.dieSize) this.value = this.dieSize;
    this.valueEl.innerText = this.value.toString();
  }

  getText(wrap = ""): string {
    return [
      wrap,
      "sd",
      this.dieSize,
      this.color ? `,${this.color}` : "",
      ": ",
      this.value,
      wrap,
    ].join("");
  }

  generateDOM({ onFocus, onChange }: DomOptions) {
    this.el = document.createElement("span");
    this.el.classList.add("srt-statdie");

    // Die button (clickable to roll)
    this.dieEl = this.el.createEl("button");
    this.dieEl.classList.add("clickable-icon", "srt-statdie-btn");
    this.dieEl.innerText = `d${this.dieSize}`;
    
    this.dieEl.onclick = (event) => {
      event.preventDefault();
      event.stopPropagation();
      this.roll();
      onChange?.();
    };

    // Minus button
    this.minusEl = this.el.createEl("button");
    this.minusEl.classList.add("clickable-icon");
    setIcon(this.minusEl, "minus");
    
    this.minusEl.onclick = (event) => {
      event.preventDefault();
      event.stopPropagation();
      this.addValue(-1);
      onChange?.();
    };

    // Value display
    this.valueEl = this.el.createEl("span");
    this.valueEl.innerText = this.value.toString();
    this.valueEl.onclick = () => onFocus?.();

    // Plus button
    this.plusEl = this.el.createEl("button");
    this.plusEl.classList.add("clickable-icon");
    setIcon(this.plusEl, "plus");
    
    this.plusEl.onclick = (event) => {
      event.preventDefault();
      event.stopPropagation();
      this.addValue(1);
      onChange?.();
    };

    // Apply color if set
    if (this.color) {
      // Apply color styling to die button
      this.dieEl.style.color = this.color;
    }
  }
}
```

### Step 3: Export the Base Widget

Add export to `src/inline/base/index.ts`:

```typescript
export * from "./statdie";
```

### Step 4: Create Live Preview Widget

Create `src/inline/live/statdie.ts`:

```typescript
import { SyntaxNode } from "@lezer/common";
import { EditorView, WidgetType } from "@codemirror/view";
import { StatDieWidgetBase, STATDIE_REGEX } from "../base";
import { focusOnNode } from "./shared";

export { STATDIE_REGEX };

export class StatDieWidget extends WidgetType {
  base: StatDieWidgetBase;
  node: SyntaxNode;

  constructor(opts: { originalNode: SyntaxNode; originalText: string }) {
    super();
    this.base = new StatDieWidgetBase(opts);
    this.node = opts.originalNode;
  }

  updateDoc(view: EditorView) {
    view.dispatch({
      changes: [
        {
          from: this.node.from,
          to: this.node.to,
          insert: this.base.getText("`"),
        },
      ],
    });
  }

  toDOM(view: EditorView): HTMLElement {
    this.base.generateDOM({
      onFocus: () => focusOnNode(view, this.node),
      onChange: () => this.updateDoc(view),
    });
    return this.base.el;
  }
}
```

### Step 5: Create Reading Mode Widget

Create `src/inline/read/statdie.ts`:

```typescript
import { TFile, App } from "obsidian";
import { replaceInFile } from "src/utils/plugin";
import { StatDieWidgetBase, STATDIE_REGEX, STATDIE_REGEX_G } from "../base";

export { STATDIE_REGEX };

export class StatDieWidget {
  base: StatDieWidgetBase;
  app: App;
  file: TFile;
  lineStart: number;
  lineEnd: number;
  index: number;

  constructor(opts: {
    app: App;
    file: TFile;
    lineStart: number;
    lineEnd: number;
    index: number;
    originalText: string;
  }) {
    this.base = new StatDieWidgetBase(opts);
    this.app = opts.app;
    this.file = opts.file;
    this.lineStart = opts.lineStart;
    this.lineEnd = opts.lineEnd;
    this.index = opts.index;
  }

  updateDoc() {
    replaceInFile({
      vault: this.app.vault,
      file: this.file,
      regex: STATDIE_REGEX_G,
      lineStart: this.lineStart,
      lineEnd: this.lineEnd,
      newValue: this.base.getText("`"),
      replaceIndex: this.index,
    });
  }

  toDOM(): HTMLElement {
    this.base.generateDOM({
      onChange: () => this.updateDoc(),
    });
    return this.base.el;
  }
}
```

### Step 6: Register in Live Preview

Update `src/inline/live/index.ts` to import and check for your widget:

```typescript
import { StatDieWidget, STATDIE_REGEX } from "./statdie";

// Inside the buildDecorations method, add:
// `sd6: 2`
if (STATDIE_REGEX.test(text)) {
  buildMeta.push(meta);
  builder.add(
    from,
    to,
    Decoration.replace({
      widget: new StatDieWidget({
        originalNode: node.node,
        originalText: text,
      }),
    })
  );
}
```

### Step 7: Register in Reading Mode

Update `src/inline/read/index.ts` to import and check for your widget:

```typescript
import { StatDieWidget, STATDIE_REGEX } from "./statdie";

// Inside the postprocessor function, add an index tracker:
const indexMap = {
  // ... existing types
  statdie: 0,
};

// Add the check in the loop:
// `sd6: 2`
if (STATDIE_REGEX.test(mdText)) {
  const widget = new StatDieWidget({
    app: plugin.app,
    file,
    lineStart,
    lineEnd,
    index: indexMap.statdie++,
    originalText: mdText,
  });
  node.replaceWith(widget.toDOM());
}
```

### Step 8: Add Styles

Add styles to `src/styles.scss`:

```scss
.srt-statdie {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  
  .srt-statdie-btn {
    font-size: 0.85em;
    font-weight: 600;
  }
}
```

### Step 9: Test Your Changes

1. Build the plugin: `npm run build`
2. Deploy to your test vault: `npm run deploy`
3. Reload Obsidian (Ctrl+R or Cmd+R)
4. Enable inline elements in settings
5. Test in a note:
   ```
   Strength: `sd6: 3`
   Dexterity: `sd8: 5`
   ```

### Step 10: Update Documentation

Update `README.md` to document your new inline element with:
- Syntax examples
- Supported parameters
- Interaction behavior

## Development Workflow

### Making Changes

1. **Create a feature branch:**
   ```bash
   git checkout -b feature/my-feature
   ```

2. **Make your changes** in small, focused commits

3. **Test thoroughly** in both Live Preview and Reading modes

4. **Build and lint:**
   ```bash
   npm run lint
   npm run build
   ```

5. **Update documentation** if you've added or changed features

### Code Organization Guidelines

- Keep base widget logic separate from view-specific code
- Use the existing `BaseWidget` interface for consistency
- Reuse utilities from `src/utils/` when possible
- Follow the naming convention: `[Type]Widget` for widgets, `[TYPE]_REGEX` for patterns
- Keep regex patterns strict to avoid false matches

## Testing Your Changes

### Manual Testing Checklist

When adding or modifying inline elements, test:

- [ ] **Live Preview Mode**
  - [ ] Widget renders correctly
  - [ ] Interactions update the widget
  - [ ] Changes persist to markdown source
  - [ ] Multiple instances on same line work
  - [ ] Cursor focus/edit mode works

- [ ] **Reading Mode**
  - [ ] Widget renders correctly
  - [ ] Interactions update the widget
  - [ ] Changes persist to markdown source
  - [ ] Multiple instances work

- [ ] **Edge Cases**
  - [ ] Invalid syntax doesn't break
  - [ ] Boundary values (min/max)
  - [ ] Special characters in parameters
  - [ ] Multiple widgets per line
  - [ ] Widgets in lists, blockquotes, tables

- [ ] **Settings**
  - [ ] Widgets disabled when `inlineCounters` is off
  - [ ] Settings persist after reload

### Test in Different Contexts

Create a test note with various scenarios:

```markdown
# Inline Element Tests

## Basic usage
`sd6: 3`

## Multiple on one line
Str: `sd6: 4` Dex: `sd8: 5` Con: `sd6: 6`

## In lists
- Strength: `sd6: 3`
- Dexterity: `sd8: 5`

## In tables
| Stat | Value |
|------|-------|
| STR  | `sd6: 3` |
| DEX  | `sd8: 5` |

## Edge cases
Min value: `sd6: 1`
Max value: `sd6: 6`
With color: `sd6,red: 3`
```

## Code Style Guidelines

### TypeScript

- Use TypeScript's type system - avoid `any`
- Prefer interfaces over type aliases for objects
- Use `const` and `let`, not `var`
- Use arrow functions for callbacks
- Add JSDoc comments for public APIs

### Naming Conventions

- **Classes**: PascalCase (e.g., `StatDieWidget`)
- **Functions/Variables**: camelCase (e.g., `parseValue`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `STATDIE_REGEX`)
- **Files**: kebab-case (e.g., `stat-die.ts`) or camelCase for base names

### CSS/SCSS

- Prefix all classes with `srt-` (Solo RPG Toolkit)
- Use BEM-like naming: `srt-component__element--modifier`
- Keep specificity low
- Use CSS variables for colors and sizes when possible

### Commit Messages

Follow conventional commits format:

- `feat: add stat die tracker inline element`
- `fix: correct dice rolling animation`
- `docs: update README with new features`
- `refactor: simplify base widget interface`
- `style: format code with prettier`
- `test: add edge case tests for counters`

## Common Patterns

### Parsing Code Block Text

Always normalize the input by removing backticks first:

```typescript
private parseValue(text: string) {
  const normalized = text.replace(/^`+|`+$/g, "");
  // Then parse normalized text
}
```

### Using Menu Options

For right-click context menus, use the `createMenu` helper:

```typescript
import { createMenu } from "./shared";

const menu = createMenu([
  {
    title: "Option 1",
    onClick: () => {
      // Handle click
      onChange?.();
    },
  },
  "-", // Separator
  {
    title: "Submenu",
    subMenu: [
      { title: "Sub-option", onClick: () => {} }
    ],
  },
]);

element.oncontextmenu = (event) => {
  event.preventDefault();
  menu.showAtMouseEvent(event);
};
```

### Clamping Values

Always validate and clamp values to acceptable ranges:

```typescript
private setValue(newValue: number) {
  this.value = newValue;
  if (this.value < this.minValue) this.value = this.minValue;
  if (this.value > this.maxValue) this.value = this.maxValue;
}
```

### State Management

The plugin uses a simple pattern:
1. Base widget holds the state
2. User interactions modify state
3. `onChange` callback triggers document update
4. Document update persists state to markdown

## Getting Help

- **Issues**: Open an issue on GitHub with:
  - Clear description of the problem
  - Steps to reproduce
  - Expected vs actual behavior
  - Environment (Obsidian version, OS)

- **Questions**: Use GitHub Discussions for:
  - Feature ideas
  - Architecture questions
  - General help

## Resources

- [Obsidian Plugin Developer Docs](https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin)
- [CodeMirror 6 Documentation](https://codemirror.net/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

Thank you for contributing to Solo RPG Toolkit! 🎲
