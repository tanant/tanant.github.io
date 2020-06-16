---
layout: default
title: Diagnostics
permalink: /diagnostics/

---
Wait, what? A diagnostics page? For a static site generator? 

Yup! This is just me mucking around a little as I customise the theme and get to understanding how it works based on poking around, one of the major things that I've started with is learning how to play with liquid, and site structure. One of the things that came to mind is looking at [static files][jekyll-staticfile-doc].

So here's a list of all the static files that are here on the site (sorted by datemod):

<div>
  {%- if site.static_files.size > 0 -%}
  {%- assign datemod_sorted = site.static_files | sort: "modified_time" | reverse -%}
  <ul>
      {%- for static_file in datemod_sorted -%}
      <li>
        {%- assign date_format = site.minima.date_format | default: "%Y-%b-%d, %k:%M:%S.%L" -%}
        {{ static_file.modified_time | date: date_format }} : <a href = "{{ static_file.path }}">{{ static_file.name }}</a>
      </li>
      {%- endfor -%}
  </ul>
  {%- else -%}
    <ul><li>No static files (really? like, really? This is so unlikely you should probably check this.)</li></ul>
  {%- endif -%}
</div>


[jekyll-staticfile-doc]: https://jekyllrb.com/docs/static-files/