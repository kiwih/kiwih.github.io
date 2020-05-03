<center><i>01001000, or "zero,one-hundred,one-thousand", is 'H' in ASCII</i></center>

# Welcome!

My name is Hammond and my github username is kiwih, and I like to tinker and build things!

A friend told me I should write about some of it, so here goes:

## Projects

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

