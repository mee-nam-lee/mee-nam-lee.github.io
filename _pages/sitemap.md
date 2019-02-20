---
layout: archive
title: "사이트맵"
permalink: /sitemap/
author_profile: false
---

<h2>Pages</h2>
  <ul>
{%- for post in site.pages -%}
  <li>{%- include archive-single.html -%}</li>
{%- endfor -%}
  </ul>

<h2>Articles</h2>
  <ul>
{%- for post in site.posts -%}
  <li>{%- include archive-single.html -%}</li>
{%- endfor -%}
  </ul>

{%- capture written_label -%}'None'{%- endcapture -%}

{%- for collection in site.collections -%}
{%- unless collection.output == false or collection.label == "posts" -%}
  {%- capture label -%}{{ collection.label }}{%- endcapture -%}
  {%- if label != written_label -%}
  <h2>{{ label }}</h2>
  {%- capture written_label -%}{{ label }}{%- endcapture -%}
  {%- endif -%}
{%- endunless -%}
{%- for post in collection.docs -%}
  {%- unless collection.output == false or collection.label == "posts" -%}
  {%- include archive-single.html -%}
  {%- endunless -%}
{%- endfor -%}
{%- endfor -%}