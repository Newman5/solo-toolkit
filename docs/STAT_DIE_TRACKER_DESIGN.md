# Feature Design: Stat Die Tracker

## Overview

The Stat Die Tracker is a new inline element that combines the functionality of dynamic counters (for adjusting values) with dice rolling (for randomization). It's designed for TTRPG systems where stats have an associated die type (e.g., Cortex Plus, Fate, Savage Worlds).

## Use Case Examples

- **Cortex Plus**: Attributes rated as dice (d4 to d12)
- **Savage Worlds**: Skills rated by dice types (d4, d6, d8, d10, d12)
- **Custom Systems**: Any game where you track both a die size and current value

## Syntax Design

### Final Recommended Syntax

```
`sd[SIZE]: [VALUE]`
```

Where:
- `sd` = "stat die" prefix (distinguishes from regular dice)
- `[SIZE]` = die size (4, 6, 8, 10, 12, 20)
- `[VALUE]` = current value (1 to SIZE)

**Basic Examples:**
```markdown
`sd6: 3`    - d6 stat, current value 3
`sd8: 5`    - d8 stat, current value 5
`sd10: 1`   - d10 stat, current value 1
```

### Extended Syntax with Options

Optional color parameter (consistent with existing dice notation):

```
`sd[SIZE],[COLOR]: [VALUE]`
```

**Examples:**
```markdown
`sd6,red: 3`       - Red colored d6 stat die
`sd8,#fb464c: 5`   - Custom hex color
`sd10,green: 7`    - Green d10
```

### Syntax Decision Rationale

**Why `sd` prefix?**
- Short and mnemonic ("stat die")
- Avoids conflict with regular dice (`d6`)
- Avoids conflict with counters (`5`)
- Follows similar pattern to sized clocks (`smclock`, `lgclock`)

**Why colon separator?**
- Consistent with existing inline elements (`clock: 1/5`, `boxes: 2/4`)
- Visually distinguishes die size from current value
- Reads naturally: "stat die 6, value 3"

**Alternative syntaxes considered:**
- `d6:3` - Too similar to regular dice with results (`d6: 4`)
- `sd(6): 3` - Unnecessary parentheses, more verbose
- `s6: 3` - Less clear, harder to understand at a glance
- `stat:d6:3` - Too verbose, harder to type

## Parsing Rules

### Regular Expression

```regex
/^`sd(4|6|8|10|12|20)([|,]#?[\w\d]+)*: ?[+-]?\d+`$/
```

**Pattern breakdown:**
- `^` and `$` - Match entire code block
- `` ` `` - Backticks (code block markers)
- `sd` - Literal prefix
- `(4|6|8|10|12|20)` - Allowed die sizes
- `([|,]#?[\w\d]+)*` - Optional color parameters (comma or pipe separated)
- `: ?` - Colon separator with optional space
- `[+-]?\d+` - Current value (optional sign, one or more digits)
- `` ` `` - Closing backtick

### Validation Rules

1. **Die Size Validation**
   - Only allow: 4, 6, 8, 10, 12, 20
   - Reject other sizes (no d2, d3, d100, dF, etc.)
   - Rationale: These are the standard RPG die sizes

2. **Value Range Validation**
   - Minimum: 1 (no zero or negative values)
   - Maximum: die size value
   - Auto-clamp on parse: if value > size, set to size
   - Auto-clamp on change: prevent values outside range

3. **Color Validation**
   - Accept predefined colors: red, orange, yellow, green, cyan, blue, purple, pink
   - Accept hex colors: `#rrggbb` format
   - Empty color = use default theme color

4. **Whitespace Handling**
   - Allow optional space after colon: `sd6: 3` and `sd6:3` both valid
   - Trim whitespace during parsing

### Edge Cases

| Input | Behavior | Reason |
|-------|----------|--------|
| `` `sd6: 0` `` | Clamp to 1 | Min value is 1 |
| `` `sd6: 10` `` | Clamp to 6 | Max value is die size |
| `` `sd6: -5` `` | Clamp to 1 | No negative values |
| `` `sd6,red,blue: 3` `` | Use first color (red) | Multiple colors not supported |
| `` `sd6: ` `` | Invalid, no match | Missing value |
| `` `sd3: 2` `` | Invalid, no match | d3 not supported |
| `` `sd100: 50` `` | Invalid, no match | d100 not supported (use regular dice) |

## Rendering Approach

### DOM Structure

