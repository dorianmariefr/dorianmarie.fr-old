---
title: Dorian Marié
display-title: false
---

<div style="display: flex"><div style="margin-right: 40px;">

<h2>Dorian Marié</h2>

<ul><li><i class="fas fa-envelope fa-fw text-red-600"></i> <a href="mailto:dorian@dorianmarie.fr">dorian@dorianmarie.fr</a></li>
<li><i class="fas fa-phone fa-fw text-green-600"></i> <a href="tel:+33767239573">+33 7 67 23 95 73</a></li>
<li><i class="fas fa-map-marker fa-fw text-yellow-600"></i> Paris, France</li>
<li><i class="fab fa-twitter fa-fw text-twitter"></i> <a href="https://twitter.com/dorianmariefr">@dorianmariefr</a></li>
<li><i class="fab fa-github fa-fw text-github"></i> <a href="https://github.com/dorianmariefr">@dorianmariefr</a></li>
<li><i class="fab fa-facebook fa-fw text-facebook"></i> <a href="https://facebook.com/dorianmariefr">Dorian Marié</a></li>
<li><i class="fab fa-reddit fa-fw text-reddit"></i> <a href="https://reddit.com/u/dorianmariefr">/u/dorianmariefr</a></li>
<li><i class="fas fa-couch fa-fw text-couchsurfing"></i> <a href="https://www.couchsurfing.com/users/2012917976">Dorian Marié</a></li>
<li><i class="fab fa-y-combinator fa-fw text-y-combinator"></i> <a href="https://news.ycombinator.com/user?id=dorianmariefr">dorianmariefr</a></li>
<li><i class="fas fa-square fa-fw text-lobsters"></i> <a href="https://lobste.rs/u/dorianmariefr">/u/dorianmariefr</a></li>
<li><i class="fab fa-linkedin fa-fw text-linkedin"></i> <a href="https://www.linkedin.com/in/dorian-marié-948b001a2">Dorian Marié</a></li></ul>

</div><div>

<h2>Pages</h2>

<ul>
{% for page in site.pages %}
{% if page.title and page.url != "/" %}
<li>
{% if page.locale == "en" %}🇬🇧{% endif %}
{% if page.locale == "fr" %}🇫🇷{% endif %}
<a href="{{ page.url }}">
{{ page.title }}
</a>
</li>
{% endif %}
{% endfor %}
</ul>

</div></div>
