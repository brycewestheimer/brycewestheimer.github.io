# Personal Site Overhaul TODO

This document tracks the comprehensive revision plan for the personal GitHub Pages site. Progress is organized by milestones and individual deliverables so that each section can be verified against authoritative sources (LinkedIn, Google Scholar, GitHub, CV).

## Milestones

- [ ] **M1 – Data Collection & Verification**  
  Gather accurate biographical details, positions, education history, research focus areas, publications, projects, awards, and social handles from primary sources. Document citations or notes for each item.
- [ ] **M2 – Content Rewrite**  
  Replace placeholder copy on all public-facing pages (`about`, `contact`, `research`, `projects`, `publications`) with verified information and cohesive narrative.
- [ ] **M3 – Metadata Alignment**  
  Ensure site-wide metadata (config, data files, includes) match the updated content and remove outdated references.
- [ ] **M4 – Review & Publish**  
  Proofread, run site locally, resolve build warnings, and deploy changes.

## Data Collection Checklist (supports M1)

- [ ] Export current résumé/CV for quick reference.  
- [ ] Capture current title, employer, and location.  
- [ ] Compile full education timeline (degrees, institutions, years, thesis topics).  
- [ ] List current and prior research areas with short descriptions.  
- [x] Extract most recent publications with accurate citations (journal, year, DOI).  
- [ ] Gather highlights from invited talks, awards, grants, and professional service.  
- [x] Inventory active software projects/repos with status and tech stack.  
- [x] Confirm preferred contact channels (email, LinkedIn, other).  
- [x] Verify social handles and usernames for GitHub, LinkedIn, Google Scholar, Twitter/X (if applicable).

## Page Rewrites (supports M2)

### `about.md`
- [ ] Rewrite professional summary grounded in current role.  
- [ ] Update employment history with bullet list of key accomplishments.  
- [ ] Add accurate education section with degrees and dates.  
- [ ] Refresh technical expertise with technologies actually used.  
- [ ] Include personal interests or outreach that reflect reality.  
- [ ] Remove placeholders for nonexistent software or accolades.

### `contact.md`
- [ ] Confirm email address and update call-to-action copy.  
- [ ] Adjust LinkedIn/GitHub links to verified profiles.  
- [ ] Remove or edit services list to match genuine offerings.  
- [ ] Update response time, location, and availability text.  
- [ ] Validate Formspree endpoint or replace with current workflow.  
- [ ] Tweak FAQ entries to reflect current policies.

### `research.md`
- [ ] Summarize active research themes accurately.  
- [ ] Highlight current grants, collaborations, or lab affiliations.  
- [ ] Replace fictional software names with real tools or remove.  
- [ ] Update “Recent Highlights” to real ongoing projects.  
- [ ] Ensure philosophy and goals match actual trajectory.  
- [ ] Provide links to selected publications or project pages.

### `projects.md`
- [ ] Curate 3–6 flagship projects with authentic descriptions.  
- [ ] For each project, note status, role, tech stack, and outcomes.  
- [ ] Link to GitHub repositories, documentation, or demo pages.  
- [ ] Remove auto-fetched repo list if not maintained, or ensure styling/data presentation works.  
- [ ] Update collaboration/community sections with real affiliations.  
- [ ] Add planned initiatives only if actively in development.

### `publications.md`
- [ ] Build list of peer-reviewed papers in reverse chronological order.  
- [ ] Include accurate co-authors, journal, volume, pages, and DOIs.  
- [ ] Separate preprints, invited talks, posters if useful.  
- [ ] Update metrics (citations, h-index) from Google Scholar or omit if not maintained.  
- [ ] Review awards/recognition section for accuracy.  
- [ ] Link to downloadable CV if available and current.

## Metadata & Supporting Files (supports M3)

- [ ] Update `_config.yml` author section (name, email, bio, avatar).  
- [ ] Revise `_data/social_media.yml` with current handles and remove unused platforms.  
- [ ] Audit `_includes` snippets (`interests.html`, `masthead.html`, `projects.html`, etc.) for outdated copy.  
- [ ] Confirm `_data/colors.json` or other assets reflect branding choices (optional).  
- [ ] Check navigation links for accuracy and order.  
- [ ] Ensure Open Graph / SEO metadata reflects new descriptions.

## Review & Deployment (supports M4)

- [ ] Run `bundle exec jekyll serve` locally; verify all pages render correctly.  
- [x] Install Ruby, MSYS2 toolchain, and Bundler dependencies for local builds.  
- [x] Run `bundle exec jekyll build` locally; confirm site generates without errors.  
- [ ] Proofread for typos, tense consistency, and tone.  
- [ ] Validate external links (LinkedIn, Google Scholar, GitHub, contact form).  
- [ ] Commit with descriptive message summarizing major content updates.  
- [ ] Push to GitHub and confirm GitHub Pages rebuild succeeds.  
- [ ] Spot-check site on mobile and desktop; capture screenshots if desired.

## Change Log

| Date | Change | Notes |
| --- | --- | --- |
| 2025-10-15 | Initial TODO drafted | Added milestone breakdown and detailed checklists for each content area. |
| 2025-10-15 | Updated data collection status | Checked off project inventory and contact/social verification after adding sources to `notes/milestone1-sources.md`. |
| 2025-10-17 | Logged publication citations | Added Crossref-backed publication list to notes and marked checklist item complete. |
| 2025-10-15 | Local build environment configured | Installed Ruby/MSYS2 toolchain, resolved tzinfo dependency, and verified `bundle exec jekyll build` succeeds. |

---

As each task is completed, update the corresponding checklist items and milestone status above. Add new rows to the change log with brief summaries to maintain a history of edits.
