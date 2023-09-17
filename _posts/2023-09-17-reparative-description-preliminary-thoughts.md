---
title: 'Encoding Reparative Description: Preliminary Thoughts'
date: 2023-09-17
permalink: /posts/2023/reparative-description-preliminary-thoughts/
excerpt: 'How can coding and programming provide supportive and ethical support for reparative description?'
header:
  image: "/assets/images/code-colors-1280w.jpg" # this required to display custom twitter card image for the post, but does it override the hero image (overlay_image)?
  overlay_image: "/assets/images/code-colors-1280w.jpg"
  overlay_filter: 0.3
  image_description: "An image displaying multicolored lines of code."
  caption: "Photo by [Markus Spiske](https://unsplash.com/@markusspiske) on [Unsplash](https://unsplash.com/s/photos/code)"
  teaser: "/assets/images/code-colors-th.jpg"
  og_image: "/assets/images/code-colors-th.jpg"
categories:
  - data curation
tags:
  - python
  - tools
  - humanities data
---

Since the beginning of this year, I have been working with the project team for [ReConnect/ReCollect](https://www.reconnect-recollect.com/), a project supported by the University of Michigan to examine the legacy of its colonial collections from the Philippines. As the project puts it, they is " committed to developing models for culturally-responsive and historically-minded stewardship of the Philippine collections at the University of Michigan."  
One portion of this work may be described as "reparative description," which in this case refers to the addresing of harmful, outdated, or incorrect terminology and language in the description of these collections. 

Because these collections are extensive and spread across multiple repositories in the University, making changes is not a simple task of updating a few database records. It also requires institutional coordination and changes that must be implemented in multiple systems by multiple people. 
To address this challenge, I worked with the team to develop tools that would help to understand the metadata across different repositories and to begin to identify pathways of addressing various problems that the project has identified. We quickly began to realize that bulk editing and analysis approaches would usefully complement the work. By analyzing the collections as a whole, we hoped to develop understandings that could inform or confirm our analyses, particularly the insights and recommendations arising from the analysis of harmful terminology. We were specifically hoping to develop tools that would help in the analysis of hundreds of descriptive records, which were identified by the project during our collections survey, in aggregate. While much work in the rest of the project has focused on collection inventory and analysis, we also hoped to create methods that could assist collections managers in analyzing and understanding collection metadata, with the ultimate goal of creating reusable, code-based tools that could assist our project and others in changing or updating collection descriptions. 

The project team identified more than two hundred finding aids that described materials at the University of Michigan related to the Philippines. These documents provided the basis of a dataset for analysis. These finding aids came from three collections on campus: the Bentley Historical Library, Special Collections Research Center, and the Clements Library. All provided data in a text-based markup format known as Encoded Archival Description (EAD), which was shared in eXtensible Markup Language, a standard data format that is frequently used to share metadata. The analysis of the data was undertaken by graduate student Ella Li (School of Information) and myself. We worked together to develop a series of analysis tools. The goal was to analyze the data, with the end goal of producing useful data visualizations, and to begin creating tools that could be used or repurposed to begin making changes to the descriptions. We developed analysis modules using the Python programming language and widely used analysis and visualization tools. The resulting code is available, and can be reused or repurposed, through a series of [interactive code examples now available on GitHub](https://github.com/jiaqili0803/ReConnect-ReCollect_Automation). 