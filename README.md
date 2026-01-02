# Solo RPG Toolkit

A plugin for Obsidian (https://obsidian.md) that adds helpful features for playing TTRPGs. Its features are mainly geared towards solo gameplay using a GM emulator.

## Features

### Dice roller

Quickly roll a bunch of dice by clicking on a die icon. Hold shift, or right click/long tap to roll dice with alternative color.

### Deck of cards

A standard deck of 52 cards plus two optional joker cards, and a deck of oracle major arcana cards.

### Custom decks

You can add any number of your own decks by creating folders of images in a special folder of your vault (default folder name is "Decks", you can change it in plugin settings). This feature can be useful with products like "The GameMaster's Apprentice" VTT decks, but can also be used with any kind of images: maps, item icons, or portraits.

#### Other custom deck options

- Cards can be rotated in your custom deck. Create a note inside your deck folder and add a word `flip` to randomize upright and upside down cards. Other options are `flip2` for upright and tapped, `flip3` for upright and tapped in both directions, `flip4` for random rotation.
- Cards can be added as URLs instead of image files. Create a note inside your deck folder and paste links to images you want to use.
- Right click on a card will shuffle it back in.

### Random generators

A set of random word generators. Great for coming up with ideas on how to progress the story.

### Custom random tables

In addition to default random word generators, you may add any number of your own random tables by creating notes inside a special folder in your vault (default folder name is "Tables", you can change it in plugin settings). To create more table categories, organize your notes into subfolders.

### Custom table templates

By default a custom random table roll will return a random line from the note. You can further customize this behavior with templates.

You can add one or multiple templates as properties of the note (simply type `---` at the start of the note). The value of a property can contain regular text as well as keywords inside curly brackets (`{keyword}`), those keywords will be replaced with a random word from a given section of the note.

Example of a note with templates:

```markdown
---
loves: "{name} loves {food} ({how much})"
hates: "{name} hates {food} ({how much})"
---

## Names
James
Louie
Quinn
Drew
Melissa
Martha
Elsa
Emily

## Food
pizza
sushi
burgers
tacos
tea
coffee

## How much
very much
a little
```

#### Other keyword options:

- Refer to other notes by specifying the note name and/or note section: `{note_name/section_title}`, or `{folder_name/note_name/section_title}`, or simply `{note_name}` if there are no sections.
- Refer to any note in the vault by its full path: `{/folder_name/subfolder_name/note_name}`.
- Refer to all sections in all notes from a specific folder: `{folder_name}`.
- Refer to multiple sections by listing them separately: `{name|food}`.
- Get a random note title inside a subfolder: `{folder_name/*}`. In order to display note's content instead of its name: `{folder_name/!}`.
- Repeat the same word multiple times: `{noun} and {<noun}`.
- Utilize built-in dictionaries by using any of the following keywords: `{noun}`, `{verb}`, `{adjective}`, `{adverb}`, `{job}`, `{aspect}`, `{town name}`, `{a}`.
- Add bell curve probability by adding Xd at the end of note name or section (e.g. `Clues 2d.md` or `# Food 3d`).
- Add weight to specific template by adding a value to its name: `loves ^10: loves {food}`
- Add weight to specific line of a note by adding a value at the end: `pizza ^10`
- Template key can be capitalized (e.g. `Loves: {food}`) if you'd like to ensure sentence capitalization.
- You can hide a subfolder from dropdown menu by adding a dot at the end of folder's name: `/Items.`, they can still be referenced with a keyword: `{item}`.

#### Cut-up method mode

Custom table notes can also be used in a "cut-up method" mode. Copy-paste any book (or any source material) content into a note, and add a note property `mode: cutup` (type `---` at the start of the note).

In this mode, a random snippet from the note will be returned, instead of a random line.

### Inline elements

This plugin also adds a few inline elements that can be helpful when playing TTRPGs.

Note that these elements are disabled by default and can be enabled in plugin settings.

#### Dynamic counters

Code blocks with a single number in it (e.g. `` `5` ``) will be rendered with - and + buttons for quick adjustment.

- Click + or - to adjust by 1
- Shift+click or right-click +/- to adjust by 10
- Click the value to edit the code directly

#### Progress trackers with limits

Code blocks with two numbers separated by a slash (e.g. `` `1/5` ``) will be rendered as a counter showing current value and maximum with +/- buttons for adjustment.

- Click + or - to adjust the current value
- Shift+click or right-click +/- to adjust the maximum value
- Right-click the value for more options (reset, drain, convert to clock/boxes)

#### Progress trackers (boxes and clocks)

Code blocks with a tracker type and value (e.g. `` `boxes: 1/5` ``) will be rendered as a set of clickable boxes or as a clock. Useful for PbtA-style clocks and general progress tracking.

Basic syntax:
- Boxes — `` `boxes: 1/5` `` or `` `b:1/5` ``
- Circles — `` `circles: 1/5` ``
- Clock — `` `clock: 1/5` `` or `` `c:1/5` ``

Size variants:
- Small — `` `smboxes: 1/5` `` or `` `s` `` prefix
- Large — `` `lgboxes: 1/5` `` or `` `l` `` prefix
- Custom size — `` `boxes,30: 1/5` `` (size in pixels)

Additional options:
- Color — `` `boxes,red: 1/5` `` or `` `boxes,#fb464c: 1/5` ``
- Available colors: red, orange, yellow, green, cyan, blue, purple, pink
- Multiple parameters — `` `boxes,40,red: 1/5` `` (custom size + color)
- Alternative separator — `` `boxes|red: 1/5` `` (pipe instead of comma)

Clocks support up to 16 segments, boxes support up to 200 segments.

Interaction:
- Click a box/segment to set progress to that position
- Right-click for menu options (fill, reset, change color, size, shape)

#### Dice

Code block with dice notation (e.g. `` `d6` `` or `` `2d8` ``) will be rendered as a dice button that can be rolled by clicking on it.

Supported dice: d4, d6, d8, d10, d12, d20, d100, dF (Fudge/Fate dice: -1, 0, +1)

Basic syntax:
- Single die — `` `d6` ``
- Multiple dice — `` `2d8` ``
- Modifier — `` `d20+5` `` or `` `2d6-2` ``

Size variants:
- Small — `` `smd6` `` or `` `sd6` ``
- Large — `` `lgd6` `` or `` `ld6` ``
- Custom size — `` `d6,30` `` (size in pixels)

Additional options:
- Color — `` `d6,red` `` or `` `d6,#fb464c` ``
- Available colors: red, orange, yellow, green, cyan, blue, purple, pink
- Show dice notation — `` `d6,show` `` displays "d6 = " before result
- Lock/disable — `` `!d6` `` prevents rolling (useful for fixed values)
- Multiple parameters — `` `d20,red,40,show` `` (multiple options)
- Alternative separator — `` `d6|red` `` (pipe instead of comma)

The dice will show the last rolled value. Click to roll with animation. Right-click for menu options (roll, color, size, lock/unlock).

#### Tab stops

Code blocks with only spaces (e.g. `` ` ` ``) will be rendered as a blank space with a set width. Useful when you need a table-like formatting without an actual table.

The width adjusts based on the number of spaces. You can also resize by dragging the edges.

Example of formatted stats with dynamic counters:

```markdown
Wounds ` ` `1`
Stress `  ` `2`
```

## Installation

### From GitHub Release

Download solo-rpg-toolkit.zip from the latest release, and extract the plugin folder into your vault's plugins folder: `<path-to-vault>/.obsidian/plugins/`.

Note that `.obsidian` folder in your vault may be hidden on Linux and MacOS.

### From source

You'll need at least node-18 to be installed on your machine. Run the following commands while inside the source folder:

```
npm install
npm run build
npm run deploy
```

You will find a newly generated plugin folder `solo-rpg-toolkit` inside folder `dist`. Move the `solo-rpg-toolkit` plugin folder into your vault's plugins folder: `<path-to-vault>/.obsidian/plugins/`.

Note that `.obsidian` folder in your vault may be hidden on Linux and MacOS.

### Updating

Simply replace `<path-to-vault>/.obsidian/plugins/solo-rpg-toolkit` folder with a new one.

Note that plugin settings are stored in the file `<path-to-vault>/.obsidian/plugins/solo-rpg-toolkit/data.json`. You may want to keep it after updating to a newer version.

## Feature and content requests

If you have an idea for a new feature or new content, feel free to create an "issue" on github or reach me at kurowski.dev@gmail.com!
