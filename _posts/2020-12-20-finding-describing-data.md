---
title: 'Wrangling Humanities Data: Finding and Describing Data'
date: 2020-12-20
permalink: /posts/2020/finding-describing-data/
excerpt: 'This post begins a series that demonstrates steps in a data preservation and analysis workflow. This step walks through the process of creating essential preservation metadata to transform a group of files into an archival information package.'
header:
  overlay_image: "/assets/images/binary-1280w.jpg"
  overlay_filter: 0.3
  image_description: "Rows and columns of 0 and 1 with a blue background."
  caption: "Photo by [Gerd Altmann](https://pixabay.com/illustrations/binary-digitization-null-one-pay-1377017/) on Pixabay"
  og_image: "/assets/images/binary-1280w.jpg"
  teaser: "/assets/images/binary-th.jpg"
categories:
  - mapping humanities
  - data curation
tags:
  - python
  - tools
  - neh 
  - digital preservation
  - fixity
  - humanities data
classes: wide
---

{% include intro_humanities_data_curation.md %}

*This is the first post in a [series](/categories/#mapping-humanities) about about curating humanities data. In this installment, I discuss how to find the data, then walk through an initial workflow to describe and preserve the data. I will explain how to provide basic metadata (provenance and description) about a dataset using the `BagIt` python library to establish fixity information about the data. This workflow follows some of the fundamental [digital collection management policies that are established by the Library of Congress](https://www.loc.gov/programs/digital-collections-management/about-this-program/).*

# Background

I first began this project with a basic question: how broadly has the National Endowment for the Humanities distributed federal support across the country? While the National Endowment for the Arts (NEA) routinely discusses how its support has gone to every Congressional district in the United States, the NEH is much less specific about how its funds are distributed. NEH can provide state-by-state profiles of funding over time and can provide custom reports, but they are generally basic, text-based lists of grant titles and amounts. So when I started this project in 2014 (at the time, I was an NEH program officer), I wanted to create a more user-friendly, accessible, and dynamic tool available over the web. At that time, it was possible to transform the agency's public-facing data (downloadable in XML files, more on that later), and to use free cloud-based tools like Google Maps and Fusion Tables, to create some inital geospatial analysis tools. With the sunsetting of Fusion tables in 2019, this tool no longer worked. This project takes new steps to revise the project, while also exploring data curation for humanities information in the process. 

In 2014, I was not aware of interactive maps that displayed the geographical distribution of NEH funding. Since that time, other projects have created such maps, such as those of ["historical organizations" produced by Engaging Places](https://engagingplaces.net/tag/maps/). Likewise, the Mellon-funded [NEH for All project has mapped selected projects](https://nehforall.org/programs/geographic-information) at the National Humanmities Alliance, have created user-friendly dynamic maps and case studies to demonstrate NEH impact to lawmakers and other decision makers according to various categories of impact. These projects offer appealing information for communicators, with appealing details, and they gather together detailed background research and talking points for advocates. Yet, it isn't clear how much of the original NEH data is presented. So in this new phase of the project, six years later, the goal of creating a geographci interface is less urgent. Instead, I aim to explain some of the data curation processes involved in finding and using the humanities data, and as a second step to update the original mapping project as a case study to illustrate ways to transform and analyze humanities data.

And a final caveat about this humanities data project and its scope. A map of a grant recipient's address does not necessarily show where cultural activity is taking place, or where there is a "humanities project". For example, many of NEH's funds go to state councils in each US state and territory, which then typically makes smaller grants around the state. Other projects support researchers who might go to a different place to do their project, such as a historian at a specific university who might travel to another community to visit an archive or gather information. So a geographical point is not a direct indicator, but it nonetheless offers a good starting point. So at least a few basic maps that show where award recipients are located can be a starting point in answering my intial question, how broadly has federal funding for the humanities been distributed across the country? For now, I will keep the scope on funds from the NEH, and focused on funds awarded to US states and territories. 

# Finding the data

The first step is to find the data. Many federal funders, including the NEH, NEA, and the Institute for Museum and Library Services (IMLS), provide data about their awards over the years. The IMLS, for example, provides a [data catalog](https://www.imls.gov/research-tools/data-collection) and [state-by-state reports](https://www.imls.gov/our-work/your-state). NEA makes state-by-state information available in an [interactive web interface](https://www.arts.gov/impact/state-profiles), which summarizes funding information over time and highlights notable projects that have been supported in a place. NEH, meanwhile, has much less data available on its website. This is in part because the work on assessing the impact of the humanities has been outsourced to the [Humanities Indicators project](https://en.wikipedia.org/wiki/Humanities_Indicators) at the American Academy of Arts & Sciences. Most of that project's information is available in static publications, rather than interactive visualizations on the web. 

The main source for the NEH grant data, however, is the agency's open data page. As far as it is possible to tell, the data is output from the agency's database that manages the intake, review, and administration of all federal funds awarded by the NEH. This data is described and provided for download at [https://securegrants.neh.gov/open/data/](https://securegrants.neh.gov/open/data/). The data is also provided and linked via data.gov at [https://catalog.data.gov/dataset?publisher=National+Endowment+for+the+Humanities](https://catalog.data.gov/dataset?publisher=National+Endowment+for+the+Humanities).
The data is created and published by the US Government and is therefore not subject to copyright; the data.gov entry offers the data under  a [CCZero](http://www.opendefinition.org/licenses/cc-zero) license, which means that it is not subject to copyright restrictions.  

**Note that the rest of this post is also available as a [Jupyter Notebook](https://github.com/morskyjezek/neh-grant-data-project/blob/main/01-inventory-and-fixity.ipynb), which can be downloaded from the GitHub repository with all of the data discussed here. File references discussed below indicate files included in my [neh-grant-data-project repository](https://github.com/morskyjezek/neh-grant-data-project).**

# Using Python to establish inventory and metadata

This section shows the process that I followed, step by step in a Jupyter notebook, 
to demonstrate the creation of data integrity and provenance information 
using Python and the bagger module to create metadata, inventory the files, 
establish fixity information for the original files, and 
then to create a BagIt object for the files. 

See the reference for the BagIt Python tool at https://github.com/LibraryOfCongress/bagit-python.

To begin, you'll need to ensure that the bagit module for Python is installed on your system:

## Setup


```python
# If you don't have bagit installed, install following instructions at https://github.com/LibraryOfCongress/bagit-python
# Alternatively, you can use the magic command on the line below by removing the hashtag and running the cell.
# (When the command below runs, you will see response output appear below this cell as the program downloads and installs.)
#!pip install bagit
```

When the bagit module is ready, import it:


```python
import bagit
```

## Bag the files

The purpose of the "bag" is to create information about the file structure, basic information that can demonstrate that the information has not changed, and to provide basic context (information about where the files came from, who filed them, and what they are). It is an open specification, so there are few requirements about how the files are structured. In this case, I am taking all of the files within a specific folder, using the Python bagit library to generate the fixity information, and explaining each step throughout the rest of this notebook:

A well-formed bag should include as much information on the "tag" as possible, 
since this is where we can include information about the source and provenance of the 
data. This "BagInfo" information can be added using arguments in the functions that 
create bags. This example creates a dictionary of the bag information called `my_BagInfo`,
which will be inserted as an argument during bag creation. If you use this code, 
replace information below with you the information appropriate to the project you're working on.


```python
# create baginfo data
my_BagInfo = {
    'Contact-Name' : 'Jesse Johnston',
    'Contact-Email': 'morskyjezek@gmail.com',
    'External-Description': 'NEH Grant data files downloaded from NEH in December 2020.',
    'Source-Organization': 'National Endowment for the Humanities (NEH)',
    'Source-URL': 'https://securegrants.neh.gov/open/data/',
    'Collected-Date': '2020-12-14'
}

print('Bag Info:\n',my_BagInfo)
```

    Bag Info:
     {'Contact-Name': 'Jesse Johnston', 'Contact-Email': 'morskyjezek@gmail.com', 'External-Description': 'NEH Grant data files downloaded from NEH in December 2020.', 'Source-Organization': 'National Endowment for the Humanities (NEH)', 'Source-URL': 'https://securegrants.neh.gov/open/data/', 'Collected-Date': '2020-12-14'}


Now that we have the tool working and created basic metadata for the bag, we can move on to "bag" the files. In this case, the files that we wanted to bag were in a directory named `neh-grants-data-2012`. We use the `make_bag()` function to make the bag, and we pass in as arguments the location of the files that we want to bag, the bag info (`my_BagInfo` dictionary), and in this case designated the `utf-8` text encoding:


```python
# create the bag
bag = bagit.make_bag('neh-grants-data-202012', bag_info = my_BagInfo, encoding='utf-8')
```

Now, we can use the `is_valid()` function to see if the bag object is ready, and if it is indeed an object that we can validate is a well-formed BagIt object:


```python
# check to see if the bag is valid
if bag.is_valid():
    print("yay :)")
else:
    print("boo :(")
```

    yay :)



```python
# to get an idea what's in the bag, display a list of the files and fixity information
line_count = 0
for path, fixity in bag.entries.items():
    line_count += 1
    print("%s. path:%s sha256:%s" % (line_count, path, fixity['sha256']))
```

    1. path:data/NEH_Grants1960s.csv sha256:20b521a035307e01fdfe00288806eb80498c9e9d19f02ffa3b5da5a4bfbaab41
    2. path:data/NEH_Grants1960s.zip sha256:8ae10fac88a052c0b1c7f9f35028dd2fe58950a43038c6000f9de4ed20d3e3e9
    3. path:data/NEH_Grants1960s_Flat.zip sha256:7c3c3464769e9f0918ac6afc271dd4c7cfb7d6098ea6cd510b9cd6ec674cf1b9
    4. path:data/NEH_Grants1970s.csv sha256:f1bc38c2124e6dc03fef8663a07b4496131aeba4c50ccd94266403571e097064
    5. path:data/NEH_Grants1970s.zip sha256:8fae9380ae095414bb7a2722deac19ac25b250255e4a5ad8ebf50f20ee909777
    6. path:data/NEH_Grants1970s_Flat.zip sha256:f671023d9552b95724bfe2b66d643cb7f4a7041180642f1cad971d776194d7a5
    7. path:data/NEH_Grants1980s.csv sha256:46c78195f14be0c99ca605550bd0389d9fcd6a9d698ea38f019fab639c16766c
    8. path:data/NEH_Grants1980s.zip sha256:a6739f1d981895915debacb65a86ee972d70c16ab3ecd1aeb340e9e64664173d
    9. path:data/NEH_Grants1980s_Flat.zip sha256:60d63627583b0c175535a0bb6d6044256b721d3f31ba214db825edebc29ac498
    10. path:data/NEH_Grants1990s.csv sha256:59483caaa328bd1e1f5883e6811180fd5fb50a7ffa4bc866873f42c231f5fab8
    11. path:data/NEH_Grants1990s.zip sha256:5cf59671595ab7480205276f911ccf173084b23473d1cb17015ee98414dbdb58
    12. path:data/NEH_Grants1990s_Flat.zip sha256:c756220a57f47f0fe1e975462627ec7afd6431bf738768c0d52a1c57f023f70d
    13. path:data/NEH_Grants2000s.csv sha256:e32cd052f276161ba01d8cc2ad5dab8b00a6827b4ff078b1be4266804286e28c
    14. path:data/NEH_Grants2000s.zip sha256:249a38c8514ba650b33e5d81cff805f591b33997ffc649bc519f2426145e9c65
    15. path:data/NEH_Grants2000s_Flat.zip sha256:8c15ce6bef5acf45eb8973113189ee1bff7f5f5ea526244dfcdc5238b46cc2bd
    16. path:data/NEH_Grants2010s.csv sha256:8ffc259b4b86b23b76624573ee1751e0e4573620d50d0772199542faa2435bf3
    17. path:data/NEH_Grants2010s.zip sha256:80fd5164ab027515fa4420ed1bcbfa4a913ee1bc4d4f80f14bb98dfc560dad83
    18. path:data/NEH_Grants2010s_Flat.zip sha256:d56b828a94a0c17c93b6ffd625bd56104d9df0ae3ebb4d653c8c67be75f4b76f
    19. path:data/NEH_Grants2020s.csv sha256:e85c45f1d326b6bcd0bbc044d8a224c1bac2c07c6247169b94328d0132242a62
    20. path:data/NEH_Grants2020s.zip sha256:b1cd8f3640a6ad4ed8b99b8ebaad5fab2927026a8feaf319680a5992def6c375
    21. path:data/NEH_Grants2020s_Flat.zip sha256:c756b3781a61fb82a06455478c386d8290eb6ddaeb925c6d7770ff652f7df67b
    22. path:data/NEH_GrantsDictionary.pdf sha256:8453313299588cc516f7a8233e3b04d5f6bb94bf80bcc5ebc0da7d38452b778f
    23. path:data/NEH_GrantsReference.pdf sha256:9df1d6593740b97c555fbe686b441be97d97a72cd3d051a20380eecdf69f8ddc
    24. path:data/README.md sha256:021683e6f0fb988a3226603c419339a058d8f0acf17f9c20201ad1fcb92d0897
    25. path:bagit.txt sha256:e91f941be5973ff71f1dccbdd1a32d598881893a7f21be516aca743da38b1689
    26. path:bag-info.txt sha256:1d637aa8471cfe8135be09dfc6d9fc2abe99d27bd57007822b6e3c3c195dc75b
    27. path:manifest-sha512.txt sha256:c8219eb061032b1732c525064f2d27a042144c92fc4f306b75acb9f55309e359
    28. path:manifest-sha256.txt sha256:ad6abefb8dbe8bbdd8ccb7addc1c3eb76ef7062456f199ef44adf5ca67fa8f50


Now we have created basic metadata for this group of files, two types of fixity signatures (sha256 and sha512), and an inventory of the files (see `neh-grants-data-202012/manifest-sha256.txt`). This will serve as a basic description of the original files. In the next activities, we will continue to work with this information, but the original files
will remain unchanged and available for further consultation or work beyond this project. All of our work will be to extract, transform, and clean the data that we pull from these files, which will be the basis for further computation, visualization, or other analysis. 

View the valid bag, including the `data` directory, the fixity manifest, and the tags manifest at [https://github.com/morskyjezek/neh-grant-data-project/tree/main/neh-grants-data-202012](https://github.com/morskyjezek/neh-grant-data-project/tree/main/neh-grants-data-202012).