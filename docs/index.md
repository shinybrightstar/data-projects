---
title: Home
---
<section class="hero">
  <p class="eyebrow">data engineering portfolio</p>
  <h1>I turn messy data into pipelines people can trust.</h1>
  <p class="hero__role">Support Specialist &rarr; Data Engineer in progress</p>
  <p class="hero__lede">
    I'm Judith. My background is customer support and community management, my degrees are in chemical
    engineering and business — and I'm now building the SQL, pipeline, and warehouse skills to work as a
    data engineer, fully remote. Everything below is real work: real queries, real datasets, real repos.
  </p>
  <div class="hero__cta">
    <a href="{{ '/projects/' | relative_url }}" class="btn">View projects</a>
    <a href="{{ '/about/' | relative_url }}" class="btn btn--ghost">About me</a>
  </div>
</section>

<section>
  <div class="section-head">
    <h2>Featured projects</h2>
    <a href="{{ '/projects/' | relative_url }}" class="view-all">all projects &rarr;</a>
  </div>
  <div class="project-grid">
    {% for project in site.projects limit:3 %}
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
</section>
