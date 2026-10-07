# Cliometrics Seminar

Website for the Cliometrics Seminar, a seminar series in Utrecht. Built with
[Hugo](https://gohugo.io/) and the [DoIt](https://hugodoit.pages.dev/) theme,
and published to GitHub Pages by the workflow in
[`.github/workflows/hugo.yml`](.github/workflows/hugo.yml).

**Live site:** <https://basm92.github.io/clioseminar/>

## Editing the site

Almost everything worth changing is a Markdown file in `content/`. Edit it on
GitHub (pencil icon), commit to `main`, and the site rebuilds and redeploys
automatically — usually within a minute or two.

| File | What it controls |
| :--- | :--- |
| `content/_index.md` | The main page: the painting, the slogan, and the schedule (which it includes from `content/schedule.md`) |
| `content/schedule.md` | The schedule table — the single source for both the Schedule page and the main page |
| `content/apply.md` | The Apply page |
| `content/contact.md` | The Contact page: organizers and the seminar's ambitions |

### Updating the schedule

The schedule is an ordinary Markdown table in **`content/schedule.md`**. That is
the only copy: the Schedule page shows it directly, and the main page pulls the
same table in with `{{< schedule >}}`, so editing the one file updates both
pages. Add, remove or edit a row there:

```markdown
| Mon 26 October 2026 | Amaury de Vicq (University of Groningen) | Aspirations, Investment Horizon... | 13:00–14:15 | Utrecht city centre |
```

Two things to leave alone: the header row and the separator row directly
underneath it, and the line `{.schedule}` just below the last row — that tag
drives the column widths. Everything else is free text. A row whose speaker is
not yet known is `TBA`.

Do not paste a second copy of the table into `content/_index.md`. The main page
is meant to hold `{{< schedule >}}` and nothing else where the table goes;
that is what keeps the two pages in step.

## Running it locally

You need [Hugo **extended**](https://gohugo.io/installation/) 0.146.0 or newer,
and the theme submodule:

```bash
git clone --recurse-submodules git@github.com:basm92/clioseminar.git
cd clioseminar
hugo server
```

Then open <http://localhost:1313/>. If you cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

`hugo` on its own builds the site into `public/` (git-ignored).

## Repository layout

```
content/            Markdown — this is where you edit
  _index.md           main page (includes the schedule via the shortcode)
  schedule.md         the schedule table — single source for both pages
  apply.md            Apply page
  contact.md          Contact page
hugo.toml           Site configuration: title, menu, theme settings
assets/css/         _custom.scss — the small amount of styling layered on the theme
layouts/shortcodes/ schedule.html — includes content/schedule.md where used
static/             Files served as-is: the painting, favicons
layouts/_markup/    Patched link and image render hooks — see below
docs/               Background documents, not part of the published site
themes/DoIt/        The theme (git submodule, pinned to a release)
```

## Notes for whoever maintains this next

- **The theme is a git submodule**, pinned to release `v1.0.2` in `themes/DoIt`.
  After changing it, commit the new pointer in the parent repo, or CI will keep
  building against the old one.

- **`layouts/_markup/render-link.html` and `render-image.html` override the
  theme.** Two reasons, both worth knowing before you touch them:

  1. The site is served from a subpath. Hugo treats a leading `/` in `relURL`
     as host-root-relative, so `[Apply](/apply/)` would resolve to
     `https://basm92.github.io/apply/` and 404. The overrides strip the leading
     slash before calling `relURL`, which resolves it against the site root
     *including* `/clioseminar/`. This is why you can keep writing ordinary
     `/apply/`-style links in Markdown and have them work.

  2. The theme resolves every Markdown link against Hugo's asset store, so a
     link to the home page — `[schedule](/)` — makes it try to publish the
     assets root, and the build fails with
     `Failed to publish Resource: open .../public: is a directory`. The link
     override skips that lookup for root-relative destinations.

  If a future theme release fixes either, these can go.

- **If you ever move the site to its own domain**, `baseURL` in `hugo.toml`
  stops having a subpath and both overrides become harmless no-ops — but
  `params.images` and any `static/` references are worth re-checking.

- **The painting is served from `static/images/clio_seminar_crop.jpg`** (a
  JPEG-compressed version of the 2 MB PNG master, which is kept in `docs/`).
  Replacing the image means producing a new JPEG of roughly the same size —
  around 1400 px wide — rather than dropping the original in.

- **The schedule table's column widths live in `assets/css/_custom.scss`**, keyed
  to the four columns of the table in `content/schedule.md` (Date, Speaker, Paper,
  Location). With `table-layout: fixed`, adding or removing a column there
  silently mis-sizes the rest rather than failing loudly, so update both together.

- **The main page's schedule comes from `content/schedule.md`** through the
  `schedule` shortcode in `layouts/shortcodes/schedule.html`, which includes that
  page's rendered content. Renaming or deleting `content/schedule.md` therefore
  fails the build on purpose, with a message naming the file — a missing page
  would otherwise drop the table from the main page without any visible error.

- **Page titles are left-aligned by an override.** The theme's
  `layouts/page.html` marks standalone pages `.special`, and `.special`
  right-aligns their title and subtitle — the theme's own design, and its demo
  site does the same, but it reads as a rendering fault on a content page.
  `_custom.scss` sets them back to `text-align: left`.
