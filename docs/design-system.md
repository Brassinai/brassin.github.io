# BrassinAI design system — version 1

This system is applied to the existing Jekyll site's home, products, blog, insights, and standard article pages. It draws on the public demos and documentation for [GSAP Vault's AI Research template](https://gsapvault.com/templates/ai-research-template), [Cybersecurity Company template](https://gsapvault.com/templates/cybersecurity-template), [Design Studio template](https://gsapvault.com/templates/design-studio-template), and [Multi-Page Developer Portfolio](https://gsapvault.com/templates/multi-page-developer-portfolio-template), plus Brassin's content and palette. The image and product layouts are original Brassin compositions; no paid template source or assets are used. Product detail and standalone service page patterns below are blueprints for pages that do not yet exist in this repository.

## Design idea

**Research made operational.** The recurring visual is a *system trace*: a precise line that moves from an input through a decision point to a deployed result. It can appear as a quiet static diagram in articles, an interactive capability map on the homepage, or a product architecture diagram. It must always explain a real system or process.

The site should feel like a technical practice with taste: editorial scale, instrument-like labels, hard evidence, and one memorable interaction per major page. Gold belongs to Brassin's own identity. The cream chapter gives long pages a change of pace; charcoal remains the default.

## What the references teach us

| Design rule | AI Research | Cybersecurity | Brassin interpretation |
| --- | --- | --- | --- |
| Opening | Large thesis headline plus one procedural object | Split hero with a live diagram and clear starting choices | Strong claim about production AI, paired with a Brassin system trace and a direct service or product route |
| Page rhythm | Alternating paper, dark, and saturated chapters | Flat panels, one signal block, repeated edge alignment | Charcoal base, occasional warm paper chapter, one gold signal panel at a time |
| Type roles | Large sans display, compact mono labels | Light sans headlines, italic asides, mono data | Sora display, Inter interface, Newsreader editorial passages, JetBrains Mono for real data and labels |
| Navigation | Sparse one-page sections | Seven real pages with shared shell and purposeful transitions | Real Jekyll links and shared header/footer; page transitions can be added later if they improve orientation |
| Interaction | Candidate selector demonstrates the subject | Diagram and service links answer a buyer's question | Interactions explain capabilities, systems, and products; no invented scores or decorative dashboards |
| Proof | Method and specimen record | Service detail and case-study timeline | Architecture, methods, shipped work, limitations, and measurable outcomes when verified |

For image and text, the **Design Studio** reference is useful because each project pairs an expressive image with a short introduction and optional deeper story. The **Multi-Page Developer Portfolio** is useful because its work index previews projects beside a readable list, while detail pages move from a lead product screen into captioned figures, decisions, and outcomes. The [Editorial Case Study section](https://gsapvault.com/sections/editorial-case-study-section) demonstrates an asymmetrical focal image, adjacent project facts, and a narrative/results spread that adapts to phones. Brassin combines those principles with a stable product-row anatomy and its own artwork.

## Foundation tokens

These are Brassin colours, not colours sampled from the paid templates. The production values live in [`assets/css/design-system.css`](../assets/css/design-system.css). Contrast values below are calculated against the paired solid backgrounds.

| Role | Token | Value | Use |
| --- | --- | --- | --- |
| Dark ground | `--ds-bg` | `#0B0907` | Default page background |
| Raised dark | `--ds-surface` | `#17130D` | Technical panels and selected cards |
| Paper | `--ds-paper` | `#F3F0E7` | Reading and explanatory chapters |
| Light text | `--ds-ink-light` | `#EDEAE3` | Main text on dark, 16.6:1 |
| Dark text | `--ds-ink-dark` | `#17130D` | Main text on paper, 16.2:1 |
| Muted text | `--ds-muted` | `#A09180` | Supporting text on dark, 6.5:1 |
| Gold signal | `--ds-gold` | `#C8952E` | Primary action, selected state, diagram signal; 7.4:1 with dark text |
| Deep gold | `--ds-gold-ink` | `#865E12` | Small gold text on paper, 5.1:1 |
| Dark rule | `--ds-rule-dark` | `#3A3023` | Dividers on dark |
| Paper rule | `--ds-rule-paper` | `#D4CCBD` | Dividers on paper |

**Colour discipline:** one signal treatment per viewport. On dark, gold can carry small text or a filled action. On paper, use deep gold for small text. Category colours in articles are metadata only; they should not compete with the site signal. Do not use gradients as the default panel fill.

### Colour application by context

The palette is a set of roles, not a requirement to show every swatch on every page. The main combinations are:

| Context | Background | Main text | Support and lines | Signal |
| --- | --- | --- | --- | --- |
| Standard dark chapter | Dark ground | Light text | Muted text, dark rule | Gold on one action, selected item, or diagram path |
| Raised technical panel | Raised dark | Light text | Muted text, dark rule | Gold for the active state or annotated path |
| Reading chapter | Paper | Dark text | Paper rule | Deep gold for small labels; gold for large decorative marks only |
| Gold signal panel | Gold | Dark text | Dark text at lower emphasis | The panel itself is the signal; do not add a second accent |
| Image frame | Ground or paper chosen for image contrast | Caption matches chapter | Hairline rule | Gold only for a meaningful callout or active figure control |

Dark ground, surface, paper, and the two inks are structural colours. Gold and deep gold are the same brand hue tuned for different contrast needs. Rules and muted text are support roles. This is why the preview's swatch row shows the six identity colours while the component styles also use the support tokens.

## Typography and hierarchy

Sora carries the brand claim and product names; Inter carries UI and short explanations; Newsreader carries reflective introductions and longer editorial text; JetBrains Mono carries real metadata. The table specifies rendered sizes at a 375px phone and a 1440px desktop. Intermediate widths interpolate with `clamp()` or the listed breakpoint values; do not scale every text style with viewport width.

| Role | Face / weight | Phone, 375px | Tablet, 768px | Desktop, 1440px | Line height / measure |
| --- | --- | --- | --- | --- | --- |
| Home hero | Sora 700 | 36px | 61px | 94px | 1.02; original claim wraps naturally |
| Page `h1` | Sora 600 | 40px | 52px | 68px | 1.06; max 13ch |
| Section `h2` | Sora 600 | 32px | 40px | 52px | 1.10; max 17ch |
| Product card title | Sora 600 | 27px | 32px | 34px | 1.12; max 25ch |
| Product detail subhead | Sora 600 | 24px | 28px | 32px | 1.20; max 22ch |
| Editorial lead | Newsreader 400 | 22px | 24px | 28px | 1.38; max 38ch |
| UI/body | Inter 400 | 16px | 17px | 18px | 1.55; max 62ch |
| Reading body | Newsreader 400 | 19px | 20px | 21px | 1.55; 60–70 characters |
| Caption/support | Inter 400 | 14px | 14px | 14px | 1.45; max 65ch |
| Eyebrow/status | JetBrains Mono 500 | 11px | 11px | 12px | 1.4; uppercase, letter spacing .08em |
| Button/link | Inter 600 | 14px | 14px | 14px | 1.2; sentence case |

The product-card content order is **category eyebrow → title → one-sentence promise → supporting explanation → status and action**. A product title is never set inside a busy image. Use a single page `h1`; product names on an index are `h2` or `h3` according to the page hierarchy. On phones, display names remain short while the full descriptor lives in body copy.

## Grid, spacing, and alignment

The production scale lives in [`assets/css/design-tokens.css`](../assets/css/design-tokens.css). Components in [`assets/css/design-system.css`](../assets/css/design-system.css) consume those tokens; page templates do not need one-off spacing values.

| Token group | Values | Purpose |
| --- | --- | --- |
| Base spacing | `--space-1` through `--space-12`: 4, 8, 12, 16, 24, 32, 48, 64, 80, 96, 112, 128px | A 4px scale for internal gaps and page rhythm |
| Semantic spacing | `--ds-section-space`, `--ds-section-end-space`, `--ds-grid-gap`, `--ds-article-gap`, `--ds-cta-space` | Consistent chapter, gallery, article, and CTA spacing |
| Width | `--ds-container: 1296px`, `--ds-reading: 700px`, `--page-gutter` | Shared page edge and comfortable reading measure |
| Shape | `--ds-radius-image: 14px`, `--ds-radius-control: 2px`, `--ds-radius-panel: 6px` | Soft image frames, restrained controls, simple panels |
| Interaction | `--ds-touch-size: 44px`, `--ds-button-size: 48px`, `--ds-motion-fast: 200ms` | Usable targets and consistent state changes |

| Layout rule | Phone, 360–599px | Tablet, 600–899px | Desktop, 900px+ |
| --- | --- | --- | --- |
| Page gutter | 24px | 32px | 48px |
| Content max | Fluid within gutters | Fluid within gutters | 1296px centered; reading column about 700px |
| Product gallery | One column | Two columns, featured item stacked | 12-column grid: featured 7/5, then paired 6/6 cards |
| Header height | 64px | 72px | 80px |
| Chapter top / bottom | 64px / 72px | 80px / 88px | 112px / 120px |
| Chapter heading to content | 48px | 64px | 64px |
| Gallery column gap | 24px | 24px | 32px |
| Featured image to copy | 24px | 24px | 48px |
| Product title to promise | 16px | 24px | 24px |
| Description to status/action | 24px | 32px | 32px |
| CTA vertical padding | 64px | 64px | 96px |

At 375px, 24px gutters leave 327px of content. Product and article images stack above their text. Body copy remains at least 16px and actions remain at least 44px high. The page uses the same left edge for headings, images, copy, and actions.

### Edge and shape policy

Product and article images use a **soft 14px frame** on desktop and **10px on phones**. The image frame has a restrained border and shadow on dark surfaces. Product gallery images do not have visible captions. Technical diagrams use `object-fit: contain` with internal padding so their labels are not clipped. Chapter bands stay straight and full bleed. Controls have a subtle 2px radius. This keeps photographic and rendered imagery inviting while the page grid and dividers remain precise.

## Buttons, links, and CTA layout

The existing booking link is the site-wide direct contact route. The original hero labels are preserved: `Contact us` and `View products`. The shared header and closing section say `Book a call`. Gold filled buttons use dark text, a 48px minimum height, 20px horizontal padding, and a 2px radius. A secondary outlined button or underlined text link follows only when there is a real second destination. Text links are at least 44px high on touch screens. Focus gets a 2px outline offset 4px.

Status is metadata, shown beside the destination-specific product link. Products without a live destination have no action link. Product images are never treated as links unless the whole card is an intentional article link.

| Location | Desktop | Phone |
| --- | --- | --- |
| Header | One compact gold booking button at the right | Booking action is the last mobile menu item |
| Hero | Original claim and paragraph, with primary and outlined secondary buttons | Original copy wraps naturally; both actions span the text column |
| Product card | Status and optional link at the foot of copy | Status and link follow the description with 24px breathing room |
| Article card | Category/author → headline → summary → underlined read link | Same order below the image; no fixed card height or cut-off title |
| Mid-page CTA | Dark band spans viewport; heading on 7 columns, copy/action on 5 | One-column band with full-width action |
| Final CTA | Gold band fills nearly a viewport; large headline above supporting copy and button | At least 70svh, with a full-width button near the bottom |

Final CTA anatomy: eyebrow (`Start a project`), one large heading, a short explanation, and one primary action. The gold section fills the viewport width and gains height from `min-height: min(820px, 90svh)` on desktop. It does not sit in a bordered card.

## Image art direction and product layouts

The image system is a **story system**: each visual advances the explanation. For every product, identify one lead visual and the question it answers. A lead image can be a genuine interface screenshot, a specific rendered product object, or a labelled system diagram. Supporting images should show a different state, workflow, output, or result. Use photography only when it depicts the actual product context, team, deployment, or customer setting. Avoid generic technology stock photos and unrelated landscapes as primary product imagery.

| Asset role | What it proves | Preferred frame | Crop and caption |
| --- | --- | --- | --- |
| Lead product view | What the product looks or feels like | Index 16:10; detail 3:2 or 16:10 | Subject occupies 60–80% of frame; detail-page captions may name the artifact |
| Interface / workflow | What a user can do | 16:10 screenshot at native ratio where possible | Keep labels legible; crop empty chrome, never crop the relevant state |
| Architecture diagram | How the system works | Flexible SVG, often full-width | Node labels and directional path; caption names components and any simplification |
| Outcome / evidence | What changed or was produced | 4:3 or 16:10 figure | Real number, date, source, and scope when a metric is shown |
| Context photograph | Where the product is used | 3:2 | Documentary, factual location or activity caption |

Use a controlled palette around the images: charcoal or paper ground, muted warm neutrals, gold only for real signals. Do not recolour a screenshot so strongly that its UI becomes misleading. Define the focal point in the art brief and protect it with a 10% safe area on all sides for responsive cropping. The same image may crop differently at desktop and mobile if the focal point remains intact; if it cannot, use a second mobile asset. On detail pages, keep captions outside images and aligned to the image's left edge. Alt text describes information in the image; captions provide context rather than repeat the alt text.

Export diagrams as SVG, rendered images or photos as optimised AVIF/WebP with an appropriate fallback, and screenshots as PNG/WebP when UI text needs fidelity. Supply intrinsic dimensions, responsive `srcset`/`sizes`, and lazy loading below the first viewport. The lead image on a product detail page loads eagerly. A technical point must also be available in text.

### Product index: responsive image grid

The product display uses one featured image-and-text story followed by paired cards. All stories share the same anatomy: **image → category → product name → promise → explanation → status and optional action**. The feature places image and copy beside each other on desktop; the paired items place text below the image. Nothing is overlaid on imagery.

```text
Desktop, 12-column gallery
┌────────────── featured image: 7 columns ──────────────┬──── copy: 5 columns ────┐
└───────────────────────────────────────────────────────┴─────────────────────────┘
┌──────── card: 6 columns ────────┬──────── card: 6 columns ────────┐
│ image / copy          │ image / copy          │
└─────────────────────────────────┴─────────────────────────────────┘
```

The homepage shows InferCloud, InferOps, and Debrief in that sequence. The products index shows all five products: one feature and two paired rows. A 1px rule starts each story. Images are 16:10, softly rounded, and use `cover` for product renders or `contain` for diagrams. The frame protects the subject; it never forces all source artwork into a square crop.

At 375px, every story stacks into one column. The image is 327px wide and roughly 204px high, then the copy after a 24px gap. The title is 27px and the promise 22px. Long product descriptions remain readable; no text sits over the image. Actions appear only when the product has a destination.

### Article cards: featured plus supporting grid

The homepage writing section begins with a full-width featured article. Its image takes 7 columns and its category, author, title, summary, and read action take 5. Four supporting cards follow in two columns. Each supporting card has a 16:10 image above its text. All cards start with a 1px rule and use 32px column gaps and 64px row gaps on desktop. Titles and descriptions are each clamped to three lines; full text remains in the linked article. At phone width the gallery becomes one column with 64px between cards. The blog and insights archives use the same paper intro and featured-plus-paired grid as the product index. Their cards show category and date, title, excerpt, and a consistent **Read the article** or **Read the insight** link. Filters sit above the grid; the first visible card becomes the feature when filtering. Archive images are locally hosted WebP with 800px and full-size variants in `srcset`; the first card loads eagerly and later cards load lazily. Each image, title, and read link points to the same article or insight.

### Product detail: image-led narrative

```text
Desktop: intro (title 7 cols / summary + status 5 cols)
         lead visual (12 cols) + caption
         problem / promise (5 cols) beside workflow image (7 cols)
         full-width system diagram + annotation and caption
         evidence image (7 cols) beside result/limitations (5 cols)
         next product or final CTA

Phone:  intro → lead visual → problem/promise → workflow image →
        system diagram (scroll-safe or simplified mobile SVG) → evidence → CTA
```

Use two to four substantive figures on a detail page; add more only when each answers a new question. Screenshots may span 12 columns to remain legible. Full-width diagrams can include an adjacent text legend; on phone, reflow or simplify the diagram rather than shrinking labels below 14px. Product details need a clear top claim, realistic product state, explanation, and evidence/limits. A product still in development can show authored concept visuals, explicitly captioned as such.

**Current asset audit:** `_data/products.yml` uses local visuals for all five stories. InferCloud and Debrief use existing concept renders exported as WebP; InferOps uses its original terminal SVG; Devocle uses the existing green product illustration also used in the writing archive; Compno uses its system diagram. Product-specific green is allowed inside Devocle artwork while the surrounding page chrome keeps the Brassin palette. Replace concept visuals with genuine interface states as products mature, while keeping the framing and caption system.

## Homepage hero

The homepage hero keeps the original headline, paragraph, and actions. It is centered horizontally and vertically on a quiet charcoal ground with a restrained warm glow. There is no decorative image or hero animation on desktop or phone. The centered claim carries the first viewport, and the actions remain readable and keyboard accessible.

## Page blueprints

| Page | Sequence | Signature |
| --- | --- | --- |
| Home | Thesis hero → 3 capability routes → selected products → how we work → verified evidence → recent research → contact | Centered, type-led claim |
| Products index | Page thesis → featured product story → standard/technical stories → optional proof strip → final CTA | Same image, status, title, description, action anatomy for every product |
| Product detail | Problem → product promise → actual workflow/architecture → proof and limitations → next action | Product-specific trace diagram |
| Services | Buyer question → capability groups → engagement method → selected work → contact | Clear starting routes, inspired by the cybersecurity demo |
| Insights / Blog | Featured item → filters → consistent article rows/cards → archive | Editorial hierarchy; low motion |
| Article / case study | Title and metadata → summary → body → figures/evidence → related reading → contact | Reading-first layout with disciplined figure captions |
| About | Point of view → people → method → selected work → contact | Editorial text and factual team information |

## Implementation in this repository

- [`assets/css/design-tokens.css`](../assets/css/design-tokens.css) is the production source for palette, typography, spacing, widths, shape, and motion values. [`assets/css/design-system.css`](../assets/css/design-system.css) implements the responsive components. Both load after legacy styles on the standalone homepage and Jekyll's shared layout.
- [`_includes/product-story.html`](../_includes/product-story.html) renders the same category, title, promise, description, status, image, and optional action in the homepage and products grids. [`_data/products.yml`](../_data/products.yml) supplies its content and image treatment.
- [`_includes/final-cta.html`](../_includes/final-cta.html) supplies the gold closing action on home, products, blog, and insights. Shared layout pages use the existing header and footer includes; the standalone homepage keeps its own matching shell.
- The product detail and standalone service page patterns above are ready for future pages when there is enough verified content and imagery to populate them.

### Source boundary

The public demos reveal visual and interaction principles. Exact private template CSS, JavaScript, and assets are available through GSAP Vault's paid downloads. This document defines an original Brassin system using the public references as design research.
