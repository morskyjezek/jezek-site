---
title: "Jesse Johnston"
layout: splash
permalink: /
as_of_date: 2023-05-10
header:
#  image: /assets/images/bookstacks-diagram-wide.jpg
#  alt: 'A cross-section schematic of the book stacks designed in the 1890s for the Library of Congress by the Snead & Company Ironworks. Image cropped from "Longitudinal section of north stack, Library of Congress, Washington, D.C. (fig. 4)," Library of Congress ([LCCN 2007682525](https://www.loc.gov/item/2007682525/)).'
  image: /assets/images/water-sand-1280w-short.jpg
  overlay_image: /assets/images/water-sand-1280w-short.jpg
  alt: "Image of a shallow, sandy-bottomed bay in Lake Superior with light playing over the rippled surface."
#  image: /assets/images/mackinac-bridge-1280w.jpg
#  overlay_image: /assets/images/mackinac-bridge-1280w.jpg
#  alt: "Photo of the Mackinac Bridge at sunrise. Photo by Aaron Burden on Unsplash."
#  image_caption: "Mackinac Bridge photo by [Aaron Burden](https://unsplash.com/@aaronburden) on Unsplash."
  overlay_filter: 0.2
#excerpt: "Librarian and archivist, teacher, administrator, and scholar with extensive experience in the public sector, academic research, and teaching."
intro:
  - excerpt: "Jesse Johnston is a librarian and archivist, teacher, administrator, and scholar with extensive experience in the public sector, academic research, and teaching."
feature_row:
  - image_path: /assets/images/jesse-teaching-202303-th.jpg
  #old image_path: /assets/images/jesse-classroom-leftfacing-th.jpg
#    alt: "Jesse teaching a Library Carpentry workshop at the Library of Congress for a group of librarians from across the federal government."
    alt: "Jesse Johnston teaching about digital forensics for archivists, librarians, and other cultural heritage applications. Photo from March 2023, courtesy Jeff Smith."
    title: "Teaching"
    excerpt: "I have taught various learner groups at the University of Michigan School of Information, the Library of Congress, the University of Maryland, and elsewhere."
    url: /teaching/
    btn_class: btn--primary
#    btn_label: "My Teaching"
  - image_path: /assets/images/rainbow-bookshelf-th.jpg #baroque-library-th.jpg 
#    image_caption: "Photo by [Valdemaras D.](https://unsplash.com/@deko_lt) on [Unsplash](https://unsplash.com/s/photos/library)"
#    alt: "Image of baroque library shelves and a ladder to access the upper shelves."
    alt: "Image of a bookshelf showing books of different colors arranged in a rainbow order."
    title: "Writing"
    excerpt: "I write for academic and public audiences. Click here to see what I've written for the public sector and for academia, ranging from advice for grantseekers to humanities data curation, digital preservation, and ethnomusicology."
    url: /writing/
    btn_class: btn--primary
#    btn_label: "My Writing"
  - image_path: /assets/images/open-books-th.jpg
#    image_caption: "Photo by [Patrick Tomasso](https://unsplash.com/@impatrickt) on [Unsplash](https://unsplash.com/s/photos/library)"
    alt: "Image of many books opened to different pages of text."
    title: "Research"
    excerpt: "My interdisciplinary, qualitative and interpretive research investigates digital preservation, recordkeeping and recordmaking policy, archives and anthropology, and ethnomusicology. Learn more about my research here."
    url: /research/
    btn_class: btn--primary
#    btn_label: "My Research"
#  - image_path: /assets/images/rainbow-bookshelf-th.jpg
#    image_caption: "Photo by [Jason Leung](https://unsplash.com/@ninjason) on [Unsplash](https://unsplash.com/s/photos/library)"
#    alt: "Image of a rainbow-styled bookshelf, showing rows of book spines with red, yellow, and orange colors."
#    title: "Public Writing"
#    excerpt: "I have written features for public audiences for a variety of federal government websites as well as my own blog. These pieces span topics from work in the humanities to data curation, digital preservation, and ethnomusicology."
#  - image_path: /assets/images/lecture-hall-empty-th.jpg
#    image_caption: "Photo by [Changbok Ko](https://unsplash.com/@kochangbok) on [Unsplash](https://unsplash.com/s/photos/teaching)"
#    alt: "Image of many rows of seats arrayed in semicircles and rising to a far wall. This appears to be an empty lecture hall."
feature_row2:
  - image_path: /assets/images/open-books-th.jpg
    image_caption: "Photo by [Patrick Tomasso](https://unsplash.com/@impatrickt) on [Unsplash](https://unsplash.com/s/photos/library)"
    alt: "Image of many books opened to different pages of text."
    title: "Research"
    excerpt: "My interdisciplinary work includes research in multiple areas, spanning digital preservation, libraries, audio archives, and ethnomusicology."
    url: /research/
    btn_label: "My Research"
    btn_class: "btn--inverse"
feature_row3:
  - image_path: /assets/images/jesse-classroom-leftfacing-th.jpg
    alt: "Jesse teaching a Library Carpentry workshop at the Library of Congress for a group of librarians from across the federal government."
    title: "Teaching"
    excerpt: "I have taught courses for various learner audiences from librarians-in-training to liberal arts students, and served as an instructor at the Library of Congress, University of Maryland College of Information Studies, George Mason University, Bowling Green State University, and the University of Michigan."
    url: /teaching/
    btn_label: "My Teaching"
    btn_class: "btn--warning"
feature_row4:
  - image_path: /assets/images/baroque-library-th.jpg #rainbow-bookshelf-th.jpg
    image_caption: "Photo by [Valdemaras D.](https://unsplash.com/@deko_lt) on [Unsplash](https://unsplash.com/s/photos/library)"
    alt: "Image of baroque library shelves and a ladder to access the upper shelves."
    title: "Writing"
    excerpt: "I write for academic and public audiences. Click here to see what I've written for the public sector and for academia, ranging from advice for grantseekers to humanities data curation, digital preservation, and ethnomusicology."
    url: /writing/
    btn_label: "My Writing"
    btn_class: "btn--info"
---
{% comment %}
to insert a centered, paragraph with the intro
{% include feature_row id="intro" type="center" %}
{% endcomment %}

{% comment %}
for horizontal 3-column row:
{% endcomment %}
{% include feature_row %}

{% comment %}
to include various other "feature" rows with different alignments
{% include feature_row id="feature_row2" type="left" %}

{% include feature_row id="feature_row3" type="right" %}

{% include feature_row id="feature_row4" type="left" %}
{% endcomment %}


# About Me

{% include content-about.html %}

{% comment %}
If desired, could insert similar research statement page here dynamically, too.
{% endcomment %}