```html
<span class="srt-statdie">
  <button class="clickable-icon srt-statdie-die">d6</button>
  <button class="clickable-icon"><svg><!-- minus icon --></svg></button>
  <span class="srt-statdie-value">3</span>
  <button class="clickable-icon"><svg><!-- plus icon --></svg></button>
</span>
```

### Visual Design

```
┌──────────────────────┐
│ [d6] [-] 3 [+]      │
└──────────────────────┘
```

**Components:**
1. **Die Button** - Shows die size, clickable to roll
2. **Minus Button** - Decrease value by 1
3. **Value Display** - Current value, clickable to edit code
4. **Plus Button** - Increase value by 1

### Color Application

When a color is specified:
- Apply color to die button text
- Apply color as border/outline to die button
- Keep consistent with existing dice color system

### Interaction Behaviors

| Action | Effect |
|--------|--------|
| Click die button | Roll 1d[size], set value to result, animate |
| Click minus | Decrease value by 1, clamp at 1 |
| Click plus | Increase value by 1, clamp at die size |
| Shift+click minus | Decrease by 10 (or to minimum) |
| Shift+click plus | Increase by 10 (or to maximum) |
| Right-click minus | Decrease by 10 (consistent with counters) |
| Right-click plus | Increase by 10 (consistent with counters) |
| Click value | Focus on code block for manual editing |
| Right-click anywhere | Show context menu (if implemented) |

### Context Menu (Optional)

If implementing a context menu:

```
Roll
────────────
Set to Max
Set to Min
────────────
Color >
  Default
  Red
  Orange
  Yellow
  Green
  Cyan
  Blue
  Purple
  Pink
────────────
Edit
```

## State Update Strategy

### Live Preview Mode (CodeMirror 6)

**On state change:**

1. Update internal widget state
2. Update DOM to reflect new state
3. Call `onChange()` callback
4. In `onChange()`, dispatch document change:
   ```typescript
   view.dispatch({
     changes: [{
       from: this.node.from,
       to: this.node.to,
       insert: this.base.getText("`"),
     }],
   });
   ```

**Example flow:**
```
User clicks plus button
→ value changes from 3 to 4
→ valueEl.innerText updates to "4"
→ onChange() called
→ Document text updated: `sd6: 3` → `sd6: 4`
→ Markdown file saved by Obsidian
```

### Reading Mode

**On state change:**

1. Update internal widget state
2. Update DOM to reflect new state
3. Call `onChange()` callback
4. In `onChange()`, use `replaceInFile()`:
   ```typescript
   replaceInFile({
     vault: this.app.vault,
     file: this.file,
     regex: STATDIE_REGEX_G,
     lineStart: this.lineStart,
     lineEnd: this.lineEnd,
     newValue: this.base.getText("`"),
     replaceIndex: this.index,
   });
   ```

**Example flow:**
```
User clicks die button
→ roll() generates new value (e.g., 1-6)
→ value changes from 3 to 5
→ valueEl.innerText updates to "5"
→ onChange() called
→ File updated via replaceInFile()
→ Markdown source changes: `sd6: 3` → `sd6: 5`
```

### Multiple Instances on Same Line

The `replaceIndex` parameter handles multiple widgets:

```markdown
Str: `sd6: 4`  Dex: `sd8: 5`  Con: `sd6: 3`
     ^index 0       ^index 1       ^index 2
```

Each widget knows its position and only updates its own occurrence.

## Edge Cases and Solutions

### 1. Multiple Stat Dice on One Line

**Scenario:**
```markdown
Str: `sd6: 4` Dex: `sd8: 5` Con: `sd6: 3`
```

**Solution:**
- Live Preview: Each widget has unique syntax node position
- Reading Mode: Use `replaceIndex` to target specific occurrence
- Test: Clicking any widget only updates that widget

### 2. Invalid Values

**Scenario:** User manually edits to `sd6: 99`

**Solution:**
- Parse with clamping: `value = Math.min(Math.max(parsedValue, 1), dieSize)`
- On render, value automatically corrected to valid range
- Next interaction will persist corrected value

### 3. Value of Zero

**Scenario:** User manually edits to `sd6: 0`

**Solution:**
- Clamp to minimum value of 1
- Rationale: Stat dice represent "having the stat", zero would be invalid

### 4. Negative Values

**Scenario:** User manually edits to `sd6: -3`

**Solution:**
- Clamp to minimum value of 1
- Negative values don't make sense for stat die size

### 5. Whitespace Variations

**Scenario:** Various spacing around colon

```markdown
`sd6:3`    - No space
`sd6: 3`   - One space (standard)
`sd6:  3`  - Two spaces
```

**Solution:**
- Regex allows optional space: `: ?`
- Parser trims value: `parseInt(value?.trim())`
- Output always uses single space: `sd6: 3`

### 6. Color Conflicts

**Scenario:** `sd6,red,blue: 3`

**Solution:**
- Take first color parameter only
- Ignore subsequent colors
- Could log warning in debug mode

### 7. Unsupported Die Sizes

**Scenario:** User types `sd100: 50`

**Solution:**
- Regex won't match, renders as regular code
- User can use regular dice for d100: `d100: 50`
- Stat dice limited to common RPG sizes

### 8. Widget in Complex Markdown

**Scenario:** Widget inside table, list, or blockquote

```markdown
| Stat | Die |
|------|-----|
| STR  | `sd6: 4` |

