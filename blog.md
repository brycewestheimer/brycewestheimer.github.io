---
layout: blog
title: "Blog"
description: "Thoughts, insights, and updates on computational chemistry and scientific computing"
permalink: /blog/
---

## Latest Posts

<div class="blog-notice">
  <h3>Blog Under Construction</h3>
  <p>This section is being rebuilt with new content. Check back soon for updates on computational chemistry, method development, and scientific computing.</p>
</div>

<div class="blog-filters">
  <button class="filter-btn active" data-filter="all">All Posts</button>
  <button class="filter-btn" data-filter="ml-ai">ML & AI</button>
  <button class="filter-btn" data-filter="computational-chemistry">Computational Chemistry</button>
  <button class="filter-btn" data-filter="general">General</button>
</div>

<div class="post-grid">
  {% for post in site.posts %}
    <article class="post-card" data-category="{% if post.categories.size > 0 %}{{ post.categories[0] }}{% endif %}">
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
  </div>
{% endif %}

<script>
// Blog filtering functionality
(function() {
  const filterBtns = document.querySelectorAll('.filter-btn');
  const postCards = document.querySelectorAll('.post-card');
  
  filterBtns.forEach(btn => {
    btn.addEventListener('click', () => {
      const filter = btn.dataset.filter;
      
      filterBtns.forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      
      postCards.forEach(card => {
        const cardCategory = card.dataset.category || 'none';
        if (filter === 'all' || cardCategory === filter) {
          card.style.display = 'block';
        } else {
          card.style.display = 'none';
        }
      });
    });
  });
})();
</script>

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
