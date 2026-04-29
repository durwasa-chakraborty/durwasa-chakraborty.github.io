# Anachronistic Monk

Source for [durwasa-chakraborty.github.io](https://durwasa-chakraborty.github.io).
Hugo + [Blowfish](https://blowfish.page). Pushes to `main` deploy automatically
via GitHub Actions.

## Write a new post

```bash
hugo new content/posts/my-new-post.md
```

That seeds the file from `archetypes/default.md` with today's date and
`draft = true`. Edit the frontmatter:

```toml
+++
title = 'Catchy Title'
date = 2026-04-29T10:00:00+05:30
draft = false
author = 'durwasa'
tags = ['programming', 'essays']
# math = true        # only set if the post contains $...$ or $$...$$
# comments = true    # opt in to Disqus on this post
+++

Your content goes here. Markdown is fine.
```

Then write your post body below the `+++` block.

## Preview locally

```bash
hugo server -D
```

`-D` includes drafts. Open <http://localhost:1313>. Save the file and
the page reloads automatically.

## Deploy

```bash
git add content/posts/my-new-post.md   # plus any new images in static/
git commit -m "post: <slug>"
git push
```

The GitHub Action (`.github/workflows/hugo.yml`) builds and publishes within
a couple of minutes. Watch it with `gh run watch` or in the Actions tab.

## Tags

Use lowercase, plural where natural, no spaces. Reuse what's already in the
tag cloud at <https://durwasa-chakraborty.github.io/tags/> before inventing
new ones — current set: `programming`, `systems`, `essays`, `language`,
`literature`, `politics`, `society`, `academia`, `ai`, `photography`.

## Math / LaTeX

Just write LaTeX inline with the delimiters you already know:

- `$x^2$` for inline
- `$$\int_0^1 x\,dx$$` or `\[ ... \]` for display

KaTeX is loaded site-wide because `params.math = true` in
`config/_default/params.toml`, so you don't need to set anything per-post.
The wiring lives in `layouts/partials/extend-head-uncached.html`.

## Images

Put images in `static/images/` and reference them as `/images/foo.png` in
markdown. They'll be copied to the site root at build time.

## Update the theme

Blowfish is a git submodule. To pick up upstream theme changes:

```bash
git submodule update --remote themes/blowfish
git add themes/blowfish
git commit -m "theme: bump blowfish"
git push
```

If you `git clone` this repo on a new machine, run
`git submodule update --init --recursive` once after cloning so
`themes/blowfish/` populates.

## Layout

```
config/_default/   site config (hugo, params, languages, menus, markup)
content/posts/     one .md file per post
layouts/partials/  local overrides (only the math wiring lives here)
static/images/     images referenced as /images/...
themes/blowfish/   theme as a submodule (do not edit in place)
.github/workflows/ Hugo build + Pages deploy
```
