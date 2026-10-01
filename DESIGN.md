# Design System: Spool

Spool is a self-hosted household watch tracker — movies, TV, and anime —
built as the private, ad-free alternative to Trakt/Simkl. It's a Django +
HTMX + Alpine.js server-rendered app (Tailwind 4 + daisyUI). Every value
below is the shipping theme in `static/src/app.css`, and the component
classes named here are defined there too — reuse them rather than
re-spelling utility strings in templates.

## 1. Direction

A dark, cinematic, media-first tracker. Artwork carries the visual
character; the interface around it stays quiet. Spool is a **tracker
first** and a browser second, so watch state, progress, ratings and
history stay easy to scan at a glance.

- **Content sits on the page.** Sections are separated by spacing,
  headings and the occasional hairline — not by wrapping everything in a
  bordered card. A card is used only when the thing *is* a card (a
  poster, a continue-watching tile, a recommendation, a sign-in form).
- **No cards inside cards**, no generic "four KPI boxes" rows, no
  glassmorphism, glow, neon, decorative gradients or blobs.
- **Layout follows content:** media → poster grids, episodes → compact
  rows, activity/history → timeline rows, continue watching → 16:9
  landscape cards, stats → grouped figures and charts, settings →
  restrained forms.
- **Motion is opt-in** (Settings → Appearance, off by default) and
  `prefers-reduced-motion` always wins. When on: 150–250ms, ease/ease-out,
  opacity/small translate/scale ≤ 1.02 only.

## 2. Color

Tokens live in the `spool` daisyUI theme and `@theme` block in `app.css`.
Never hard-code hexes in templates.

| Token | Value | Use |
|---|---|---|
| `base-100` | `#0e1014` | Page background (never pure black) |
| `base-200` | `#16181d` | Surfaces: real cards, inputs, menus |
| `base-300` | `#1f2229` | Raised/hover, progress tracks, chips |
| `line` | `#262930` | The one hairline color |
| `ink` / `base-content` | `#eceef2` | Primary text |
| `ink-dim` | `#9b9fae` | Secondary text, metadata |
| `ink-faint` | `#646979` | Tertiary text, placeholders |
| `primary` | `#e8a63c` | Spool amber — active nav, primary CTA, progress, focus, selection |
| `movie` / `tv` / `anime` | amber / `#8b85d6` / `#3fa9a0` | Media type — dots, chart series, split bars |
| `success` / `error` / `info` | `#5bd58a` / `#f0604a` / `#6fb7ce` | Watched/complete, destructive, informational/in progress |

Amber is used **selectively** so it stays important: the active nav item,
the one primary action, progress fills, focus rings, today/selected
states. It is not a decoration color. Movie/TV/Anime colors communicate
type, never decorate large areas. Rating-source brand marks (IMDb, RT,
Trakt, MAL, Metacritic, TMDB) appear only as small marks next to their
own scores.

## 3. Typography

- **Public Sans** for the whole interface, headings included
  (`--font-display` is Public Sans now). Page title `.page-title`
  28–32px/700; section heading `.section-title` 18–20px/650; card title
  `.media-title` 14px/600; metadata 12–13px `ink-dim`; body 13–15px.
- **Bebas Neue** (`.font-brand`) only for the SPOOL wordmark.
- **JetBrains Mono** only for episode codes (`S01E04`), timestamps where
  alignment matters, and technical values (API keys, versions). Ordinary
  numbers use Public Sans with `tabular-nums`.
- Title case over UPPERCASE; no letter-spaced uppercase labels.

## 4. Shape & depth

Radius scale (remapped in `@theme` so old and new markup agree): posters,
buttons, inputs 8px (`rounded-lg`); cards 10–12px (`rounded-xl`/`2xl`);
large panels ≤ 14px; full pills only for chips, filter chips, segmented
controls and avatars. Shadows only on floating things — menus/popovers
(`.menu-surface`), modals, the poster in the title hero.

## 5. Shell

- **Desktop (lg+):** slim left sidebar (`sidebar.html`) — wordmark, Home,
  Browse (Movies/TV/Anime), Library (Calendar/History/Lists/Stats/
  Activity), Settings at the bottom. Muted by default; active item gets a
  faint amber wash, amber icon, bright label. Collapsible to an icon rail
  (`html[data-nav="collapsed"]`, persisted in localStorage).
- **Top bar:** global search field (press `/` to focus), notifications,
  household friends, profile menu. No navigation links duplicated here.
- **Below lg:** docked bottom nav (Home, Browse, Calendar, Search, More)
  plus the More sheet; the top bar keeps the wordmark and account icons.
- **Content width:** `.page` caps at 1560px with responsive padding;
  reading-heavy pages (History, Activity feed, Notifications, Settings)
  constrain themselves further.

## 6. Components (app.css)

- **Page/section headers:** `.page-head`, `.page-title`, `.page-subtitle`,
  `.section`, `.section-head`, `.section-title`, `.section-count`,
  `.section-link` ("View all →"), `.subhead`.
- **Media cards:** `discover_tile.html` (TMDB items) and
  `poster_card.html` (library titles) share one structure — `.poster`
  artwork, `.media-title` + one `.meta` line underneath, a persistent
  `.card-state` marker for watched/in-progress, and a `.card-actions` bar
  (watched / list) revealed over the poster on hover/focus or first tap.
  `.card-actions` must stay a sibling of the overflow-hidden poster —
  its popovers are `position: fixed`.
- **Landscape cards:** Continue Watching (`poster_card.html` with
  `progress`) and Recently Watched (`watch_event_card.html`) — 16:9 art,
  bottom scrim with title/episode, 3px amber progress line.
- **Grids/rows:** `.media-grid` (responsive auto-fill posters),
  `.media-row` / `.landscape-row` (one row: sideways scroll on phones,
  clamped single grid row from sm), `.rows` (hairline-separated list).
- **Metadata:** `.meta` renders "2024 · Movie · 2h 14m" from plain spans;
  `.type-dot` + `media_label`/`media_dot_class` filters for media type.
  Prefer this over badges.
- **Chips:** `.chip` (+ `-primary/-success/-error/-info/-tv/-anime/
  -movie/-scrim`) only for state worth isolating: status, filler/recap,
  season finale, sync state. `.filter-chip` for removable active filters.
- **Controls:** `.seg`/`.seg-item` segmented control (type switches),
  `.tabs-line`/`.tab-line` underline tabs (in-page categories),
  `.toolbar`, `.field` (icon + input), daisyUI `btn` at `btn-sm`
  (`btn-primary` solid amber, `btn-soft` dark surface, `btn-ghost` quiet).
- **Ratings:** `pill_badges.html` — your rating first in amber, then each
  source's mark + score on one line, no boxed pills.
- **Episodes:** `title_episodes.html` — season poster picker, then one
  compact row per episode; watched rows recede, the first unwatched
  episode after a watched one is marked "Up next" (`.ep-row` CSS).
- **Empty states:** `.empty` + `.empty-title` — short heading, one plain
  sentence, optional action.
- **Figures:** `.figure` / `.figure-label` for headline numbers (Stats,
  This Month).

## 7. Anti-patterns

- No light mode, no pure black, no fourth media-type color.
- No bordered card around every section, no cards inside cards, no
  uppercase metadata pills where a `.meta` line would do.
- No glow, neon, glassmorphism, large decorative gradients or blobs.
- No idle/looping animation; nothing that moves unless the user acted.
- No AI-copy clichés; Spool's copy stays plain and specific.
