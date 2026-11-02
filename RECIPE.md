# Pixel Spaces

Herdr's sidebar in pixel art. Each space wears a small 8×8 sprite in its own color where Herdr
shows a tile with the first letter, the Go to picker shows the same sprites, Claude Code and Codex
get 9×9 pixel icons, and the header keeps only a pixel bell that counts the agents waiting for
you. Everything else is Herdr's.

![Pixel Spaces](preview.png)

## What you see

- **Header** (block `brand`, 46px): on macOS 76px left for the traffic lights, then, only when
  agents wait, a 24px-high pill: a 14px pixel bell and the count in semibold 13.5px. Pill color is
  the theme's `done` color when every waiting agent is done, its `blocked` color when any is
  blocked. With nothing waiting the spot is empty. Herdr's logo and the word "Herdr" are gone.
  Right of it, Herdr's search and sidebar buttons.
- **Spaces list**: exactly as in Herdr's `main`: a "Spaces" header with `+`, a group header per
  machine, one row per space (30px, 28px for worktree rows), the name, then repo, branch and
  ahead/behind where unusual. In place of the 18px rounded tile with a white capital letter: an
  18px box holding a 16×16 sprite, 8×8 sprite pixels at 2px each, no tile behind it.
- Sprite colors: `x` pixels in the space's avatar color (a stored `#rrggbb`, or Herdr's `MARK`
  table: `ansi6` #22a99a, `ansi3` #e3814f, `ansi4` #5b8def, `ansi5` #8b7cf6, `ansi2` #3fae6a,
  `ansi1` #d9649a), `o` pixels #ffffff, `k` pixels #17181c, `.` clear.
- 15 sprites: slime, ghost, cat, robot, mushroom, octopus, skull, heart, potion, frog, rocket,
  terminal (a `>_` prompt), coffee, bomb, star. All original drawings.
- Worktree rows keep Herdr's branch icon and no sprite; the narrow rail keeps Herdr's number and
  status glyph and hides the bell.
- **Go to picker** (`cmd-p` and the header's search button): each space's heading row shows its
  sprite, same color, in place of the 18px letter tile.
- **Agent icons**: Claude Code is a 9×9 pixel spark (8 rays; the horizontal and vertical ones
  run to the edge, the diagonals are two dots each, a 3×3 core), all #D97757. Codex is a 9×9
  pixel `>_`: a 2-pixel-thick `>` over rows 1–5 and a 5-pixel `_` on row 7, all #7A9DFF. No tile
  behind either.

## How it behaves

- Waiting means an agent's status is `blocked` or `done` (finished while you weren't looking;
  Herdr turns it `idle` once you look). The count is over every agent the window knows.
- Clicking the bell runs `pane.focus` on the most urgent waiting agent: blocked before done, then
  the highest `seq` (most recent change). The press doesn't reach the header behind it.
- A space without a sprite gets the one used least among the shown spaces; ties are broken by a
  hash of the space's name. Once given, a sprite stays.
- Option-click (Alt) on a sprite steps that space to the next sprite in the list, wrapping
  around, and the click does not select the space. Other clicks, right-click and drag behave as
  in Herdr's list.
- Sprites of spaces that are gone are dropped, as Herdr drops their colors.
- When a mod sees no spaces at all (as under `herdr-app check`), it changes neither colors nor
  sprites.

## Built from

- Layout `layouts/main-pixel.json`: Herdr's `main` (0.1.11), cloned with `layout.clone`, with
  block `brand` set to `content.mod: "pixel-chrome"` and block `spaces` to
  `content.mod: "pixel-spaces"`. The footer block stays Herdr's `workspace-chrome`.
