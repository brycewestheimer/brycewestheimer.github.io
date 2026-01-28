# Personal Website Overhaul - Implementation Plan

Complete redesign of [brycewestheimer.github.io](https://brycewestheimer.github.io) using the Astro framework.

## User Decisions ✅

| Decision | Choice |
|----------|--------|
| **Framework** | Migrate to **Astro** |
| **Color Scheme** | Deep teal/navy + warm accent |
| **Research Page** | Keep separate |
| **Tutorials** | Keep in primary nav |
| **CV/Resume** | Add as downloadable PDF |
| **Profile Photo** | Download from LinkedIn |

---

## Navigation (8 Primary Items)

1. About — Professional bio, timeline, skills
2. Research — Focus areas and collaborations
3. Publications — Academic work with DOIs
4. Projects — Software showcase
5. Blog — Technical writing
6. Tutorials — Guides and walkthroughs
7. CV — Downloadable PDF
8. Contact — Professional contact options

---

## Astro Framework Migration

### Why Astro?

- **Zero JS by default** — Ultra-fast page loads
- **Content Collections** — Type-safe Markdown handling
- **GitHub Pages ready** — Static output, simple deployment
- **Modern tooling** — TypeScript, Tailwind CSS, component islands

### New Project Structure

```
brycewestheimer.github.io/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── Hero.astro
│   │   ├── ProjectCard.astro
│   │   ├── PublicationCard.astro
│   │   ├── Timeline.astro
│   │   └── SkillsGrid.astro
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── PostLayout.astro
│   ├── pages/
│   │   ├── index.astro      # Home page
│   │   ├── about.astro
│   │   ├── research.astro
│   │   ├── publications.astro
│   │   ├── projects.astro
│   │   ├── blog/
│   │   │   └── [...slug].astro
│   │   ├── tutorials/
│   │   │   └── [...slug].astro
│   │   ├── cv.astro
│   │   └── contact.astro
│   ├── content/
│   │   ├── blog/            # Blog posts (Markdown)
│   │   ├── tutorials/       # Tutorial posts
│   │   └── config.ts        # Content collection schemas
│   └── styles/
│       └── global.css       # Design system
├── public/
│   ├── cv.pdf               # Downloadable CV
│   ├── profile.jpg          # Profile photo
│   └── favicon.ico
├── astro.config.mjs
├── tailwind.config.mjs
└── package.json
```

---

## Design System

### Color Palette (Option A)

| Token | Light Mode | Dark Mode |
|-------|------------|-----------|
| **Primary** | `#0F4C5C` (deep teal) | `#1A6B7D` |
| **Accent** | `#E8AA42` (warm gold) | `#F0B856` |
| **Background** | `#FAFAFA` | `#0D1117` |
| **Surface** | `#FFFFFF` | `#161B22` |
| **Text** | `#1F2937` | `#F0F6FC` |
| **Muted** | `#6B7280` | `#8B949E` |

### Typography

- **Headings**: Inter (600-700 weight)
- **Body**: Inter (400 weight)
- **Code**: JetBrains Mono

---

## Page Designs

### Home Page
- Hero with name, title, animated gradient background
- CTA buttons: "View Projects" / "Read Publications"
- Featured projects grid (3 cards)
- Latest blog post preview

### About Page
- Profile photo + bio summary
- Career timeline (visual)
- Education cards
- Skills grid with technology icons
- Awards & recognition highlights

### Projects Page
- Featured project hero
- Project cards with:
  - Status badge (Active/Preview/Development)
  - Tech stack tags
  - GitHub + demo links

### Publications Page
- Year-grouped publication cards
- DOI links as buttons
- Collapsible abstracts/highlights

### CV Page
- Embedded CV preview (optional)
- Download PDF button

---

## Content Migration

Migrate existing Jekyll content to Astro Content Collections:

| Source | Destination |
|--------|-------------|
| `_posts/*.md` | `src/content/blog/` |
| `_tutorials/*.md` | `src/content/tutorials/` |
| [about.md](file://wsl.localhost/Ubuntu/home/westh/portfolio/programming/brycewestheimer.github.io/about.md) | `src/pages/about.astro` |
| [research.md](file://wsl.localhost/Ubuntu/home/westh/portfolio/programming/brycewestheimer.github.io/research.md) | `src/pages/research.astro` |
| [publications.md](file://wsl.localhost/Ubuntu/home/westh/portfolio/programming/brycewestheimer.github.io/publications.md) | `src/pages/publications.astro` |
| [projects.md](file://wsl.localhost/Ubuntu/home/westh/portfolio/programming/brycewestheimer.github.io/projects.md) | `src/pages/projects.astro` |
| [contact.md](file://wsl.localhost/Ubuntu/home/westh/portfolio/programming/brycewestheimer.github.io/contact.md) | `src/pages/contact.astro` |

---

## Implementation Order

1. Initialize Astro project with Tailwind CSS
2. Create design system (colors, typography)
3. Build base layout + navigation
4. Build home page with hero
5. Create About page with timeline
6. Build Projects page with cards
7. Create Publications page
8. Migrate blog posts
9. Migrate tutorials
10. Add CV page
11. Contact page
12. Final polish and deploy

---

## Verification Plan

### Local Development

```bash
npm run dev
```
Opens at `http://localhost:4321`

### Testing Checklist

- [ ] All 8 nav links work
- [ ] Responsive at mobile/tablet/desktop
- [ ] Dark mode toggle functions
- [ ] Blog/tutorial posts render correctly
- [ ] CV download works
- [ ] Contact form submits

### Deployment

```bash
npm run build
# Commit and push to main
# GitHub Pages auto-deploys
```

---

## Manual Steps Required

> [!IMPORTANT]
> **Before implementation begins:**
> 1. Download your profile photo from LinkedIn and save as `profile.jpg`
> 2. Export your current CV as PDF named `cv.pdf`
> 3. Place both files in a location I can access (e.g., the current repo's `assets/` folder)
