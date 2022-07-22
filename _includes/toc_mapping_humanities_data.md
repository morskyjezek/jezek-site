This post continues with my occasional 
[humanities data
series](/tags/#humanities-data),
which outlines a humanities data curation project
using publicly-available data on grants and awards made
by the [National Endowment for the Humanities (NEH)](http://www.neh.gov).

For reference, here are the other esssays in the series:

{% assign hums_data_posts = site.posts | where: "tags", "humanities data" %}
{% for post in hums_data_posts %}
* [{{ post.title }}]({{ post.url }}) ({{ post.date | date: "%e %B %Y" | lstrip }})
{% endfor %}