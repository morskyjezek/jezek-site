---
title: 'Building Jekyll Site Pages with Python'
date: 2021-01-06
permalink: /posts/2021/building-site-pages-with-python/
excerpt: 'After moving this site to jekyll, I developed a new python script to automatically generate individual pages for my teaching experiences. This post provides the code developed in a Jupyter notebook.'
header:
  overlay_image: "board-electronics.png"
  overlay_filter: 0.3
  image_description: "A closeup image of a green circuit board with silver circuit lines running diagonally across it."
  caption: "Image by [blickpixel](https://pixabay.com/users/blickpixel-52945/?utm_source=link-attribution&amp;utm_content=453758) on [Pixabay](https://pixabay.com/photos/board-electronics-computer-453758/)"
  og_image: "board-electronics.png"
  teaser: "board-electronics-th.jpg"
categories:
  - technology
tags:
  - python
  - data
  - data wrangling
  - web publishing
---

I recently migrated this site from a custom-built application running on GoogleAppEngine to an easier-to-maintain site that is powered via Github pages. I've been interested in creating a site of static HTML pages powered by jekyll for a while now, but until GitHub added the possibility to automagically publish repositories to the web, the greater support for [jekyll](https://jekyllrb.com/) natively, and the growing use of templates like this ([academic-pages](https://github.com/academicpages/academicpages.github.io)), I was hesitant to make the move. 

