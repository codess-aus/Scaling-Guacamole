# Scaling Guacamole Project Guidelines

## Architecture

- This repository is a dependency-free static site built with HTML, CSS, and
  JavaScript. Do not introduce a framework or build system unless explicitly
  requested.
- `index.html` contains the homepage blog cards, `posts/` contains published
  articles, `docs/` contains optional Markdown source, and `images/` contains
  article assets.
- Markdown files in `docs/` are not published or converted automatically.

## Blog Publishing

- Follow `docs/PUBLISHING.md` whenever creating, publishing, or refreshing a
  blog post.
- Treat `docs/<slug>.md` as the authoritative article content when it exists.
  Replace the complete HTML article body during a refresh so stale sections are
  not retained.
- Preserve the shared page shell and existing site conventions. Start new pages
  from `posts/TEMPLATE.html` or a recent post.
- Keep the article metadata, hero image, date, title, URL, alternative text, and
  homepage card in sync.
- Article pages must have one visible `<h1>`; converted source headings begin
  at `<h2>`.
- Preserve fenced-code language classes, semantic tables and lists, and MathJax
  delimiters when converting Markdown.

## Editing Conventions

- Use relative asset and navigation paths consistent with neighboring files.
- Reuse `styles.css` and `script.js`; add shared styles only when the article
  content requires them.
- Maintain semantic HTML and accessible labels and alternative text.
- Keep changes focused and do not rewrite unrelated posts or user-authored content.

## Validation

- Preview both `index.html` and the changed article at desktop and mobile
  widths.
- Check light and dark themes, local assets, links, tables, code blocks, and
  equations when present.
- Run `npx --yes html-validate posts/<slug>.html` for changed posts and run
  `git diff --check` before finishing.
- Do not publish or merge automatically unless explicitly requested.
