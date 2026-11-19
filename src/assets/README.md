# Front-end styles

Everything visual is driven from four files, in this order:

| File | Role |
| --- | --- |
| `vars.scss` | the source of truth for SCSS values: palette, radius scale, type scale, motion curves, and the Bootstrap override points. It is imported **before** Bootstrap, so its values are what the framework compiles. |
| `tokens.scss` | the runtime layer: `@font-face` for the locally vendored woff2 files, then `--k-*` custom properties on `:root` (light) and `body.dark` (dark). Stylesheets and canvas renderers read these. |
| `motion.scss` | hover, press, focus ring, skip link, route transitions, the live indicator and the `prefers-reduced-motion` behaviour. |
| `app.scss` | component rules that apply to both themes. It imports the three files above plus Bootstrap, so theme switching needs no per-component work. |

## Rules

1. **No hex colours in components.** Use a `--k-*` token; if a value is missing, add it to `vars.scss` and
   `tokens.scss` once rather than repeating a literal.
2. **Do not add `.dark { … }` blocks.** `src/mixins/theme.js` toggles `body.dark`, which re-points every token.
   The legacy SCSS names components already import (`$secondary-text`, `$dark-bg`, `$highlight-white`,
   `$maintenance`, …) now resolve to those tokens, so an existing `color: $secondary-text` is already correct in
   both themes. They must stay plain `var()` references: passing one to an SCSS colour function (`rgba()`,
   `darken()`) breaks the build.
3. **Status is never colour alone.** A state renders a word plus its mark (`--k-up`, `--k-down`, `--k-warn`,
   `--k-maint` with `*-tint` backgrounds sized for 4.5:1 text).
4. **Motion:** animate transform, opacity and colour only - never width, height, margin or padding. Durations come
   from `--k-dur-*` and the single ease is `--k-ease`. Anything that keeps moving after 5 s needs a pause control.
5. **Canvas widgets read tokens too** (see `components/HeartbeatBar.vue`): read them from the element itself, not
   from `document.documentElement`, because the dark values live on `body.dark`.
6. **Fonts are served from `fonts/`** (OFL, latin subsets, weights 400 and 500 only) so an offline self-hosted
   instance renders identically. Keep `"Twemoji Country Flags"` first in the UI stack - the flag polyfill in
   `src/App.vue` depends on it.
7. **Numbers you compare** (uptime, timings, ids, durations) use `--k-font-mono` with `font-variant-numeric:
   tabular-nums` so columns line up.
8. **New user-facing strings** go in `src/lang/en.json` only and are used through `$t()`; the other locale files are
   maintained on Weblate and are protected by CI.
