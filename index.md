---
title: My Notes
---

<!-- Dark mode toggle button -->
<button id="darkModeToggle" style="float:right; margin:10px 30px 0 0; font-size:1em;">🌙 Dark Mode</button>

<!-- Navigation bar -->
{% include header-custom.html %}

# My Notes

Welcome! Here are all my notes:

<ul>
{% for post in site.posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

<!-- Dark mode toggle script -->
<script>
  document.addEventListener('DOMContentLoaded', function () {
    var btn = document.getElementById('darkModeToggle');
    if (btn) {
      btn.onclick = function () {
        document.body.classList.toggle('dark-mode');
        localStorage.setItem('darkMode', document.body.classList.contains('dark-mode') ? 'on' : '');
      };
      if(localStorage.getItem('darkMode') === 'on') {
        document.body.classList.add('dark-mode');
      }
    }
  });
</script>
