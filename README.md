# Engineering & Systems Journal

Lalit's technical journal at <https://lalitb22.github.io/>, including a 10-day,
30-hour practical AI Engineering learning journey. Existing posts remain in
`_posts/`; the roadmap and chapters live in `_pages/` with explicit URLs.

## Theme and deployment

This site uses **Minimal Mistakes 4.24.0 via remote theme**, pinned in
`_config.yml`. The Gemfile uses `github-pages` and explicitly installs
`jekyll-include-cache`. Do not also install `minimal-mistakes-jekyll` or add a
`theme:` setting. See the theme's
[GitHub Pages setup](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/).

There is no deployment workflow in this checkout. In GitHub **Settings → Pages**,
verify **Deploy from a branch**, the intended publishing branch, and **/ (root)**.
The working branch is `migrate-to-minimal-mistakes`; it need not be the production
publishing branch. The unauthenticated Pages API returned 404 during this update,
so an administrator must verify the actual settings.

If Pages uses GitHub Actions, verify that its workflow builds Jekyll with this
Gemfile and uploads `_site`. This checkout does not provide such a workflow.

## Publishing

No local Ruby installation or build is required. Commit and push the website
changes, then check the GitHub Pages build in the repository's Actions tab.
Deployment occurs when the changes reach the configured publishing branch.

## Adding chapters

1. Add a Markdown page in `_pages/` with `layout: single` and an explicit URL.
2. Enable `toc`, `toc_sticky`, and `sidebar: {nav: ai-engineering}`.
3. Update `_data/navigation.yml`, the roadmap, and the previous chapter's next
   link when the new chapter exists. Keep unpublished chapters marked planned.
4. Distinguish measured observations from illustrative code and record limits.
5. Review the diff, then check the GitHub Pages build and published pages.

Preserve existing post filenames, categories, dates, and global permalink
settings to retain article URLs. This update leaves both article files unchanged.