- Strength: `sd6: 4`

> Health: `sd8: 6`
```

**Solution:**
- Works automatically (same as other inline elements)
- Test all markdown contexts during development

## Style Considerations

### CSS Classes

```scss
.srt-statdie {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  
  .srt-statdie-die {
    font-size: 0.85em;
    font-weight: 600;
    min-width: 30px;
    padding: 2px 6px;
    border: 1px solid currentColor;
    border-radius: 3px;
    
    &:hover {
      background-color: var(--interactive-hover);
    }
  }
  
  .srt-statdie-value {
    min-width: 24px;
    text-align: center;
    font-weight: 500;
    cursor: pointer;
    
    &:hover {
      color: var(--text-accent);
    }
  }
}
```

### Consistent with Existing Elements

- Use Obsidian's `clickable-icon` class for buttons
- Use `setIcon()` for plus/minus icons (matches counters)
- Follow spacing and sizing from existing inline elements
- Respect theme colors and dark/light mode

## Compatibility Considerations

### Doesn't Break Existing Elements

**Regular Dice:**
- `d6` still matches dice pattern, not stat die
- Different prefix (`d` vs `sd`)

**Counters:**
- `5` still matches counter pattern
- Stat die requires `sd` prefix and colon

**Progress Trackers:**
- `1/5` still matches counter with limit
- Stat die doesn't use slash separator

**Clocks/Boxes:**
- `clock: 1/5` still matches clock pattern
- Different prefix and syntax structure

### Testing Strategy

Create test note with all inline elements:

```markdown
# All Inline Elements Test

Regular dice: `d6` `2d8+3`
Counters: `5` `10`
Counter with limit: `1/5` `3/10`
Boxes: `boxes: 2/5`
Clock: `clock: 3/6`
Tab stop: ` `

