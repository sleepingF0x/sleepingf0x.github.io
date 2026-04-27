# Article Sidebar TOC Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a desktop-only sticky sidebar table of contents to long article pages so readers can jump to Markdown headings.

**Architecture:** Reuse PaperMod's existing `toc.html` partial and the site's existing anchored heading rendering. Wrap article content in a small two-column reading layout on pages where `ShowToc` is enabled and the page has headings, then style the right column as sticky on desktop and hidden on mobile.

**Tech Stack:** Hugo, PaperMod, Go templates, CSS in `assets/css/extended/foxden-white.css`.

---

## File Structure

- Modify `layouts/_default/single.html`: keep the current Foxden article header/footer, but wrap the body in `foxden-single__body`; render `partial "toc.html" .` in an aside when `.Param "ShowToc"` is true and `.TableOfContents` is not empty.
- Modify `assets/css/extended/foxden-white.css`: widen the single article shell on desktop, add the body grid, sticky right aside, TOC styling, and a mobile breakpoint that returns to the current single-column layout.

### Task 1: Render the sidebar TOC in the single article template

**Files:**
- Modify: `layouts/_default/single.html:18-24`

- [ ] **Step 1: Replace the current content block with a body grid and optional aside**

Change the current block:

```html
  <div class="post-content foxden-single__content md-content">
    {{ if not (.Param "disableAnchoredHeadings") }}
      {{ partial "anchored_headings.html" .Content }}
    {{ else }}
      {{ .Content }}
    {{ end }}
  </div>
```

To:

```html
  <div class="foxden-single__body">
    <div class="post-content foxden-single__content md-content">
      {{ if not (.Param "disableAnchoredHeadings") }}
        {{ partial "anchored_headings.html" .Content }}
      {{ else }}
        {{ .Content }}
      {{ end }}
    </div>

    {{ if and (.Param "ShowToc") (ne .TableOfContents "") }}
    <aside class="foxden-single__toc" aria-label="Article outline">
      {{ partial "toc.html" . }}
    </aside>
    {{ end }}
  </div>
```

- [ ] **Step 2: Run Hugo to verify the template compiles**

Run:

```bash
hugo
```

Expected: build succeeds with no template errors.

### Task 2: Style the desktop sticky sidebar and mobile fallback

**Files:**
- Modify: `assets/css/extended/foxden-white.css:572-610`

- [ ] **Step 1: Update the article shell and body layout styles**

Replace:

```css
.foxden-single {
  max-width: 760px;
  margin: 0 auto;
}
```

With:

```css
.foxden-single {
  max-width: 1080px;
  margin: 0 auto;
}
```

Add this after `.foxden-single__meta`:

```css
.foxden-single__body {
  display: grid;
  grid-template-columns: minmax(0, 760px) 220px;
  gap: 56px;
  align-items: start;
}
```

- [ ] **Step 2: Add sidebar TOC styling**

Add this after `.foxden-single__content`:

```css
.foxden-single__toc {
  position: sticky;
  top: 96px;
  max-height: calc(100vh - 128px);
  overflow: auto;
  padding-top: 38px;
  font-size: 13px;
}

.foxden-single__toc .toc {
  margin: 0;
  padding: 0 0 0 18px;
  border-left: 1px solid var(--border);
}

.foxden-single__toc .toc summary {
  display: block;
  margin-bottom: 12px;
  color: var(--tertiary);
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  cursor: default;
}

.foxden-single__toc .toc summary::-webkit-details-marker {
  display: none;
}

.foxden-single__toc .toc .inner {
  margin: 0;
  padding: 0;
}

.foxden-single__toc .toc ul {
  margin: 0;
  padding-left: 0;
  list-style: none;
}

.foxden-single__toc .toc ul ul {
  padding-left: 12px;
}

.foxden-single__toc .toc li {
  margin: 0;
  padding: 4px 0;
}

.foxden-single__toc .toc a {
  color: var(--secondary);
  text-decoration: none;
  line-height: 1.45;
}

.foxden-single__toc .toc a:hover,
.foxden-single__toc .toc a:focus-visible {
  color: var(--foxden-accent);
  text-decoration: underline;
  text-underline-offset: 3px;
}
```

- [ ] **Step 3: Add responsive fallback**

Inside the existing `@media (max-width: 768px)` block, add:

```css
  .foxden-single {
    max-width: 760px;
  }

  .foxden-single__body {
    display: block;
  }

  .foxden-single__toc {
    display: none;
  }
```

Expected: desktop can show a two-column article layout; mobile keeps the current single column and hides the TOC.

- [ ] **Step 4: Run Hugo to verify CSS is accepted**

Run:

```bash
hugo
```

Expected: build succeeds.

### Task 3: Manual verification

**Files:**
- Verify: generated article pages under `public/tech/`

- [ ] **Step 1: Start local server**

Run:

```bash
hugo server -D
```

Expected: server starts and prints a local URL, usually `http://localhost:1313/`.

- [ ] **Step 2: Open a long Tech article with headings**

Open a long article such as:

```text
http://localhost:1313/tech/skills/
```

Expected on desktop width: the article body appears on the left and a right sidebar TOC appears on the right.

- [ ] **Step 3: Click sidebar links**

Click several TOC entries.

Expected: the browser jumps to the corresponding Markdown heading. Existing anchored heading links still work.

- [ ] **Step 4: Check mobile width**

Resize below 768px.

Expected: the right TOC disappears and the article returns to one column.

---

## Self-Review

- Spec coverage: implements the chosen A option, reuses Markdown headings, desktop sticky sidebar, mobile single-column fallback, and no JavaScript.
- Placeholder scan: no TBD/TODO placeholders remain.
- Type/template consistency: class names used in the template match the CSS selectors.
