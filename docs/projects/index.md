---
title: Projects
permalink: /projects/
---
<p class="eyebrow">projects</p>
<h1>Projects</h1>
<p class="hero__lede">SQL, pipelines, and platforms — each one links to a full write-up with queries and reasoning included.</p>

<div class="project-grid">
  {% for project in site.projects %}
  <a href="{{ project.url | relative_url }}" class="project-card">
    <h3>{{ project.title }}</h3>
    <p>{{ project.tagline }}</p>
    {% if project.flow %}
    <div class="flow-marker">
      {% assign steps = project.flow | split: "->" %}
      {% for step in steps %}
        <span class="flow-step">{{ step | strip }}</span>
        {% unless forloop.last %}<span class="flow-arrow">&rarr;</span>{% endunless %}
      {% endfor %}
    </div>
    {% endif %}
  </a>
  {% endfor %}
</div>
