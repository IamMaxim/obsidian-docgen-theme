# Bullet journal checkboxes — design

**Date:** 2026-07-21
**Theme:** Docgen (Obsidian), `theme.css`
**Status:** approved, ready for implementation planning

## Goal

Style the alternate task markers Obsidian exposes as `data-task` — `[/]`, `[>]`,
`[!]`, `[?]` and so on — so a bullet-journal workflow reads correctly in the
Docgen theme. Today only the default checked/unchecked states are styled, via
the `--checkbox-*` variables at `theme.css:319`.

Prior art: the Things and Minimal themes, both of which the vault already has
installed. The mechanism below is the one they use; the visual treatment is
Docgen's own.

## Visual direction

**Treatment D — borderless annotations, boxed states.**

The 23 types split into two classes:

- **State** — `[ ]`, `[x]`, `[/]`, `[-]`. These keep the checkbox: a bordered
  box the reader can click. They answer "is this done?"
- **Annotation** — the other 19. These drop the box entirely and render as a
  bare colored icon at the full checkbox size. They answer "what kind of thing
  is this?"

Dropping the border for annotations is what makes them legible: inside a
bordered box the icon only gets about `size - 5px`, which is unreadable at
default text size. Without the box it gets the whole width.

State rendering:

| Marker | Rendering |
| --- | --- |
| `[ ]` | Empty bordered box (core default, unchanged) |
| `[x]` | Box filled with `--accent`, check knocked out in `--bg` (core default, unchanged) |
| `[/]` | Box with its **left half** filled in `--accent` |
| `[-]` | Bordered box with a muted 2px bar inside |

`[/]` and `[-]` are drawn as pseudo-element geometry, not icons. The `[-]` bar
reuses core's own indeterminate-checkbox measurements so it matches what
Obsidian already draws elsewhere:

```css
top: calc(var(--checkbox-size) / 2 - 2px);
width: calc(var(--checkbox-size) - 6px);
height: 2px;
```

## Sizing

**Do not set `--checkbox-size`.** Core defines it as `var(--font-text-size)`,
and derives both the vertical offset and the hanging indent from it:

```css
--checkbox-size: var(--font-text-size);   /* 16px at default text size */
--checkbox-radius: var(--radius-s);
top: 0.2em;
margin-inline-start: calc(var(--checkbox-size) * -1.5);
```

(Verified against the installed app at
`/Volumes/Samsung02/Applications/Obsidian.app`.)

Because the indent is a multiple of the checkbox size, hardcoding a pixel size
desyncs the indent from the text whenever the user changes their font size.
Inheriting the default means checkboxes scale with the reader's font-size
setting, baselines stay on the existing line box, and the theme needs no
line-height override. Confirmed visually at 16/17/18/20px: no awkward line
heights at any of them.

Icons are sized relative to the box, not in absolute pixels, so they scale with
it.

## Icon set — 23 types

Grouped by purpose. All icons use canonical **Lucide** paths — the family
Obsidian itself ships. Lucide's consistent 2px internal padding is what keeps
optical weight and centring uniform across the set; hand-drawn approximations
were tried first and were visibly unbalanced at small sizes.

| Group | Markers |
| --- | --- |
| State | `[ ]` `[x]` `[/]` `[-]` |
| Scheduling | `[>]` forwarded (arrow-right), `[<]` scheduled (calendar) |
| Emphasis | `[!]` important, `[?]` question, `[*]` star, `[I]` idea (lightbulb), `[f]` fire, `[w]` win (trophy) |
| Reference | `["]` quote, `[i]` info, `[b]` bookmark, `[l]` location (map-pin), `[n]` note (pencil), `[k]` key |
| Judgement & trend | `[p]` pro (thumbs-up), `[c]` con (thumbs-down), `[u]` trend-up, `[d]` trend-down, `[S]` savings (dollar) |

## Color

Existing Docgen tokens only — no palette additions. The `green` and `red`
values already exist as `--green-rgb` / `--red-rgb` for callouts and need
promoting to plain color tokens for reuse.

| Token | Used by |
| --- | --- |
| `--accent` | `[x]` `[/]` `[>]` `[I]` `[k]` |
| `--accent-hi` | `[*]` `[w]` |
| `--warn` | `[!]` `[f]` |
| `--info` | `[<]` `[?]` `[i]` |
| `--talk` | `["]` `[b]` |
| green | `[l]` `[p]` `[u]` `[S]` |
| red | `[c]` `[d]` |
| `--text-mute` | `[-]` |
| `--text-dim` | `[n]` |

Both dark and light mode, since the theme ships both. Tokens are already
mode-scoped, so the checkbox rules themselves are mode-independent — only the
promoted green/red tokens need a per-mode value.

## Mechanism

Each icon is a URL-encoded SVG `mask-image`; the visible color comes from
`background-color`, so one token drives the whole glyph. This is the standard
approach and what Things and Minimal both do.

Selector pattern, per marker — three forms, covering reading view (with and
without the paragraph wrapper) and Live Preview:

```css
input[data-task='X']:checked,
li[data-task='X'] > input:checked,
li[data-task='X'] > p > input:checked { … }
```

Every non-space marker renders as `:checked` in Obsidian, so `:checked` is
correct for all alternate types, not just `[x]`.

Annotation types share one reset block (drop border, radius, background image;
set mask sizing/position), then one small rule per type supplying its mask and
color. State types are handled separately since they keep their box.

## Line text treatment

- `[x]` — strikethrough, muted. Core already does this via
  `--checklist-done-decoration`; leave it alone.
- `[-]` — strikethrough, muted. Needs an explicit rule.
- `[>]`, `[<]` — dimmed to `--text-dim`, no strikethrough. The task is still
  open, just not now.
- All others — normal text color.

## Structure

One new section in `theme.css`, appended as **§4i · task markers**, after the
existing Live Preview parity block. The file's established structure —
tokens → variable bridge → signature overrides — is unchanged. The existing
`--checkbox-*` bridge block at `theme.css:319` stays as is; this section adds
to it rather than replacing it.

Ordering within the section: state types, then the shared annotation reset,
then annotation types grouped as in the table above.

## Out of scope

- Changing `--checkbox-size`, line-height, or list indentation.
- Plugin-specific task rendering (Tasks, Dataview).
- The `[P]` / `[M]` / `[D]` pull-request markers Things ships — not part of a
  bullet-journal workflow.

## Verification

Visual, in the app — this is a stylesheet with no test harness.

1. A note containing all 23 markers renders correctly in **reading view**.
2. The same note renders correctly in **Live Preview**.
3. Both **dark and light** mode.
4. Changing the font-size setting scales the checkboxes and keeps the indent
   aligned.
5. Clicking `[ ]` and `[x]` still toggles the task.

Design mockups from the brainstorming session are preserved under
`.superpowers/brainstorm/` (gitignored).
