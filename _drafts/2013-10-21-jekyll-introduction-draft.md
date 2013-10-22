---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}


This is an example of a draft. Read more here: [http://jekyllrb.com/docs/drafts/](http://jekyllrb.com/docs/drafts/)

add images like so...
	
    <img src="/assets/images/{{ post.category }}.png" alt="{{ post.category | uppercase }}" title="Post language: {{ post.category }}" width="20" height="10"/>
    

upload to assets/images folder