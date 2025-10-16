# Personal Site Overhaul TODO

This document tracks the comprehensive revision plan for the personal GitHub Pages site. Progress is organized by milestones and individual deliverables so that each section can be verified against authoritative sources (LinkedIn, Google Scholar, GitHub, CV).

## Milestones

- [x] **M1 – Data Collection & Verification**  
  Gather accurate biographical details, positions, education history, research focus areas, publications, projects, awards, and social handles from primary sources. Document citations or notes for each item.
- [x] **M2 – Content Rewrite**  
  Replace placeholder copy on all public-facing pages (`about`, `contact`, `research`, `projects`, `publications`) with verified information and cohesive narrative.
- [x] **M3 – Metadata Alignment**  
  Ensure site-wide metadata (config, data files, includes) match the updated content and remove outdated references.
- [ ] **M4 – Review & Publish**  
  Proofread, run site locally, resolve build warnings, and deploy changes.

## Data Collection Checklist (supports M1)

- [x] Export current résumé/CV for quick reference.  
- [x] Capture current title, employer, and location.  
- [x] Compile full education timeline (degrees, institutions, years, thesis topics).  
- [x] List current and prior research areas with short descriptions.  
- [x] Extract most recent publications with accurate citations (journal, year, DOI).  
- [x] Gather highlights from invited talks, awards, grants, and professional service.  
- [x] Inventory active software projects/repos with status and tech stack.  
- [x] Confirm preferred contact channels (email, LinkedIn, other).  
- [x] Verify social handles and usernames for GitHub, LinkedIn, Google Scholar, Twitter/X (if applicable).

## Page Rewrites (supports M2)

### `about.md`
- [x] Rewrite professional summary grounded in current role.  
- [x] Update employment history with bullet list of key accomplishments.  
- [x] Add accurate education section with degrees and dates.  
- [x] Refresh technical expertise with technologies actually used.  
- [x] Include personal interests or outreach that reflect reality.  
- [x] Remove placeholders for nonexistent software or accolades.

### `contact.md`
- [x] Confirm email address and update call-to-action copy.  
- [x] Adjust LinkedIn/GitHub links to verified profiles.  
- [x] Remove or edit services list to match genuine offerings.  
- [x] Update response time, location, and availability text.  
- [x] Validate Formspree endpoint or replace with current workflow.  
- [x] Tweak FAQ entries to reflect current policies.

### `research.md`
- [x] Summarize active research themes accurately.  
- [x] Highlight current grants, collaborations, or lab affiliations.  
- [x] Replace fictional software names with real tools or remove.  
- [x] Update “Recent Highlights” to real ongoing projects.  
- [x] Ensure philosophy and goals match actual trajectory.  
- [x] Provide links to selected publications or project pages.

### `projects.md`
- [x] Curate 3–6 flagship projects with authentic descriptions.  
- [x] For each project, note status, role, tech stack, and outcomes.  
- [x] Link to GitHub repositories, documentation, or demo pages.  
- [x] Remove auto-fetched repo list if not maintained, or ensure styling/data presentation works.  
- [x] Update collaboration/community sections with real affiliations.  
- [x] Add planned initiatives only if actively in development.

### `publications.md`
- [x] Build list of peer-reviewed papers in reverse chronological order.  
- [x] Include accurate co-authors, journal, volume, pages, and DOIs.  
- [x] Separate preprints, invited talks, posters if useful.  
- [x] Update metrics (citations, h-index) from Google Scholar or omit if not maintained.  
- [x] Review awards/recognition section for accuracy.  
- [x] Link to downloadable CV if available and current.

## Metadata & Supporting Files (supports M3)

- [x] Update `_config.yml` author section (name, email, bio, avatar).  
- [x] Revise `_data/social_media.yml` with current handles and remove unused platforms.  
- [x] Audit `_includes` snippets (`interests.html`, `masthead.html`, `projects.html`, etc.) for outdated copy.  
- [x] Confirm `_data/colors.json` or other assets reflect branding choices (optional).  
- [x] Check navigation links for accuracy and order.  
- [x] Ensure Open Graph / SEO metadata reflects new descriptions.  

## Review & Deployment (supports M4)

- [ ] Run `bundle exec jekyll serve` locally; verify all pages render correctly.  
- [x] Install Ruby, MSYS2 toolchain, and Bundler dependencies for local builds.  
- [x] Run `bundle exec jekyll build` locally; confirm site generates without errors.  
- [x] Proofread for typos, tense consistency, and tone.  
- [x] Validate external links (LinkedIn, Google Scholar, GitHub, contact form).  
- [ ] Commit with descriptive message summarizing major content updates.  
- [ ] Push to GitHub and confirm GitHub Pages rebuild succeeds.  
- [ ] Spot-check site on mobile and desktop; capture screenshots if desired.

## Change Log

| Date | Change | Notes |
| --- | --- | --- |
| 2025-10-16 | Synced metadata files | Updated `_config.yml` author/topics/skills/projects and trimmed `_data/social_media.yml` to verified platforms. |
| 2025-10-16 | Refined homepage includes | Updated `masthead`, `interests`, and `projects` partials to pull from verified metadata and remove placeholder content. |
| 2025-10-16 | Aligned navigation & branding | Switched mobile nav social icons to central data source and confirmed navigation ordering matches live pages. |
| 2025-10-16 | Updated SEO metadata | Refreshed `header.html` OG/Twitter tags and JSON-LD to use verified descriptions, images, and social links. |
| 2025-10-16 | Hardened project listings | Removed unauthenticated GitHub API dependency, updated `projects.html`, and reran `bundle exec jekyll build` successfully. |
| 2025-10-16 | Proofread & link validation | Reviewed core content pages for tone/typos and verified LinkedIn, GitHub, Scholar, and contact form links. |
| 2025-10-15 | Initial TODO drafted | Added milestone breakdown and detailed checklists for each content area. |
| 2025-10-15 | Updated data collection status | Checked off project inventory and contact/social verification after adding sources to `notes/milestone1-sources.md`. |
| 2025-10-17 | Logged publication citations | Added Crossref-backed publication list to notes and marked checklist item complete. |
| 2025-10-16 | Completed M1 biographical data pass | Added CV snapshot, employment/education timeline, and presentation records to `notes/milestone1-sources.md`; checked off remaining data collection tasks. |
| 2025-10-15 | Local build environment configured | Installed Ruby/MSYS2 toolchain, resolved tzinfo dependency, and verified `bundle exec jekyll build` succeeds. |
| 2025-10-16 | Completed primary page rewrites | Replaced placeholder content in `about`, `research`, `contact`, `projects`, and `publications`; marked M2 checklist items complete. |

---

As each task is completed, update the corresponding checklist items and milestone status above. Add new rows to the change log with brief summaries to maintain a history of edits.
