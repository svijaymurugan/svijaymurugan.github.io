# Editing this site

Notes to self for changing svijaymurugan.github.io.

This is a **Quarto** site, not Jekyll. Pages are `.qmd` files (Markdown plus a YAML header).
The rendered HTML is committed into `docs/`, and GitHub Pages serves that folder directly.

---

## The loop, every single time

```bash
cd ~/Desktop/Gold/svijaymurugan.github.io

quarto preview          # 1. edit with live reload; Ctrl-C when done
quarto render           # 2. rebuild docs/ for real
git add -A
git commit -m "what changed"
git push                # 3. live in ~1 minute
```

**Step 2 is the one that bites.** `quarto preview` shows you changes in the browser but does not
reliably update `docs/`. If you skip `quarto render`, you can edit, commit, push, and see
*absolutely nothing change* on the live site — because Pages serves `docs/`, and `docs/` still
holds the old HTML. Any time the live site looks stale, this is why.

Check before you push:

```bash
git status --short       # docs/*.html should appear among the changes
```

---

## Where everything lives

| What you want to change | File |
|---|---|
| Name, degree line, email, LinkedIn, GitHub | `index.qmd` (YAML header at the top) |
| About Me / career paragraphs | `index.qmd` (the body, between `:::{#hero-heading}` and `:::`) |
| DuQuantum headline, blurb, link | `files/includes/_duquantum.html` |
| Research page | `research.qmd` |
| CV — the PDF itself | replace `files/cv.pdf` |
| Site title, navbar, footer, URLs | `_quarto.yml` |
| Colors, fonts, spacing | `styles.css` |
| Headshot | replace `files/images/headshot.jpg` |

---

## The common edits

### Swap the CV

```bash
cp ~/Desktop/Gold/whatever-the-new-one-is.pdf files/cv.pdf
quarto render && git add -A && git commit -m "Update CV" && git push
```

Keep the name `files/cv.pdf`. Nothing else needs touching.

### Swap the headshot

Same idea — overwrite `files/images/headshot.jpg`, keeping that exact name, lowercase `.jpg`.

Two gotchas. Use a **square** image; the layout crops to a square and an off-centre face looks
wrong. And keep the extension lowercase: macOS treats `Headshot.JPG` and `headshot.jpg` as the
same file, but **GitHub's server does not**, so a capitalised extension works locally and gives
you a broken image on the live site. The template shipped with exactly that bug.

### Change the degree line under your photo

It is **not** in the page body. It is `subtitle:` in the `index.qmd` YAML header:

```yaml
subtitle: "B.S. Physics &amp; Mathematics, Duke University &middot; Expected 2027"
```

Quarto normally prints that next to your name; `styles.css` moves it below the links. Write
`&amp;` for `&` and `&middot;` for the dot separator.

### Change the About Me text

In `index.qmd`, everything between `:::{#hero-heading}` and the closing `:::`. Normal Markdown:
`**bold**`, `*italic*`, `[text](https://url)`. Blank line between paragraphs.

Leave the `:::` fences alone — they are what tells Quarto this text goes in the right-hand column
next to your photo.

### Update the DuQuantum announcement

`files/includes/_duquantum.html`. Change the headline text, the blurb, or the `href`.

Two things to leave alone, both commented in the file:

- It must be a `<div>`, **never** an `<aside>`. Quarto's CSS throws any `<aside>` into the right
  margin column — that is what made it sit squeezed to one side originally.
- No `column-page` / `column-body` class on it. Quarto flips the about block's own column class in
  response, and the two end up mismatched. The width is pinned in `styles.css` instead.

**When applications close, delete the announcement:** remove these three lines from the
`index.qmd` header.

```yaml
format:
  html:
    include-before-body: files/includes/_duquantum.html
```

The file stays on disk for next year.

### Fill in the Research page

`research.qmd` is scaffolding — every placeholder is marked `TODO`. Find them with:

```bash
grep -rn TODO *.qmd files/includes/_duquantum.html
```

Copy a `## TODO: Project title` block for each project, delete the spares. Until it is filled in,
that page shows placeholder text publicly.

### Add a whole new page

1. Create `talks.qmd` in the repo root:

   ```markdown
   ---
   title: "Talks"
   description-meta: "Talks and posters"
   title-block-banner: false
   ---

   Content here.
   ```

2. Add it to the navbar in `_quarto.yml`, under `navbar: left:`:

   ```yaml
      - text: Talks
        href: talks.qmd
   ```

Any `.qmd` in the root becomes a page automatically; the navbar entry is what makes it
*findable*. Only `.qmd` files are rendered — this file, `EDITING.md`, is Markdown precisely so it
stays out of the site.

### Hide a page without deleting it

Comment out its navbar lines in `_quarto.yml` with `#`. The page still exists at its URL, it just
leaves the menu. That is how `teaching.qmd`, `people.qmd`, `projects.qmd`, `publications.qmd`,
`contact.qmd` and the posts are currently parked — the commented blocks are still in `_quarto.yml`
if you ever want them back.

---

## Colors

Defined in `styles.css`. The purple used for "DuQuantum":

```css
.duq-brand { color: #6B21A8; }
```

If you swap it, keep it dark enough to read on white — aim for a contrast ratio of 4.5:1 or
better. A quick check: https://webaim.org/resources/contrastchecker/

---

## When something breaks

**Live site didn't change.** You skipped `quarto render`. Run it, commit `docs/`, push again.

**`quarto render` fails on `openpyxl`.** The template's publication pipeline is disabled
(commented out at the top of `_quarto.yml`). If you deliberately re-enable it, first run
`pip install openpyxl`.

**Broken image on the live site but fine locally.** Filename case. See the headshot note above.

**A link opens in a new tab when it shouldn't**, or vice versa — `link-external-newwindow` and
`link-external-filter` at the bottom of `_quarto.yml`.

**Check what Pages thinks:**

```bash
gh api repos/svijaymurugan/svijaymurugan.github.io/pages \
  --jq '{status, source: .source, url: .html_url}'
```

Should say `main` / `/docs`. If the live site shows a README instead of the site, Pages is
serving from the repo root rather than `docs/`.

**Undo everything since the last commit:**

```bash
git checkout -- .
```

---

## Quarto reference

- Markdown basics: https://quarto.org/docs/authoring/markdown-basics.html
- About pages (the homepage layout): https://quarto.org/docs/websites/website-about.html
- Navigation: https://quarto.org/docs/websites/website-navigation.html
