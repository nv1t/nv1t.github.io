---
layout: page
title: Eleventeen
tagline: "Coz' imaginary numbers can be real"
---
{% include JB/setup %}

{% for post in site.posts %}
<div class="container">
    <article>
        <h2 class="title"><a href="{{ post.url }}">{{ post.title }}</a></h2>
        <time>Posted on: {{ post.date | date_to_string }}.</time>
        {{ post.content }}
        <div class="blog-feedback">
            <h2 class="blog-feedback-header with-twitter">
                Have feedback on this post? Let <a href="https://twitter.com/intent/tweet?text=@{{ site.twitter }}&url=http://{{ site.url }}{{ post.url }}" target="blank">@{{ site.twitter }}</a> know on Twitter.
            </h2>
            <p class="blog-feedback-description">
                Want to talk about this over email? <a href="mailto:{{ site.email }}">Contact me</a>.
            </p>
        </div>
    </article>
    <div class="links">Posted in <a href="#">{{ post.category }}</a>. <a href="{{ post.url }}">Permalink</a> to post.</div>
</div>
{% endfor %}

