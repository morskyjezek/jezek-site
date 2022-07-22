This is a post in my occasional 
[humanities data curation series](/tags/#humanities-data),
which outlines humanities data curation activities
using publicly-available data on grants and awards made
by the [National Endowment for the Humanities (NEH)](http://www.neh.gov). A [subset of the series](/categories/#mapping-humanities) focuses on mapping the data and creating geospatial data visualizations.

For reference, here are the other esssays in the series:

{% assign hums_data_posts = site.posts | where: "tags", "humanities data" %}
{% for post in hums_data_posts %}
* [{{ post.title }}]({{ post.url }}) ({{ post.date | date: "%e %B %Y" | lstrip }})
{% endfor %}