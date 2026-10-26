# dsh-workpet-alice

An **alice** appearance for the [DeepSeek Harness](https://github.com/lastorder/dsh-workpet) **Work Pet** (`dsh-workpet`): a bob-haired young woman in round glasses and a brown blazer. She breathes while idle, bobs while the model writes, sways while tools run, shakes when something fails, hops when a turn finishes, and dangles when you drag her.

Artwork is converted from a Mori 2D sprite pack (`alice_half`): one frame per state, embedded as WebP, drawn into the pet's 100 × 80 SVG box.

[中文说明](README.zh.md)

## Requirements

- DeepSeek Harness (Desktop app or CLI-managed profile)
- The `dsh-workpet` bundle installed in the same profile — it provides `ctx.workpet` and ships the default kitten

## Install

### 1. DeepSeek Harness session (works for the Desktop app)

Ask the agent in any session to run:

```text
install_bundle dsh-workpet
install_bundle <absolute path to this repo>     # e.g. d:/POC/chatbot/work/dsh-workpet-alice
```

The first command is only needed once. The target may be an absolute path (as above) or a package name once this package is on npm.

### 2. CLI-managed profile

```sh
dsh plugin --profile <profile> add link:/absolute/path/to/dsh-workpet-alice
```

`link:` installs point at the folder, so later edits to `client.js` take effect on the next window refresh without reinstalling.

After installing, **refresh the Harness window**. alice appears next to the other pets, above the composer, with no extra setting.

## Using it

- She follows the session automatically: idle → breathing, thinking → tilting, writing → bobbing, tool calls → swaying, failure → shaking, done → hopping.
- **Click** her to pet her (she perks up), **drag** her to move her around; the position is remembered under the skin id `alice`.
- Open **Plugins → Work Pet** to see the appearance cards. The action buttons under alice's card play each of the 13 activities on demand, so you can check every pose without waiting for a real session to fall asleep or fail.
- Switch the bundle off in the Plugin Manager to remove her again; the default kitten is unaffected.

## Activity mapping

| `scene.activity` | When | Sprite state used | CSS motion |
|---|---|---|---|
| `absent` | No session open | sleeping | slow breathing |
| `idle` | Session idle | idle | breathing |
| `sleeping` | Idle > 70 s | sleeping | slow breathing |
| `thinking` | Model running, nothing written | thinking | gentle tilt |
| `writing` | Model streaming text | recording | small bob |
| `tool` | A tool call is running | walking | sway |
| `retrying` | Waiting to retry a failed request | error | shake |
| `awaiting` | Waiting for your approval | recording | small bob |
| `delegating` | Idle while subagents run | idle | looking around |
| `done` | Just finished a turn | done | hop |
| `error` | Just failed | error | shake |
| `petted` | You clicked her | done | soft bounce |
| `dragged` | You are dragging her | dragging | dangle |

The Work Pet draws the status bubble, tool badge, helper companions, and effects (`z`, `!`, `?`, hearts, sparkles) itself — this package never draws those.

## How it works

`client.js` is loaded **byte-for-byte** by Harness (no build step, no TypeScript). It contains three parts:

1. **ARTWORK** — `ART`, a map from state name to a `data:image/webp;base64,…` URI, plus `RATIO` (aspect ratio of each cutout) and `ACT` (activity → `[state, cssClass]`).
2. **ANIMATION** — the `CSS` string with `al-`-prefixed classes and `@keyframes` (breathe, bob, tilt, sway, shake, hop, look, pet, dangle), including a `prefers-reduced-motion` opt-out.
3. **PLUG-IN** — `ctx.effect` wiring: injects the stylesheet, registers the locale dictionaries (`workpetAlice`), and registers the skin:

```js
ctx.workpet.register({
  id: 'alice',        // stable id; her on-screen position is saved under it
  order: 10,          // 0 is the default kitten
  title: () => t('title'),
  description: () => t('description'),
  Figure: AliceFigure // ({ scene, compact }) => <svg viewBox="0 0 100 80">
})
```

`AliceFigure` returns plain SVG markup for the 100 × 80 box: a soft shadow ellipse plus a `<image>` of the current state, wrapped in a `<g>` whose class carries the motion. `compact` (helper companions) renders the same artwork minus scenery — the current art has no scenery, so both modes look identical.

Files:

| File | Purpose |
|---|---|
| `client.js` | Everything the browser runs: artwork, animation, registration |
| `index.js` | Empty host half; the Loader row needs a module to load |
| `cordis.patch.yml` | Inserts the `dsh-workpet-alice` row when the bundle is installed |
| `package.json` | Bundle manifest; `dsh.client.inject: ["dsh-workpet"]` makes Work Pet load first |
| `locale/{en,zh}.json` | Name and description in the Plugin Manager |
| `icon.svg` | Plugin Manager icon |

## Customizing

### Swap the artwork

The frames were cut from `alice_half` sheets (`1024×1024`, 4 × 4 grid, 256 px cells): frame 0 of each state, alpha-cropped, scaled to 160 px tall, encoded as WebP q88. The 8 source sheets live in [`assets/sprites/`](assets/sprites/); specs and provenance are in [`assets/README.md`](assets/README.md).

To use your own sheets:

1. Slice your sheet into 4 × 4 cells and pick one frame per state.
2. Crop to the alpha bounding box, resize to 160 px tall, encode as WebP, base64 it.
3. Replace the matching entry in `ART` and update `RATIO` with the cutout's width ÷ height.
4. Refresh the Harness window (restart the app if the old drawing sticks).

Keep each image under ~120 KB: every pet lives in the same page, and the whole `client.js` is loaded as one file.

### Change motion

Add or edit a class in `CSS` and reference it from `ACT`. Two rules matter in SVG:

- A CSS transform pivots on `transform-origin` measured in **viewBox units** (`style="transform-origin:50px 78px"`); without it the pivot is `(0,0)`.
- Never put an animation class on an element that also has a `transform="…"` attribute — nest a `<g>` instead, because the CSS transform replaces the attribute.

### Rename the pet

If you fork this for another character, change all of: `name` in `package.json`, the row `id`/`name` in `cordis.patch.yml`, the `id` passed to `window.__ModuleLoader__.load` (it must equal the package name), the skin `id` in `ctx.workpet.register`, and the `al-` CSS prefix (so two pets on one page never collide).

## Troubleshooting

| Symptom | Fix |
|---|---|
| Nothing appears | `dsh-workpet` must be installed in the **same** profile; check the Plugin Manager that both bundles are enabled |
| Old drawing after a refresh | Restart DeepSeek Harness so it reloads `client.js` |
| She shows up but never moves | You may have `prefers-reduced-motion` enabled at the OS level; motion is intentionally disabled then |
| Only `dsh-workpet-alice` is enabled | The kitten disappears only if `dsh-workpet` itself is off — re-enable it; appearances are additive, not replacements |
| Position resets after reinstall | Position is stored per skin `id`; changing `id` resets it |

## Uninstall

Turn the bundle off in the Plugin Manager, or remove it from the profile:

```text
install_bundle --remove dsh-workpet-alice
```

(CLI-managed profiles: `dsh plugin --profile <profile> remove dsh-workpet-alice`.)

## Development

```sh
npm run check     # node --check on client.js and index.js, no dependencies needed
```

There is no build step: edit `client.js`, refresh the Harness window, and use the action buttons in **Plugins → Work Pet** to preview all 13 activities.

## Publishing to npm

Once published, installs can use the package name instead of a path:

```sh
npm login
npm publish
# then: install_bundle dsh-workpet-alice
```

The repo follows the `dsh-workpet-parrot` conventions (bundle patch + `dsh.client` manifest), so a Trusted Publisher setup on npmjs.com can take over releases later.

## Credits & license

- Plugin structure: [dsh-workpet-parrot](https://github.com/lastorder/dsh-workpet-parrot) by lastorder, MIT.
- Artwork: the 8 source sheets in `assets/sprites/` (1024×1024, 4×4, half-body); pipeline and provenance in [assets/README.md](assets/README.md).
- Code and artwork: MIT, © 2026 Yu Dong — see [LICENSE](LICENSE).
