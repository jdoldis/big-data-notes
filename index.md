---
title: Big Data?
---

Modern data platforms are made up of a lot of different frameworks. With these frameworks comes a lot of buzzwords and context that I found confusing starting out as a graduate data engineer.

In an effort to understand them, I've been playing with some of the core big data technologies on a Raspberry Pi cluster and have made notes:

{% assign sorted_topics = site.topics | sort: "sequence"  %}
{% for topic in sorted_topics %}- [{{ topic.title }}]({{ site.baseurl }}{{ topic.url }})
{% endfor %}

See [this page]({{ site.baseurl }}{% link pi.md %}) for the gear used to build the pi cluster.
