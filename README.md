# BrassinAI Blog

A technical blog built with **Jekyll** and deployed on **GitHub Pages**. Articles are written in Markdown and auto-build on push.

## 🏗 Project Structure

```
├── _config.yml          # Jekyll configuration
├── _layouts/            # HTML templates
│   ├── default.html     # Base layout (header + footer)
│   └── post.html        # Individual article layout
├── _includes/           # Reusable partials
│   ├── header.html
│   └── footer.html
├── _posts/              # 📝 Markdown articles go here
│   ├── 2025-11-01-exploring-vision-language-models.md
│   ├── 2025-11-01-getting-started-with-hpc.md
│   ├── 2025-11-01-inside-the-compiler.md
│   ├── 2025-11-01-notes-on-simplicity-and-documentation.md
│   └── 2025-11-01-weekend-hack-building-a-static-blog.md
├── assets/css/style.css # Site styling
├── images/              # Static images
├── index.html           # Homepage with category filters
├── about.md             # About page
├── CNAME                # Custom domain
├── Gemfile              # Ruby dependencies
└── README.md            # This file
```

## 🚀 Running Locally

### Prerequisites

- **Ruby** (>= 2.7) - check with `ruby -v`
- **Bundler** - install with `gem install bundler`

### Setup & Serve

```bash
# Install dependencies
bundle install

# Start local dev server (with live reload)
bundle exec jekyll serve --livereload
```

Open **http://localhost:4000** in your browser.

### Build only (no server)

```bash
bundle exec jekyll build
```

Output goes to `_site/`.

## ✍️ Writing a New Article

1. Create a Markdown file in `_posts/`:

   ```
   _posts/YYYY-MM-DD-your-post-slug.md
   ```

2. Add front matter at the top:

   ```yaml
   ---
   layout: post
   title: "Your Article Title"
   date: 2026-02-17
   category: vlm          # one of: vlm, hpc, compiler, general, other
   author: Your Name
   excerpt: "A brief summary shown on the homepage card."
   ---
   ```

3. Write your content in Markdown below the front matter.

4. Push to `main` - GitHub Pages will build and deploy automatically.

## 🖥 Presentations

The site supports two slide sources:

- Markdown slides: use `layout: slide` and separate slides with `---`.
- PDF presentations: upload the `.pdf`, then use `layout: slide` with `slide_source: pdf`.

### Markdown slide post

```yaml
---
layout: slide
title: "Your Slide Title"
date: 2026-06-11
category: hpc
author: BrassinAI
excerpt: "Short description for the insights page."
---
```

Separate each slide with a horizontal rule:

```markdown
# Title Slide

---

## Second Slide
```

### PDF presentation post

```yaml
---
layout: slide
slide_source: pdf
title: "Example Presentation"
date: 2026-06-11
category: general
author: BrassinAI
excerpt: "Short description for the insights page."
pdf_file: /assets/presentations/example.pdf
---
```

GitHub Pages serves the `.pdf` file and the slide page renders it page-by-page as a presentation. 

### Supported Categories

| Category   | Topics                                        |
|------------|-----------------------------------------------|
| `vlm`      | Vision-Language Models, multimodal AI          |
| `hpc`      | CUDA, MPI, distributed computing              |
| `compiler` | IR, SSA, optimization, language design         |
| `general`  | Productivity, docs, engineering culture        |
| `other`    | Miscellaneous experiments & projects           |

## 🌐 Deployment

This site deploys automatically via **GitHub Pages** on every push to `main`. The custom domain `brassinai.com` is configured via the `CNAME` file.

No CI/CD setup required - GitHub Pages has built-in Jekyll support.

## 📄 License

Content © BrassinAI. All rights reserved.
