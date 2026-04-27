# Foxden

Hi, I'm Ethan.

I write about software engineering, AI-assisted development, personal knowledge systems, and small observations from daily life.

Foxden is my personal blog. It collects technical notes, workflow experiments, reading notes, and life observations that are worth returning to later.

## What I write about

- AI-assisted development and workflows
- Engineering practice, tools, and system design
- Personal knowledge management
- Books, people, and everyday observations

## Site

- URL: https://foxden.vault2049.xyz/
- Tech stack: Hugo + PaperMod
- Main sections: Home / Tech / Life / About

## Local development

Start the local development server:

```bash
hugo server -D
```

Visit: `http://localhost:1313`

## Create a new post

```bash
hugo new tech/your-post-slug.md
```

After writing, set `draft: false` in the front matter.

If the post belongs to a series, add a `series` field:

```toml
series = ["Claude-Code-Guide"]
```

## Deployment

Pushing to `main` triggers GitHub Actions to build and publish the site to GitHub Pages.
