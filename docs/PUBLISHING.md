# Publishing Blog Posts

Scaling Guacamole is a static website. There is no build process that converts
Markdown into published pages at deployment time.

Files in `docs/` are source material. A post is visible on the website only when
it has an HTML page in `posts/` and a corresponding card in `index.html`.

## Repository Structure

| Path | Purpose |
| --- | --- |
| `docs/<slug>.md` | Optional Markdown source for an article |
| `posts/<slug>.html` | Published article page |
| `images/` | Hero images and other article assets |
| `index.html` | Homepage and blog-card listing |
| `posts/TEMPLATE.html` | Basic page shell for a new article |
| `styles.css` | Shared site and article styling |
| `script.js` | Shared theme and interaction behavior |

Use the same lowercase, hyphenated slug for the source file, post, links, and
related image names where practical.

## Create or Refresh a Post

1. Write or update the source article in `docs/<slug>.md` when a Markdown source
  is used.
2. Add an optimized hero image to `images/`. Follow the format, dimensions,
  size, and naming guidance in `images/README.md`.
3. Create `posts/<slug>.html` from `posts/TEMPLATE.html` or the page shell of a
  recent post.
4. Convert the complete Markdown article into semantic HTML inside
  `<div class="post-body">`.
5. Update the page metadata, visible title, publication date, hero image, and
  alternative text.
6. Add or update the matching blog card near the top of the `.blog-grid` in
  `index.html`.
7. Preview and validate the changed pages before merging.

Refreshing an existing post should replace the complete article body. Do not
append new Markdown-derived content to stale HTML or leave sections from the
previous version behind.

## Article Requirements

Each published article must include:

- A concise `<meta name="description">` value.
- A descriptive `<title>` suitable for a browser tab.
- One visible `<h1 class="post-title">`.
- A `<time>` element whose `datetime` uses `YYYY-MM-DD`.
- A hero image with meaningful alternative text.
- Semantic headings beneath the page title: article sections start at `<h2>`.
- Relative paths such as `../styles.css`, `../images/<file>`, and
  `../script.js`.
- The shared header, theme toggle, footer, stylesheet, and script used by other
  posts.

Keep the homepage card title, URL, date, image, alternative text, and excerpt
synchronized with the article.

## Markdown Conversion Notes

Preserve the structure and meaning of the source:

- Convert paragraphs, lists, blockquotes, tables, and fenced code blocks to
  semantic HTML.
- Preserve fenced-code language classes, for example `language-python`.
- Escape source text and code correctly rather than inserting unsafe raw markup.
- Shift Markdown heading levels beneath the page's existing `<h1>`.
- Keep exactly one `<h1>` on the article page.

For articles containing TeX equations, retain the MathJax configuration used
by existing mathematical posts. Preserve `\(...\)` for inline equations and
`\[...\]` for display equations. Display equations may use the existing
`.math-display` wrapper for horizontal overflow on small screens.

## Images

Hero images should normally use a roughly 2:1 aspect ratio and remain under
500 KB. Prefer WebP or JPEG for photographs and PNG only when transparency or
graphic fidelity requires it.

Check both references when adding an image:

```html
<!-- posts/<slug>.html -->
<img src="../images/<image-file>" alt="Meaningful description">

<!-- index.html -->
<img src="images/<image-file>" alt="Meaningful description" loading="lazy">
```

## Local Preview

Run a static server from the repository root:

```bash
python -m http.server 8000
```

Open `http://localhost:8000/` and inspect both the homepage and the article at
`http://localhost:8000/posts/<slug>.html`.

Check desktop and mobile widths, light and dark themes, long code blocks,
tables, equations, image loading, and all links.

## Validation Checklist

- The article opens without missing assets or console errors.
- The homepage card opens the correct article.
- The article and card metadata agree.
- The page contains exactly one `<h1>` and a logical heading hierarchy.
- Images include useful alternative text and do not overflow.
- Code blocks, tables, and equations remain usable on narrow screens.
- No stale content from an earlier article version remains.
- Editor diagnostics report no errors in changed files.
- `git diff --check` passes.

For a focused HTML validation of a post, run:

```bash
npx --yes html-validate posts/<slug>.html
```

## Publication

Publishing is automatic after changes are merged or pushed to `main`:

1. The GitHub Pages workflow uploads the repository as a static artifact and
  deploys it.
2. The Azure Static Web Apps workflow deploys the same static content.

The GitHub Pages workflow can also be started manually from the Actions tab
with **Run workflow**. Pull requests to `main` receive an Azure Static Web Apps
preview deployment when the required repository secret is available.
