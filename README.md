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
| `content/_index.md` | The main page: the painting, the slogan, and the schedule table |
| `content/apply.md` | The Apply page |
| `content/contact.md` | The Contact page: organizers and the seminar's ambitions |

### Updating the schedule

The schedule is an ordinary Markdown table in `content/_index.md`. Add, remove
or edit a row:

```markdown
| Mon 26 October 2026 | Amaury de Vicq (University of Groningen) | Aspirations, Investment Horizon... | 13:00–14:15 | Utrecht city centre |
```

Two things to leave alone: the header row and the separator row directly
underneath it, and the line `{.schedule}` just below the last row — that tag
drives the column widths. Everything else is free text. A row whose speaker is
not yet known is `TBA`.

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
  _index.md           main page, including the schedule table
  apply.md            Apply page
  contact.md          Contact page
hugo.toml           Site configuration: title, menu, theme settings
assets/css/         _custom.scss — the small amount of styling layered on the theme
static/             Files served as-is: the painting, favicons
layouts/_markup/    A patched link render hook — see below
docs/               Background documents, not part of the published site
themes/DoIt/        The theme (git submodule, pinned to a release)
```

## Notes for whoever maintains this next

- **The theme is a git submodule**, pinned to release `v1.0.2` in `themes/DoIt`.
  After changing it, commit the new pointer in the parent repo, or CI will keep
  building against the old one.

- **`layouts/_markup/render-link.html` overrides the theme.** The theme resolves
  every Markdown link against Hugo's asset store, so an ordinary link to the home
  page — `[schedule](/)` — makes it try to publish the assets root and the build
  fails with `Failed to publish Resource: open .../public: is a directory`. The
  override skips that lookup for the `""` and `/` destinations. If a future theme
  release fixes this, the override can go.

- **The painting is served from `static/images/clio_seminar_crop.jpg`** (a
  JPEG-compressed version of the 2 MB PNG master, which is kept in `docs/`).
  Replacing the image means producing a new JPEG of roughly the same size —
  around 1400 px wide — rather than dropping the original in.

- **Page titles are right-aligned.** That is the DoIt theme's own styling for
  standalone pages, not a bug; the theme's demo site does the same. To
  left-align them, add to `assets/css/_custom.scss`:

  ```scss
  .special .single-title,
  .special .single-subtitle {
    text-align: left;
  }
  ```
