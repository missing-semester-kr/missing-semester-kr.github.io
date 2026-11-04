---
layout: page
title: "2019년 강의"
description: >
  MIT IAP 2019 Missing Semester의 강의 노트와 영상입니다.
permalink: /2019/
phony: true
translation_status: draft
---

{% assign lecture_date = site['2019'] | group_by: 'date' | sort: 'name' %}
{% for date in lecture_date %}
  {% if date.name == "" %}{% continue %}{% endif %}
  <h2>{{ date.name | date: '%A, %-m/%-d' }}</h2>
  {% assign lectures = date.items | sort: 'order' %}
  <ul>
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <a href="{{ lecture.url }}">{{ lecture.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
  </ul>
{% endfor %}

# MIT 밖으로

다른 사람들도 이 자료에서 도움을 얻을 수 있기를 바라며 MIT 밖에도 이 수업을 공유했습니다. 다음에서 관련 게시물과 토론을 찾아볼 수 있습니다.

- [Hacker News](https://news.ycombinator.com/item?id=19078281)
- [Lobsters](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on)
- [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/)
- [r/programming](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/)
- [Twitter](https://twitter.com/Jonhoo/status/1091896192332693504)
- [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv)

# 감사의 말

이 수업은 [SIPB IAP 2019](https://sipb.mit.edu/iap/2019/)의 일부로 진행되었으며, [SIPB](https://sipb.mit.edu/)와 [MIT EECS](https://www.eecs.mit.edu/)가 공동 후원했습니다.
