---
layout: archive
title: "Archive"
permalink: /categories/
author_profile: false
---
<h2 id="page-title" class="page__title">Tags</h2>

  {%- include collecttags.html -%}

{%- assign categories_max = 0 -%}
{%- for category in site.categories -%}
  {%- if category[1].size > categories_max -%}
    {%- assign categories_max = category[1].size -%}
  {%- endif -%}
{%- endfor -%}

<h2 id="page-title" class="page__title">카테고리</h2>

<ul>
  {%- for i in (1..categories_max) reversed -%}
    {%- for category in site.categories -%}
      {%- if category[1].size == i -%}
        <li>
          <a href="#{{ category[0] | slugify }}">
            <span style="font-size=1.5rem"> <strong>{{ category[0] }}</strong> ( {{ i }} )</span>
          </a>
        </li>
      {%- endif -%}
    {%- endfor -%}
  {%- endfor -%}
</ul>

{%- for i in (1..categories_max) reversed -%}
  {%- for category in site.categories -%}
    {%- if category[1].size == i -%}
      <section id="{{ category[0] | slugify | downcase }}">
        <h3>{{ category[0] }}</h3>
        <div class="entries-{{ page.entries_layout | default: 'list' }}">
          <ul>
          {%- for post in category.last -%}
            <li>{%- include archive-single.html type=page.entries_layout -%}</li>
          {%- endfor -%}
          </ul>
        </div>
        <div class="pagination">
        <a href="#page-title">{{ site.data.ui-text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
        </div>
        <br/>
      </section>
    {%- endif -%}
  {%- endfor -%}
{%- endfor -%}
