---
# the default layout is 'page'
icon: fas fa-cloud
order: 5
excerpt_separator: <!--continue for more proxmox-->
---
# Proxmox

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
