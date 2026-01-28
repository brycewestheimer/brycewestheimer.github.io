---
layout: page
title: "Contact"
description: "Get in touch to discuss research, collaborations, or opportunities"
permalink: /contact/
---

## Let’s Connect

I welcome conversations about fragment-based quantum chemistry, high-performance computing, and the software ecosystems that make those methods usable. If you’re exploring multi-layer adaptive partitioning, modernizing GAMESS-based workflows, or planning a research talk, I’d be glad to hear from you.

<div class="contact-grid">
  <div class="contact-methods">

    ### Primary Channels

    <div class="contact-item">
      <h4>📧 Email</h4>
      <p><a href="mailto:{{ site.author.email }}">{{ site.author.email }}</a></p>
      <small>Direct line for research collaborations, invitations, and media inquiries.</small>
    </div>

    <div class="contact-item">
      <h4>💼 LinkedIn</h4>
      <p><a href="https://linkedin.com/in/{{ site.social_media.linkedin }}" target="_blank">Connect on LinkedIn</a></p>
      <small>Best for networking and ongoing project updates.</small>
    </div>

    <div class="contact-item">
      <h4>🐙 GitHub</h4>
      <p><a href="https://github.com/{{ site.social_media.github }}" target="_blank">@{{ site.social_media.github }}</a></p>
      <small>Open-source discussions for `libfrag`, `public_libaccefp`, and related tooling.</small>
    </div>

    <div class="contact-item">
      <h4>🎓 Google Scholar</h4>
      <p><a href="https://scholar.google.com/citations?user={{ site.social_media.google_scholar }}" target="_blank">Scholar Profile</a></p>
      <small>Publication record and citation metrics.</small>
    </div>

    ### Location & Response

    <div class="contact-item">
      <h4>📍 Location</h4>
      <p>{{ site.author.location }}</p>
      <small>Collaborating with the Guidez & Lin groups at CU Denver; remote-friendly worldwide.</small>
    </div>

    <div class="contact-item">
      <h4>⏱️ Response Expectations</h4>
      <p>Replies within 2–3 business days</p>
      <small>Time-sensitive requests: mention your deadline in the subject line.</small>
    </div>

  </div>

  <div class="contact-form-section">

    ### Send a Message

    <form class="contact-form" action="https://formspree.io/f/xpzvnqwr" method="POST">
      <div class="form-group">
        <label for="name">Name *</label>
        <input type="text" id="name" name="name" required>
      </div>

      <div class="form-group">
        <label for="email">Email *</label>
        <input type="email" id="email" name="email" required>
      </div>

      <div class="form-group">
        <label for="affiliation">Affiliation</label>
        <input type="text" id="affiliation" name="affiliation" placeholder="University, lab, or company">
      </div>

      <div class="form-group">
        <label for="topic">Topic *</label>
        <select id="topic" name="topic" required>
          <option value="">Select one…</option>
          <option value="map-collaboration">MAP / fragment-based collaboration</option>
          <option value="hpc-modernization">HPC or software modernization</option>
          <option value="speaking-invitation">Speaking invitation</option>
          <option value="media-interview">Media or outreach request</option>
          <option value="student-question">Student or mentoring question</option>
          <option value="other">Other</option>
        </select>
      </div>

      <div class="form-group">
        <label for="message">Message *</label>
        <textarea id="message" name="message" rows="6" required placeholder="Please include project context, desired outcomes, and any timeline details."></textarea>
      </div>

      <div class="form-group">
        <button type="submit" class="btn btn-primary">Send Message</button>
      </div>

      <input type="hidden" name="_subject" value="Personal site contact">
      <input type="hidden" name="_next" value="{{ site.url }}{{ site.baseurl }}/contact/thank-you/">
    </form>

  </div>
</div>

## Collaboration Focus

- **Fragment-based & MAP methods** – Joint method development, benchmarking, or shared software infrastructure.
- **High-performance computing** – Porting or optimizing GAMESS-derived workflows for accelerator-rich systems.
- **Open-source science** – Contributions to `libfrag`, `public_libaccefp`, `public_libaccsapt`, or related tooling.
- **Speaking engagements** – Conference talks and panels on fragment-based theory, HPC modernization, and software sustainability (e.g., ACS Fall 2024 PHYS Division).

## Frequently Asked Questions

<details>
<summary><strong>How quickly can I expect a reply?</strong></summary>
<p>I review new messages twice a week. If your request is time-sensitive, highlight the deadline up front and I’ll do my best to accommodate.</p>
</details>

<details>
<summary><strong>Do you offer mentoring for students or early-career researchers?</strong></summary>
<p>Yes—especially for topics related to GAMESS development, MAP workflows, or HPC enablement. Share your goals and I’ll suggest next steps or resources.</p>
</details>

<details>
<summary><strong>Can we collaborate on software outside of GAMESS?</strong></summary>
<p>Absolutely. I’m particularly interested in open-source projects that advance fragment-based methods or improve accelerator portability across chemistry codes.</p>
</details>

<details>
<summary><strong>Are you available for interviews or speaking invitations?</strong></summary>
<p>Yes. Please include the event name, audience, preferred topic, and timeline so I can confirm availability.</p>
</details>

---

*Looking for specifics not covered here? Drop me a note—the best collaborations often start with a quick message.*