NEW - Stat dice: `sd6: 3` `sd8: 5` `sd10,red: 7`
```

Verify:
- All elements render correctly
- No interference between types
- Settings toggle affects all

## Implementation Checklist

### Phase 1: Base Implementation
- [ ] Create `src/inline/base/statdie.ts`
  - [ ] Define `STATDIE_REGEX` pattern
  - [ ] Implement `StatDieWidgetBase` class
  - [ ] Implement `parseValue()` method with validation
  - [ ] Implement `generateDOM()` method
  - [ ] Implement `getText()` method
  - [ ] Implement `roll()` method
  - [ ] Implement `addValue()` method with clamping
- [ ] Export from `src/inline/base/index.ts`

### Phase 2: Live Preview Integration
- [ ] Create `src/inline/live/statdie.ts`
  - [ ] Implement `StatDieWidget` class extending `WidgetType`
  - [ ] Implement `toDOM()` method
  - [ ] Implement `updateDoc()` method
- [ ] Register in `src/inline/live/index.ts`
  - [ ] Import widget and regex
  - [ ] Add check in `buildDecorations()`
  - [ ] Add to buildMeta tracking

### Phase 3: Reading Mode Integration
- [ ] Create `src/inline/read/statdie.ts`
  - [ ] Implement `StatDieWidget` class
  - [ ] Implement `toDOM()` method
  - [ ] Implement `updateDoc()` using `replaceInFile`
- [ ] Register in `src/inline/read/index.ts`
  - [ ] Import widget and regex
  - [ ] Add to indexMap
  - [ ] Add check in post-processor loop

### Phase 4: Styling
- [ ] Add styles to `src/styles.scss`
  - [ ] `.srt-statdie` container
  - [ ] `.srt-statdie-die` button
  - [ ] `.srt-statdie-value` display
  - [ ] Color variants
  - [ ] Hover states

### Phase 5: Testing
- [ ] Test Live Preview mode
  - [ ] Basic rendering
  - [ ] Die button rolls correctly
  - [ ] Plus/minus buttons work
  - [ ] Values clamp correctly
  - [ ] Multiple on same line
  - [ ] Color variations
- [ ] Test Reading mode
  - [ ] Basic rendering
  - [ ] All interactions work
  - [ ] File updates persist
- [ ] Test edge cases
  - [ ] Invalid values clamp correctly
  - [ ] Whitespace handling
  - [ ] In tables, lists, blockquotes
  - [ ] Settings toggle on/off
- [ ] Test compatibility
  - [ ] Doesn't break existing inline elements
  - [ ] Works alongside other elements

### Phase 6: Documentation
- [ ] Update README.md
  - [ ] Add Stat Die Tracker section
  - [ ] Include syntax examples
  - [ ] Document interaction behavior
- [ ] Update CONTRIBUTING.md
  - [ ] Add stat die to examples (if needed)
- [ ] Add inline comments to code

### Phase 7: Polish (Optional)
- [ ] Implement context menu
- [ ] Add roll animation (like regular dice)
- [ ] Add tooltips
- [ ] Add keyboard shortcuts
- [ ] Consider size variants (small/large)

## Estimated Implementation Size

**Lines of Code:**
- Base widget: ~150 lines
- Live widget: ~30 lines  
- Read widget: ~40 lines
- Registration code: ~20 lines
- Styles: ~40 lines
- **Total: ~280 lines** (small, focused change)

**Files Modified/Created:**
- Created: 3 new files
- Modified: 4 existing files (live/read index, base index, styles)

**Risk Level:** Low
- Self-contained feature
- No changes to existing widgets
- Follows established patterns
- Minimal surface area for bugs

## Alternative Approaches Considered

### Alternative 1: Extend Existing Counter

**Approach:** Add die button to existing counter widget

**Pros:**
- Reuses existing code
- Less new code to write

**Cons:**
- Counters have different semantics (arbitrary numbers)
- Would need to make die size configurable per counter
- Syntax becomes complex: `5,d6` (ambiguous)
- Breaks single responsibility principle

**Verdict:** Not recommended - separate concerns

### Alternative 2: Extend Existing Dice

**Approach:** Add +/- buttons to existing dice widget

**Pros:**
- Reuses dice rendering
- Familiar die visual

**Cons:**
- Regular dice are for rolling, not tracking state
- Dice results are transient, stat values are persistent
- Would complicate dice regex (already complex)
- Different use case semantics

**Verdict:** Not recommended - different use cases

### Alternative 3: Composite Syntax

**Approach:** Combine existing elements: `d6` `5`

**Pros:**
- No new code needed
- Uses existing elements

**Cons:**
- Requires two separate code blocks
- No visual connection between die and value
- Confusing UX (which die goes with which value?)
- Can't roll and set value atomically

**Verdict:** Not recommended - poor UX

### Recommended: New Widget

**Approach:** Create dedicated stat die widget

**Pros:**
- Clear, unambiguous syntax
- Appropriate semantics for use case
- Clean implementation following patterns
- No compromise on UX

**Cons:**
- Adds one new file type

**Verdict:** ✅ Best solution

## Future Enhancements

Possible future additions (out of scope for initial implementation):

1. **Step Die** - Auto-upgrade/downgrade die size
   - `sd6: 3` with "step up" → `sd8: 3`
   - Useful for Cortex Plus, Savage Worlds advancement

2. **Die Pool** - Multiple dice of same size
   - `3sd6: 4,5,6` (three d6s with individual values)
   - For dice pool systems

3. **Exploding/Reroll** - Special roll behaviors
   - `sd6,explode: 3` (reroll on max)
   - System-specific mechanics

4. **Linked Stats** - Die affects modifier
   - `sd8+2: 5` (d8 with +2 modifier based on value)
   - Complex stat systems

5. **Visual Die Face** - Show actual die graphic
   - SVG die face instead of text
   - More visual, but more complex

These can be added incrementally without changing the core syntax.

## Summary

The Stat Die Tracker is a focused, well-scoped feature that:

✅ Solves a real use case (stat tracking with die sizes)  
✅ Uses clear, intuitive syntax  
✅ Follows established patterns  
✅ Doesn't break existing features  
✅ Has manageable implementation size  
✅ Provides excellent UX  

**Recommended for implementation.**
