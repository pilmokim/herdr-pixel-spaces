# Herdr Pixel Spaces

Pixel-art avatars for the spaces list of the [Herdr](https://herdr.dev) desktop app. Instead of a
colored tile with the space's first letter, each space gets a little 8×8 sprite in its color: a
slime, a ghost, a frog, a potion...

![Pixel sprites in the spaces list](preview.png)

- **15 sprites.** slime, ghost, cat, robot, mushroom, octopus, skull, heart, potion, frog, rocket,
  terminal, coffee, bomb and star. All drawn for this mod, not taken from any game.
- **One per space, kept.** A new space gets the sprite used least so far, and keeps it.
- **Your pick.** Option-click (Alt-click) a sprite to step to the next one. A plain click still
  opens the space.
- **Still Herdr's spaces list.** It's Herdr's own `spaces` mod with only the avatar changed:
  colors, worktrees under their repo, branch and ahead/behind, a group per machine, the
  right-click menu and drag to reorder all work as before.
- **In the Go to picker too.** `pixel-picker` is Herdr's `picker` (`cmd-p`) with the same
  sprites next to each space's name.

![All 15 sprites](sprites.png)

## Install

Ask your agent: "add https://github.com/pilmokim/herdr-pixel-spaces to my Herdr app". Or by hand:

```sh
git clone https://github.com/pilmokim/herdr-pixel-spaces.git
cp -R herdr-pixel-spaces/mods/pixel-spaces herdr-pixel-spaces/mods/pixel-picker ~/.config/herdr-app/mods/
cp herdr-pixel-spaces/layouts/main-pixel.json ~/.config/herdr-app/layouts/
herdr-app do layout.switch '{"name":"main-pixel"}'
```

`main-pixel` is Herdr's own `main` layout (0.1.11) with the spaces list swapped for this one.
`cmd-shift-l` switches back to any other layout.

For the picker, point `cmd-p` at it in `~/.config/herdr-app/keys.json` (this repo's `keys.json`
has the line to merge):

```json
{ "global": { "cmd-p": "mod.open:pixel-picker" } }
```

The sidebar's search button still opens Herdr's own picker.

### Your own layout

In a layout of your own, point the spaces block at this mod:

```json
{ "id": "spaces", "type": "block", "size": { "weight": 1 }, "min": 90, "content": { "mod": "pixel-spaces" } }
```

## Draw your own sprite

Sprites live in `mods/pixel-spaces/sprites.js`, each one 8 strings of 8 characters:

```js
ghost: [
  "..xxxx..",
  ".xxxxxx.",
  "xxxxxxxx",
  "xokxxokx",
  "xxxxxxxx",
  "xxxxxxxx",
  "xxxxxxxx",
  "xx.xx.xx",
],
```

`x` is the space's color, `o` white, `k` near-black, `.` clear. Add an entry and save: Herdr
reloads the mod at once and the new sprite joins the rotation. `mods/pixel-picker/sprites.js` is
a copy of the same file (a Herdr mod can't import from another mod's folder): change both.

## How it's kept

Each space's sprite is stored by space id in Herdr's shared mod state (`space.sprites`), next to
the colors Herdr already keeps there (`space.colors`). Unlike the shipped mods, these two leave
both alone when they see no spaces at all, so a `herdr-app check` run doesn't wipe everyone's
colors.

## License

Apache-2.0. `mods/pixel-spaces/main.js` and `mods/pixel-picker/main.js` are Herdr's `spaces` and
`picker` mods (Apache-2.0) with the avatar changed, as their headers say; `sprites.js` and the
sprites are new.
