# Implementation Plan: Stat Die Tracker

This document outlines a step-by-step implementation plan for adding the Stat Die Tracker inline element to Solo RPG Toolkit.

## Implementation Strategy

The implementation is divided into small, isolated PRs/commits that can be reviewed and tested independently. Each step builds on the previous one and maintains a working state.

## Prerequisites

Before starting:
- Read [CONTRIBUTING.md](/CONTRIBUTING.md) for development setup
- Review [STAT_DIE_TRACKER_DESIGN.md](/docs/STAT_DIE_TRACKER_DESIGN.md) for feature specification
- Ensure development environment is set up (`npm install` completed)
- Have a test Obsidian vault ready with the plugin in dev mode

## Phase 1: Foundation (Base Widget)

### PR #1: Add Base Widget Class

**Goal:** Implement the core stat die logic without any UI integration.

**Files to create:**
- `src/inline/base/statdie.ts`

**Files to modify:**
- `src/inline/base/index.ts` (add export)

**Implementation steps:**

1. Create `src/inline/base/statdie.ts`:
   ```typescript
   // Define regex patterns
   export const STATDIE_REGEX = /^`sd(4|6|8|10|12|20)([|,]#?[\w\d]+)*: ?[+-]?\d+`$/;
   export const STATDIE_REGEX_G = /`sd(4|6|8|10|12|20)([|,]#?[\w\d]+)*: ?[+-]?\d+`/g;
   
   // Define StatDieWidgetBase class
   // - Properties: dieSize, value, color
   // - parseValue() method
   // - roll() method (using nrandom from utils)
   // - addValue() method with clamping
   // - getText() method
   // - generateDOM() method
   ```

2. Add export to `src/inline/base/index.ts`:
   ```typescript
   export * from "./statdie";
   ```

3. Build and verify no TypeScript errors:
   ```bash
   npm run build
   ```

**Testing at this stage:**
- TypeScript compiles without errors
- No runtime testing yet (not integrated into UI)

**Success criteria:**
- ✅ Build completes successfully
- ✅ Base widget class follows existing patterns (CountWidgetBase, DiceWidgetBase)
- ✅ Code passes linting

**Commit message:** `feat: add StatDieWidgetBase class for stat die tracking`

---

## Phase 2: Live Preview Integration

### PR #2: Add Live Preview Support

**Goal:** Make stat dice render and work in Live Preview mode (editing view).

**Files to create:**
- `src/inline/live/statdie.ts`

**Files to modify:**
- `src/inline/live/index.ts` (register widget)

**Implementation steps:**

1. Create `src/inline/live/statdie.ts`:
   ```typescript
   import { SyntaxNode } from "@lezer/common";
   import { EditorView, WidgetType } from "@codemirror/view";
   import { StatDieWidgetBase, STATDIE_REGEX } from "../base";
   import { focusOnNode } from "./shared";
   
   export { STATDIE_REGEX };
   
   export class StatDieWidget extends WidgetType {
     // Wrap StatDieWidgetBase
     // Implement toDOM()
     // Implement updateDoc()
   }
   ```

2. Register in `src/inline/live/index.ts`:
   - Import `StatDieWidget` and `STATDIE_REGEX`
   - Add regex check in `buildDecorations()` method
   - Add to `buildMeta` tracking

3. Build and test:
   ```bash
   npm run build
   npm run deploy
   ```

4. Manual testing in Obsidian:
   - Create test note
   - Enable inline elements in settings
   - Type: `` `sd6: 3` ``
   - Switch to Live Preview mode
   - Verify widget renders
   - Test interactions: click die (roll), click +/- (adjust)
   - Verify markdown updates when values change

**Testing checklist:**
- [ ] Widget renders in Live Preview
- [ ] Die button rolls correctly
- [ ] Plus/minus buttons adjust value
- [ ] Values clamp at boundaries (1 to die size)
- [ ] Multiple stat dice on same line work independently
- [ ] Colored variant works: `sd6,red: 3`
- [ ] Switching to source mode shows updated markdown
- [ ] Clicking value focuses on code for editing

**Success criteria:**
- ✅ Live Preview mode fully functional
- ✅ All interactions update markdown source
- ✅ No TypeScript errors
- ✅ No console errors in Obsidian

**Commit message:** `feat: add stat die support in Live Preview mode`

---

## Phase 3: Reading Mode Integration

### PR #3: Add Reading Mode Support

**Goal:** Make stat dice render and work in Reading mode (preview view).

**Files to create:**
- `src/inline/read/statdie.ts`

**Files to modify:**
- `src/inline/read/index.ts` (register widget)

**Implementation steps:**

1. Create `src/inline/read/statdie.ts`:
   ```typescript
   import { TFile, App } from "obsidian";
   import { replaceInFile } from "src/utils/plugin";
   import { StatDieWidgetBase, STATDIE_REGEX, STATDIE_REGEX_G } from "../base";
   
   export { STATDIE_REGEX };
   
   export class StatDieWidget {
     // Wrap StatDieWidgetBase
     // Implement toDOM()
     // Implement updateDoc() using replaceInFile
   }
   ```

2. Register in `src/inline/read/index.ts`:
   - Import `StatDieWidget` and `STATDIE_REGEX`
   - Add `statdie: 0` to `indexMap`
   - Add regex check in post-processor loop
   - Create and render widget

3. Build and test:
   ```bash
   npm run build
   npm run deploy
   ```

4. Manual testing in Obsidian:
   - Open test note with stat dice
   - Switch to Reading mode
   - Verify widgets render
   - Test all interactions
   - Switch back to Live Preview, verify changes persisted

**Testing checklist:**
- [ ] Widget renders in Reading mode
- [ ] All interactions work (roll, +/-)
- [ ] File updates persist
- [ ] Multiple instances work correctly
- [ ] Works in complex markdown (tables, lists, blockquotes)
- [ ] Switch between Live Preview and Reading preserves state

**Success criteria:**
- ✅ Reading mode fully functional
- ✅ File persistence works correctly
- ✅ No race conditions with multiple instances
- ✅ Works seamlessly with Live Preview

**Commit message:** `feat: add stat die support in Reading mode`

---

## Phase 4: Styling and Polish

### PR #4: Add Styles and Visual Polish

**Goal:** Add CSS styling to make stat dice look good and consistent with existing inline elements.

**Files to modify:**
- `src/styles.scss`

**Implementation steps:**

1. Add styles to `src/styles.scss`:
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
       transition: background-color 0.1s ease;
       
       &:hover {
         background-color: var(--interactive-hover);
       }
     }
     
     .srt-statdie-value {
       min-width: 24px;
       text-align: center;
       font-weight: 500;
       cursor: pointer;
       padding: 2px 4px;
       border-radius: 2px;
       
       &:hover {
         color: var(--text-accent);
         background-color: var(--background-modifier-hover);
       }
     }
   }
   ```

2. Test appearance in both light and dark themes

3. Test with various colors: `sd6,red: 3`, `sd8,blue: 5`, etc.

**Testing checklist:**
- [ ] Styling consistent with other inline elements
- [ ] Looks good in light theme
- [ ] Looks good in dark theme
- [ ] Hover states work properly
- [ ] Colors apply correctly
- [ ] Responsive to different font sizes
- [ ] Works with custom themes

**Success criteria:**
- ✅ Professional appearance
- ✅ Consistent with plugin's design language
- ✅ Accessible (good contrast, hover indicators)
- ✅ Theme-compatible

**Commit message:** `style: add CSS styling for stat die tracker`

---

## Phase 5: Documentation

### PR #5: Update Documentation

**Goal:** Document the new feature for users and contributors.

**Files to modify:**
- `README.md`
- `CONTRIBUTING.md` (optional - examples already included)

**Implementation steps:**

1. Add section to `README.md` under "Inline elements":
   ```markdown
   #### Stat die trackers
   
   Track stats with associated die types (useful for Cortex Plus, Savage Worlds, etc.)
   
   Basic syntax:
   - `sd6: 3` - d6 stat with current value 3
   - `sd8: 5` - d8 stat with current value 5
   - `sd10: 7` - d10 stat with current value 7
   
   Supported die sizes: d4, d6, d8, d10, d12, d20
   
   With color: `sd6,red: 3` or `sd6,#fb464c: 3`
   
   Interaction:
   - Click the die button to roll
   - Click +/- to adjust value (shift+click or right-click for ±10)
   - Click the value to edit manually
   ```

2. Add examples section:
   ```markdown
   **Example character sheet:**
   ```
   Strength: `sd6: 4`
   Dexterity: `sd8: 6`
   Intelligence: `sd6: 3`
   Willpower: `sd10: 8`
   ```
   ```

3. Review CONTRIBUTING.md to ensure stat die examples are current (already included in the guide)

**Success criteria:**
- ✅ Clear explanation of syntax
- ✅ Examples provided
- ✅ Interaction behavior documented
- ✅ Use cases explained

**Commit message:** `docs: add stat die tracker to README`

---

## Phase 6: Testing and Edge Cases

### PR #6: Comprehensive Testing

**Goal:** Test all edge cases and ensure robustness.

**Testing approach:**

Create a comprehensive test note:

```markdown
# Stat Die Tracker Tests

## Basic functionality
`sd4: 2` `sd6: 3` `sd8: 5` `sd10: 7` `sd12: 9` `sd20: 15`

## Multiple on same line
STR: `sd6: 4`  DEX: `sd8: 5`  CON: `sd6: 6`

## With colors
Red: `sd6,red: 3`
Green: `sd8,green: 5`
Blue: `sd10,blue: 7`
Custom: `sd6,#ff69b4: 4`

## Edge cases - boundary values
Min: `sd6: 1`
Max: `sd6: 6`
Out of range high: `sd6: 10` (should clamp to 6)
Out of range low: `sd6: 0` (should clamp to 1)

## In different contexts

### In lists
- Strength: `sd6: 4`
- Dexterity: `sd8: 5`
  - Acrobatics: `sd6: 3`
  - Stealth: `sd8: 6`

### In tables
| Stat | Die |
|------|-----|
| STR  | `sd6: 4` |
| DEX  | `sd8: 5` |
| INT  | `sd6: 3` |

### In blockquotes
> Character stats:
> - Combat: `sd8: 6`
> - Magic: `sd10: 8`

### In callouts (Obsidian-specific)
> [!info] Character Stats
> Strength: `sd6: 4`
> Dexterity: `sd8: 5`

## Compatibility check
All inline elements together:

Counter: `5`
Counter with limit: `3/10`
Boxes: `boxes: 2/5`
Clock: `clock: 3/6`
Dice: `d6` `2d8+3`
Stat die: `sd6: 3`
```

**Manual test procedure:**

1. **Live Preview Mode:**
   - [ ] All stat dice render correctly
   - [ ] Rolling works (values change within range)
   - [ ] +/- buttons work
   - [ ] Shift+click for ±10 works
   - [ ] Right-click for ±10 works
   - [ ] Multiple instances don't interfere
   - [ ] Colors apply correctly
   - [ ] Edge cases handle gracefully

2. **Reading Mode:**
   - [ ] Same tests as Live Preview
   - [ ] File updates persist correctly

3. **Switching Modes:**
   - [ ] State preserved when switching
   - [ ] No visual glitches

4. **Edge Cases:**
   - [ ] Invalid values clamp correctly
   - [ ] Out-of-range values auto-correct
   - [ ] Unsupported die sizes don't match (`sd100: 50` stays as code)
   - [ ] Missing values don't crash (`sd6:` doesn't match)

5. **Compatibility:**
   - [ ] Other inline elements still work
   - [ ] No interference between types
   - [ ] Settings toggle affects stat dice
   - [ ] Disable inline elements - stat dice become plain code

6. **Performance:**
   - [ ] No lag with many stat dice
   - [ ] Fast updates on interaction
   - [ ] No memory leaks

**Success criteria:**
- ✅ All test cases pass
- ✅ No console errors
- ✅ No TypeScript errors
- ✅ Smooth user experience
- ✅ Compatible with all markdown contexts

**Commit message:** `test: verify stat die tracker edge cases and compatibility`

---

## Phase 7 (Optional): Advanced Features

### PR #7: Add Context Menu (Optional)

**Goal:** Add right-click menu for additional options.

**Implementation:**
- Add menu to base widget using `createMenu` helper
- Menu items: Roll, Set to Max, Set to Min, Color submenu, Edit
- Follow pattern from `DiceWidgetBase` and `ClockWidgetBase`

**Commit message:** `feat: add context menu to stat die tracker`

---

### PR #8: Add Roll Animation (Optional)

**Goal:** Add visual feedback when rolling.

**Implementation:**
- Reuse roll animation from `DiceWidgetBase`
- Show intermediate values before settling on result
- Use `rollIntervals` timing array

**Commit message:** `feat: add roll animation to stat die tracker`

---

## Timeline Estimate

**Minimal viable feature (PR #1-5):**
- PR #1 (Base): 1-2 hours
- PR #2 (Live Preview): 1-2 hours
- PR #3 (Reading): 1 hour
- PR #4 (Styling): 1 hour
- PR #5 (Docs): 30 minutes
- **Total: 5-7 hours** for basic implementation

**With testing and polish (PR #6-8):**
- PR #6 (Testing): 2-3 hours
- PR #7 (Context menu): 1 hour
- PR #8 (Animation): 1 hour
- **Total: 9-12 hours** for complete implementation

## Risk Mitigation

### Potential Issues and Solutions

1. **Regex conflicts with existing patterns**
   - Mitigation: Test regex thoroughly, use very specific pattern
   - Fallback: Adjust `sd` prefix if needed (e.g., `std`, `sdie`)

2. **Performance with many widgets**
   - Mitigation: Reuse existing widget architecture (proven performant)
   - Fallback: Add throttling if needed

3. **File persistence issues**
   - Mitigation: Use proven `replaceInFile` utility
   - Fallback: Use alternative file update method

4. **Theme compatibility**
   - Mitigation: Use CSS variables, test multiple themes
   - Fallback: Add theme-specific overrides

5. **Mobile compatibility**
   - Mitigation: Test on mobile (plugin claims mobile support)
   - Fallback: Adjust touch event handling if needed

## Success Metrics

### Definition of Done

Feature is considered complete when:

✅ All 5 core PRs merged (PRs #1-5)  
✅ Works in both Live Preview and Reading modes  
✅ Documentation updated  
✅ No TypeScript errors  
✅ No console errors in Obsidian  
✅ Manually tested in multiple contexts  
✅ Compatible with existing inline elements  
✅ Settings toggle controls stat dice  
✅ Looks professional in light and dark themes  

### Optional Enhancements Complete

✅ Context menu implemented (PR #7)  
✅ Roll animation added (PR #8)  
✅ Comprehensive test suite run (PR #6)  

## Rollback Plan

If issues arise:

1. **Before merging PR #2:** Simply don't merge, no impact
2. **After merging PRs #1-2:** Remove live integration, keep base class
3. **After merging all PRs:** 
   - Feature flag approach: Add setting to disable stat dice specifically
   - Full rollback: Revert commits, remove files
   - The feature is isolated enough that removal is clean

## Post-Implementation

After successful implementation:

1. **Monitor for issues:**
   - Watch GitHub issues for bug reports
   - Test in various Obsidian versions
   - Test with popular themes

2. **Gather feedback:**
   - Community response
   - Feature requests
   - UX improvements

3. **Future enhancements:**
   - Consider Phase 7 features
   - Implement based on user feedback
   - Prioritize most requested features

## Conclusion

This implementation plan provides a clear, step-by-step path to adding the Stat Die Tracker feature. Each phase is self-contained, testable, and builds on the previous one. The feature can be implemented incrementally with low risk.

**Ready to implement? Start with PR #1!**
