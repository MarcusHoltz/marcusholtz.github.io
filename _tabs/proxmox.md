---
# the default layout is 'page'
icon: fas fa-cloud
order: 5
excerpt_separator: <!--continue for more proxmox-->
---
# Proxmox

<ul>
  {% for post in site.posts %}
    {% for proxmox in site.tags %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
{% endfor %}
</ul>

