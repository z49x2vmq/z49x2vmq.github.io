{% for cat in site.categories %}
  <h2> {{cat[0]}} </h2>
  {% for post in cat[1] %} - [{{post.title}}]({{post.url}}) 
  {% endfor %}
{% endfor %}