- `mods/pixel-chrome/`: Herdr's `workspace-chrome` cloned with `mod.clone`. `render()`'s header
  row has `waiting(t)` where the logo `image` and "Herdr" text were; `waiting` reads `agents()`,
  filters `blocked`/`done`, and returns an empty `div` or the pill (`svg("mods/pixel-chrome/
  bell.svg")` tinted by `text_color`, `on_mouse_down` stopping propagation, `on_click` →
  `run("pane.focus", { pane })`). The search button runs `mod.open { name: "pixel-picker" }`. It
  keeps only the glyphs it uses: `bell.svg` (new, 8×8 rects), Lucide `search`, `sidebar`,
  `settings`, `plus`.
- `mods/pixel-spaces/`: Herdr's `spaces` cloned. The avatar `h_flex` in `row()` holds
  `sprite(name, fill, ...)` with an `on_mouse_down("left")` that, when `e.modifiers.alt`, calls
  `cx.stop_propagation()` and writes the next name to shared `space.sprites`; `render()` assigns
  sprites with `assign()` after the colors, both inside `if (spaces().length)`.
- `sprites.js`: `SPRITES` (name to 8 strings of 8 chars), `NAMES`, `sprite()` (each row's runs of
  one character become one `div` of width `2 * run`, height 2), `assign()`.
- `mods/pixel-picker/`: Herdr's `picker` cloned. The heading row's avatar holds
  `sprite(shared("space.sprites")[id], spaceColor(t, w), ...)`; `spaceColors()` returns the stored
  colors untouched when `spaces()` is empty. It only reads `space.sprites`. Its own copy of
  `sprites.js`. `keys.json`: `"cmd-p": "mod.open:pixel-picker"` in `global`.
- Agent icons: `pixel-icons/claude.svg` and `pixel-icons/codex.svg` (`viewBox="0 0 9 9"`, one
  `rect` per run of a row, `shape-rendering="crispEdges"`), and `programs.json`:
  `"claude": { "icon": "pixel-icons/claude.svg", "tint": false }` and the same for `"codex"`.
  Herdr's program table gives every mod the same icon.
- `herdr` functions: `agents`, `spaces`, `shared`, `setShared`, `theme`, `run`, plus everything
  Herdr's mods use. No programs, no network, no files outside the mods.

## Watch out

- `herdr-app check` runs a mod with no spaces against the real shared state. Herdr's own
  bookkeeping drops every entry whose space isn't listed, so a check run would wipe
  `space.colors` and `space.sprites`. Keep the `spaces().length` guards in both mods.
- Herdr caches images by path: a changed SVG under the same name kept showing the old picture.
  Save a changed glyph under a new name.
- Option is Alt only where Herdr sends it as Alt (`option_as_alt`, "auto" on US/ABC layouts). On
  other keyboard layouts set `"option_as_alt": "on"` in settings.json for option-click.
- Herdr doesn't watch a symlinked mod folder; copy it rather than link it.
- A mod can't import from another mod's folder (`../pixel-spaces/sprites.js` is refused as
  outside the mod), so each mod carries `sprites.js`; keep the copies identical.
- Sprites must be exactly 8×8 and use only `.`, `x`, `o`, `k`.
- To see the bell without a real agent waiting, a terminal can report a status itself:
  `printf '\033]7501;state=blocked:app=demo\007'` (`app=claude-code` or `app=codex` shows that
  agent's icon).
- The agent icons stand for third-party products: keep them original glyphs, not traced logos,
  and keep the trademark notice in the README.

## Done when

- The header shows nothing while no agent is blocked or done, and a bell with the right count
  otherwise, in the blocked color when any is blocked; clicking it shows that agent.
- Every non-worktree space row shows a sprite in its color and no letter tile.
- A new space gets a sprite not yet used by the shown spaces (while fewer than 15 spaces show).
- Option-click on a sprite changes it to the next one and leaves the selected space as it was.
- `cmd-p` and the search button open the picker with each space's sprite beside its name.
- A Claude Code agent shows the orange pixel spark and a Codex agent the blue pixel `>_` in the
  agents list.
- `herdr-app check` passes for all three mods and leaves `space.colors` and `space.sprites` in
  `shared.json` unchanged.
