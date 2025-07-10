---
layout: blog
title: "Blog"
description: "Thoughts, insights, and updates on computational chemistry and scientific computing"
permalink: /blog/
---

## Latest Posts

<div class="post-grid">
  {% for post in site.posts %}
    <article class="post-card">
      <header class="post-header">
        <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
        <div class="post-meta">
          <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time>
          <span class="post-author">by {{ post.author | default: site.author.name }}</span>
          {% if post.read_time %}
            <span class="read-time">{{ post.read_time }} min read</span>
          {% endif %}
        </div>
      </header>
      
      {% if post.excerpt %}
        <div class="post-excerpt">
          {{ post.excerpt | strip_html | truncatewords: 50 }}
        </div>
      {% endif %}
      
      <footer class="post-footer">
        {% if post.categories %}
          <div class="post-categories">
            {% for category in post.categories %}
              <span class="category">{{ category }}</span>
            {% endfor %}
          </div>
        {% endif %}
        
        {% if post.tags %}
          <div class="post-tags">
            {% for tag in post.tags %}
              <span class="tag">{{ tag }}</span>
            {% endfor %}
          </div>
        {% endif %}
        
        <a href="{{ post.url | relative_url }}" class="read-more">Read More →</a>
      </footer>
    </article>
  {% endfor %}
</div>

{% if site.posts.size == 0 %}
  <div class="no-posts">
    <h3>Coming Soon!</h3>
    <p>I'm working on some exciting blog posts about computational chemistry, method development, and scientific computing. Check back soon for updates!</p>
    
    <div class="planned-topics">
      <h4>Planned Topics:</h4>
      <ul>
        <li>Getting Started with GPU-Accelerated Quantum Chemistry</li>
        <li>Machine Learning in Computational Chemistry: A Practical Guide</li>
        <li>Optimizing Python Code for Scientific Computing</li>
        <li>Best Practices for High-Performance Computing in Chemistry</li>
        <li>Building Scalable Quantum Chemistry Workflows</li>
      </ul>
    </div>
    
    <p>Want to be notified when new posts are published? <a href="/contact/">Get in touch</a> and I'll add you to my mailing list!</p>
  </div>
{% endif %}

## What You'll Find Here

### Research Insights
Deep dives into quantum chemistry methods, computational strategies, and research findings from my work in the field.

### Technical Tutorials
Step-by-step guides for implementing computational chemistry methods, optimizing code performance, and using specialized software.

### Industry Perspectives
Thoughts on the intersection of academic research and industry applications, technology transfer, and practical implementations.

### Software Development
Best practices for scientific software development, code optimization, and building maintainable research tools.

### Career & Education
Advice for students and early-career researchers in computational chemistry and scientific computing.

## Subscribe for Updates

Stay updated with my latest posts and research developments:

<div class="subscribe-section">
  <form class="subscribe-form" action="https://formspree.io/f/xpzvnqwr" method="POST">
    <input type="email" name="email" placeholder="Enter your email" required>
    <button type="submit" class="btn btn-primary">Subscribe</button>
    <input type="hidden" name="_subject" value="Blog Subscription">
    <input type="hidden" name="subscription" value="blog">
  </form>
  <p><small>No spam, just quality content. Unsubscribe anytime.</small></p>
</div>

---

*Have a topic you'd like me to write about? [Send me a suggestion](/contact/) and I'll consider it for a future post!*