The "Academic Pages" template, updated and made available by Stuart Geiger, has a lot of nice features, including support for individual pages devoted to talks, publications, and teaching. Many of these are supported with helpers that assist in generating the markdown files that are needed to create individual pages, including [this Jupyter notebook that generates markdown for individual talks and served as a template for the code below](https://github.com/academicpages/academicpages.github.io/blob/master/markdown_generator/talks.ipynb). However, the templates did not create tools to create new entries for teaching experiences. Since I wanted to include teaching samples, I created a new tool that creates these pages using data from a source `csv` file. Using this tool, you can list out your teaching in a relatively simple list, then create all of the markdown files from that list. Following the model of the other generators, which are in jupyter notebooks, here's how the code looks:

# Teaching markdown generator for academicpages

Takes a CSV of teaching experience with metadata and converts them for use with [academicpages.github.io](academicpages.github.io). This is an interactive Jupyter notebook ([see more info here](http://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/what_is_jupyter.html)). Run either from the `markdown_generator` folder after replacing `teaching.csv` with one containing your data.


```python
import pandas as pd
import os
```

## Data format

The CSV needs to have the following columns: title, type, url_slug, venue, date, location, description, with a header at the top. Many of these fields can be blank, but the columns must be in the CSV.

- Fields that cannot be blank: `title`, `url_slug`, `date`. All else can be blank. `collection` defaults to "teaching" 
- `date` must be formatted as YYYY-MM-DD.
- `url_slug` will be the descriptive part of the .md file and the permalink URL for the page about the paper. 
    - The .md file will be `YYYY-MM-DD-[url_slug].md` and the permalink will be `https://[yourdomain]/teaching/YYYY-MM-DD-[url_slug]`
    - The combination of `url_slug` and `date` must be unique, as it will be the basis for your filenames

This is how the raw file looks (it doesn't look pretty, use a spreadsheet or other program to edit and create).


```python
!cat teaching.csv
```

    title,type,url_slug,venue,date,location,description
    Implementing Digital Curation (INST 742),Graduate course,implementing-digital-curation,University of Maryland,2019-01-01,"College Park, MD",This graduate course explored advanced techniques for managing digital projects for students training to become archivists and librarians. The [2019 syllabus is available here](https://goo.gl/udK14A).
    Implementing Digital Curation (INST 742),Graduate course,implementing-digital-curation,University of Maryland,2018-01-01,"College Park, MD",This graduate course explored advanced techniques for managing digital projects for students training to become archivists and librarians. The [2018 syllabus is available here](https://goo.gl/NuxV6y).
    Digital Preservation (LBSC 784),Graduate course,digital-preservation,University of Maryland,2015-09-01,"College Park, MD",This graduate course taught the fundamentals of digital preservation for Master's students in the library and archvies tracks at the Maryland iSchool. The [2015 syllabus is available here](https://goo.gl/qH2nI1).
    Administration of Archives and Manuscripts (History 690),Graduate course,intro-to-archives,George Mason University,2016-09-01,"Fairfax, VA","This graudate course introduced the principles and concepts necessary for managing and understanding archival collections, with a focus on translating the skills of public historians to work in archives. The [2016 syllabus is available here](https://goo.gl/cf6UjG)."
    Music Bibliography (MHM 503),Graduate course,music-bibliography,University of Michigan,2012-09-01,"Ann Arbor, MI",This syllabus outlines a graduate course for music performers and researchers to understand the major reference resources and concepts for designing and carrying out music-focused research. The 2012 syllabus is available here](http://www-personal.umich.edu/~jajohnst/courses/mhm503-2012/).
    Music of World Cultures,Undergraduate course,music-of-world-cultures,Bowling Green State University,2009-09-01,"Bowling Green, OH","This course surveyed selected traditional and popular music cultures from around the globe, while also teaching basic listening and writing skills."
    Music of World Cultures,Undergraduate course,music-of-world-cultures,Bowling Green State University,2010-01-01,"Bowling Green, OH","This course surveyed selected traditional and popular music cultures from around the globe, while also teaching basic listening and writing skills."

## Import CSV

Pandas makes this easy with the `read_csv` function. We are using a CSV, so we specify the separator as a comma, or `,`.

I found it important to put this data in a tab-separated values format, because there are a lot of commas in this kind of data and comma-separated values can get messed up. However, you can modify the import statement, as pandas also has read_excel(), read_json(), and others.


```python
teaching = pd.read_csv("teaching.csv", sep=",", header=0)
teaching
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>type</th>
      <th>url_slug</th>
      <th>venue</th>
      <th>date</th>
      <th>location</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Implementing Digital Curation (INST 742)</td>
      <td>Graduate course</td>
      <td>implementing-digital-curation</td>
      <td>University of Maryland</td>
      <td>2019-01-01</td>
      <td>College Park, MD</td>
      <td>This graduate course explored advanced techniq...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Implementing Digital Curation (INST 742)</td>
      <td>Graduate course</td>
      <td>implementing-digital-curation</td>
      <td>University of Maryland</td>
      <td>2018-01-01</td>
      <td>College Park, MD</td>
      <td>This graduate course explored advanced techniq...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Digital Preservation (LBSC 784)</td>
      <td>Graduate course</td>
      <td>digital-preservation</td>
      <td>University of Maryland</td>
      <td>2015-09-01</td>
      <td>College Park, MD</td>
      <td>This graduate course taught the fundamentals o...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Administration of Archives and Manuscripts (Hi...</td>
      <td>Graduate course</td>
      <td>intro-to-archives</td>
      <td>George Mason University</td>
      <td>2016-09-01</td>
      <td>Fairfax, VA</td>
      <td>This graudate course introduced the principles...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Music Bibliography (MHM 503)</td>
      <td>Graduate course</td>
      <td>music-bibliography</td>
      <td>University of Michigan</td>
      <td>2012-09-01</td>
      <td>Ann Arbor, MI</td>
      <td>This syllabus outlines a graduate course for m...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Music of World Cultures</td>
      <td>Undergraduate course</td>
      <td>music-of-world-cultures</td>
      <td>Bowling Green State University</td>
      <td>2009-09-01</td>
      <td>Bowling Green, OH</td>
      <td>This course surveyed selected traditional and ...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Music of World Cultures</td>
      <td>Undergraduate course</td>
      <td>music-of-world-cultures</td>
      <td>Bowling Green State University</td>
      <td>2010-01-01</td>
      <td>Bowling Green, OH</td>
      <td>This course surveyed selected traditional and ...</td>
    </tr>
  </tbody>
</table>
</div>



## Escape special characters

YAML is very picky about how it takes a valid string, so we are replacing single and double quotes (and ampersands) with their HTML encoded equivilents. This makes them look not so readable in raw format, but they are parsed and rendered nicely.


```python
html_escape_table = {
    "&": "&amp;",
    '"': "&quot;",
    "'": "&apos;"
    }

def html_escape(text):
    if type(text) is str:
        return "".join(html_escape_table.get(c,c) for c in text)
    else:
        return "False"
```

## Creating the markdown files

This is where the heavy lifting is done. This loops through all the rows in the TSV dataframe, then starts to concatentate a big string (```md```) that contains the markdown for each type. It does the YAML metadata first, then does the description for the individual page.


```python
loc_dict = {}

for row, item in teaching.iterrows():
    
    md_filename = str(item.date) + "-" + item.url_slug + ".md"
    html_filename = str(item.date) + "-" + item.url_slug 
    year = item.date[:4]
    
    md = "---\ntitle: \""   + item.title + '"\n'
    md += "collection: teaching" + "\n"
    
    if len(str(item.type)) > 3:
        md += 'type: "' + item.type + '"\n'
    else:
        md += 'type: "Course"\n'
    
    md += "permalink: /teaching/" + html_filename + "\n"
    
    if len(str(item.venue)) > 3:
        md += 'venue: "' + item.venue + '"\n'
        
    if len(str(item.date)) > 3:
        md += "date: " + str(item.date) + "\n"
    
    if len(str(item.location)) > 3:
        md += 'location: "' + str(item.location) + '"\n'
           
    md += "---\n"
    

    if len(str(item.description)) > 3:
        md += "\n" + html_escape(item.description) + "\n"
        
        
    md_filename = os.path.basename(md_filename)
    #print(md)
    
    with open("../_teaching/" + md_filename, 'w') as f:
        f.write(md)
```

These files are in the teaching directory, one directory below where we're working from.


```python
!ls ../_teaching
```

    2009-09-01-music-of-world-cultures.md
    2010-01-01-music-of-world-cultures.md
    2012-09-01-music-bibliography.md
    2015-09-01-digital-preservation.md
    2016-09-01-intro-to-archives.md
    2018-01-01-implementing-digital-curation.md
    2019-01-01-implementing-digital-curation.md



```python
!cat ../_teaching/2009-09-01-music-of-world-cultures.md
```

    ---
    title: "Music of World Cultures"
    collection: teaching
    type: "Undergraduate course"
    permalink: /teaching/2009-09-01-music-of-world-cultures
    venue: "Bowling Green State University"
    date: 2009-09-01
    location: "Bowling Green, OH"
    ---
    
    This course surveyed selected traditional and popular music cultures from around the globe, while also teaching basic listening and writing skills.
