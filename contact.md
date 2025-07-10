---
layout: page
title: "Contact"
description: "Get in touch to discuss research, collaborations, or opportunities"
permalink: /contact/
---

## Let's Connect

I'm always interested in discussing computational chemistry, research collaborations, and new opportunities. Whether you're a fellow researcher, industry professional, or student, I'd love to hear from you.

<div class="contact-grid">
  <div class="contact-methods">
    
    ### Quick Contact
    
    <div class="contact-item">
      <h4>📧 Email</h4>
      <p><a href="mailto:{{ site.author.email }}">{{ site.author.email }}</a></p>
      <small>Best for detailed discussions and formal inquiries</small>
    </div>
    
    <div class="contact-item">
      <h4>💼 LinkedIn</h4>
      <p><a href="https://linkedin.com/in/{{ site.social_media.linkedin }}" target="_blank">Connect on LinkedIn</a></p>
      <small>Professional networking and industry connections</small>
    </div>
    
    <div class="contact-item">
      <h4>🐙 GitHub</h4>
      <p><a href="https://github.com/{{ site.social_media.github }}" target="_blank">Follow on GitHub</a></p>
      <small>Code collaborations and open source projects</small>
    </div>
    
    <div class="contact-item">
      <h4>🎓 Google Scholar</h4>
      <p><a href="https://scholar.google.com/citations?user={{ site.social_media.google_scholar }}" target="_blank">View Publications</a></p>
      <small>Research publications and academic collaborations</small>
    </div>
    
    ### Location & Availability
    
    <div class="contact-item">
      <h4>📍 Location</h4>
      <p>{{ site.author.location }}</p>
      <small>Open to remote collaborations worldwide</small>
    </div>
    
    <div class="contact-item">
      <h4>⏰ Response Time</h4>
      <p>Usually within 24-48 hours</p>
      <small>Faster response for urgent matters</small>
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
        <label for="organization">Organization</label>
        <input type="text" id="organization" name="organization">
      </div>
      
      <div class="form-group">
        <label for="subject">Subject *</label>
        <select id="subject" name="subject" required>
          <option value="">Please select...</option>
          <option value="research-collaboration">Research Collaboration</option>
          <option value="industry-partnership">Industry Partnership</option>
          <option value="speaking-opportunity">Speaking Opportunity</option>
          <option value="consulting-inquiry">Consulting Inquiry</option>
          <option value="student-inquiry">Student/Academic Inquiry</option>
          <option value="software-support">Software Support</option>
          <option value="other">Other</option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="message">Message *</label>
        <textarea id="message" name="message" rows="6" required placeholder="Please provide details about your inquiry, including any relevant background information and what you'd like to discuss."></textarea>
      </div>
      
      <div class="form-group">
        <button type="submit" class="btn btn-primary">Send Message</button>
      </div>
      
      <input type="hidden" name="_subject" value="Website Contact Form">
      <input type="hidden" name="_next" value="{{ site.url }}{{ site.baseurl }}/contact/thank-you/">
    </form>
    
  </div>
</div>

## What I'm Looking For

### Research Collaborations
- Joint research projects in computational chemistry
- Method development partnerships
- High-performance computing applications
- Machine learning in chemistry

### Industry Partnerships
- Consulting opportunities in quantum chemistry
- Software development projects
- Training and workshop delivery
- Technology transfer initiatives

### Speaking Opportunities
- Conference presentations
- Workshop instruction
- Seminar talks
- Podcast appearances

### Academic Engagement
- Postdoc and student mentoring
- Visiting researcher opportunities
- Grant collaboration
- Peer review and editorial activities

## Frequently Asked Questions

<details>
<summary><strong>What's the best way to reach you for urgent matters?</strong></summary>
<p>Email is still the best method for urgent matters. Please mark your email subject with "URGENT" and I'll prioritize my response.</p>
</details>

<details>
<summary><strong>Do you provide consulting services?</strong></summary>
<p>Yes, I offer consulting services for computational chemistry projects, software development, and high-performance computing applications. Please contact me with details about your project.</p>
</details>

<details>
<summary><strong>Are you available for speaking engagements?</strong></summary>
<p>I regularly accept speaking invitations for conferences, workshops, and seminars. Please provide details about your event, audience, and preferred topics.</p>
</details>

<details>
<summary><strong>Do you offer training or workshops?</strong></summary>
<p>Yes, I provide training in computational chemistry methods, GPU programming, and scientific software development. Contact me to discuss your training needs.</p>
</details>

<details>
<summary><strong>Can you help with my research project?</strong></summary>
<p>I'm always interested in discussing potential collaborations. Please provide details about your project, timeline, and how you think I might contribute.</p>
</details>

## Professional Services

### Consulting Areas
- Quantum chemistry method development
- High-performance computing optimization
- GPU acceleration strategies
- Scientific software development
- Machine learning applications in chemistry

### Training & Workshops
- Introduction to quantum chemistry programming
- GPU programming for scientific applications
- High-performance computing for chemists
- Machine learning in computational chemistry
- Software development best practices

### Speaking Topics
- Modern quantum chemistry methods
- GPU acceleration in scientific computing
- Machine learning applications in chemistry
- Open source software in research
- Industry applications of computational chemistry

---

*I look forward to hearing from you and discussing how we might work together to advance computational chemistry research and applications.*
