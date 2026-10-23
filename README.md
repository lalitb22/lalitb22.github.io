# Engineering & Systems Journal

Lalit's journal at <https://lalitb22.github.io/>. The design uses the Soft Slate &
Muted Blue palette, a profile sidebar, article cards, a data-driven AI roadmap,
and discovery panels. Articles have a separate reading layout.

## Theme and deployment

Minimal Mistakes **4.24.0** remains pinned as a remote theme in `_config.yml`.
The Gemfile uses `github-pages` and `jekyll-include-cache`; do not add a theme gem
or a `theme:` setting alongside the remote theme. SEO metadata, syntax highlighting,
RSS, sitemap, and the table-of-contents include remain theme/Jekyll features.
There is no separate frontend build system or new deployment requirement.

GitHub Pages deploys `main` through its existing dynamic **pages build and
deployment** workflow. There is no custom workflow in this checkout. Keep this
working deployment method; no Netlify setup is needed.

## Design files

- `_layouts/journal-base.html`: document shell, theme head, SEO, and scripts.
- `_layouts/journal.html`: profile, content, and discovery columns.
- `_layouts/single.html`: article typography, TOC, tags, sharing, and adjacent posts.
- `_includes/head/custom.html`: custom stylesheet and explicit light color scheme.
- `_includes/journal/`: sidebar, navigation, feature, post card, roadmap, panels.
- `assets/css/main.scss`: standard Minimal Mistakes SCSS entry point with palette overrides.
- `assets/css/journal.css`: shared CSS tokens, components, and responsive rules.
- `_data/navigation.yml`: global navigation and chapter navigation data.
- `_data/ai-roadmap.yml`: ten modules, descriptions, publication status, and URLs.
- `index.html`: homepage sections derived from posts and roadmap data.
- `_pages/`: AI landing page, preserved Day 1 tutorial, categories, tags, archives, About.

The profile expects a PNG at `assets/images/avatar.png`, configured through
`author.avatar` in `_config.yml`. An initials mark is used if that setting is absent.
Only the existing GitHub account and RSS link are used; no contact details are invented.

## Content maintenance

Existing post files, categories, tags, dates, and URL generation are unchanged.
Day 1's technical content and recorded experiments are preserved; only its next
chapter note was aligned to the new roadmap. Days 1 and 2 are published. Day 2 lives in `_posts/` with an explicit series URL
and appears in the homepage, archives, categories, tags, and RSS feed.

When publishing a chapter, add its page with `layout: single` and an explicit
permalink. Then update its roadmap record to `status: published` with that exact
URL. Upcoming records have `url: null` and render without links. Keep experimental
results distinct from illustrative examples.

Posts can set `last_modified_at` to an ISO date to appear in Recently Updated.
Until update dates exist, that panel uses publication dates with an explanatory
label. Topics are ranked by article frequency; their counts are not displayed.

## Verification and optional preview

At the user's request, no local Jekyll build, dependency installation, preview
server, or visual viewport test was completed for this redesign. Source checks
cover YAML/front matter, local navigation/roadmap destinations, Liquid block
balance, unchanged post files, and whitespace. Actual generated HTML and rendered
layout should be reviewed before publishing.

If you later choose to preview in an environment with the project dependencies
already available:

```sh
bundle exec jekyll serve
```

Open <http://localhost:4000/> and check the homepage, Day 1, both posts, categories,
tags, archives, and About at 375 px, 768 px, and 1440 px. To build without serving:

```sh
bundle exec jekyll build
```

These commands are optional instructions; they were not run during this redesign.
The original migration verification notes above describe that earlier pass.
For the sidebar and copy refinements, the local build was attempted but Jekyll
dependencies were unavailable. The existing GitHub Pages workflow verifies the
commit pushed to `main`; check its result in the repository Actions tab.
