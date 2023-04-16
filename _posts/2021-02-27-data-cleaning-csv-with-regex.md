---
title: 'Wrangling Humanities Data: Using Regex to Clean a CSV'
date: 2021-02-27
last_modified_at: 2023-04-16
permalink: /posts/2021/data-cleaning-csv-with-regex/
excerpt: 'Have you heard of regular expressions before and wondered how to make use of them? This post is for someone who has asked this question. It assumes a basic understanding of "regex" and shows how to use a full-featured text editor to cleanup plain-text data.'
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
  - regex
  - tools
  - humanities data
---

{% include intro_humanities_data_curation.md %}

Have you heard of regular expressions before and wondered how to make use of them? This post is for someone who has asked this question. It assumes a basic understanding of "regex" and shows how to use a full-featured text editor to cleanup plain text data. The data in question comes from a larger project, which is pulling bibliographic data from a major citation database in CSV form, transforming the data and extracting certain elements (DOIs of publications), then feeding the information into Zotero to create a shared bibliography. At some point in summer 2019, the CSV files began to include new fields that contained line breaks and non-text characters, which broke my data workflow. In a project that I could previuosly do with the output from the database, I now needed to clean up the CSV with the formatting errors. At first, this was not too onerous - a few lines to delete - but after a month, this grew to hundreds of lines in a CSV with thousands of lines. I needed a way to quickly search for the error patterns and fix as many problems at once in a batch. I decided to explore regex as a solution. Around the same time, I taught a workshop that included an overview of regular expressions, and someone asked for a "real world" use case for regex that could illustrate how to implement regex in a functional way. This is the use case.

## Project background: the use case

The use case arose from an ongoing project that arose from the Covid-19 pandemic. Even after the pandemic had only lasted a couple of months, it had already brought about major reorientations in health and bioscience research. This first affected researchers in epidemiology and the health sciences who immediately began to on new vaccines, but it quickly included research into the effectiveness of public health measures, and the direct impact as well as cascading effects of the disease on particular communities, among many other areas. At large research universities, active research into the novel coronavirus immediately ramped up, and many existing research projects and labs have been reoriented to investigate the virus and the disease. This resulted in a flood of medical, scientific, and social research publications. At the University of Michigan, I began a project to track these publications starting in April 2020. As with so many things involving the Covid pandemic, our work has been responsive, and quickly adapting to the situation; in the last ten months, we honed the process of identifying citations, identified multiple ways to present the list of publications, and reworked the workflow to gather and process the data. 

In this post, I will explain how I've been using advanced text editors and pattern recognition routines to parse and clean the data we're gathering. Specifically, I will demonstrate how I use [Virtual Studio Code (aka VSCode)](https://code.visualstudio.com/) to clear up some data quality issues, including multi-point editing and regular expression strings to identify patterns for correction. If you are looking for a text editor, the [wikipedia comparison of text editors](https://en.wikipedia.org/wiki/Comparison_of_text_editors) is a good place to start; in the past, I have used TextWrangler, Sublime, Brackets, and Atom, but at present VSCode is an excellent option. In future, I plan to add another post or two to this that will explain the data workflow in more detail, since this is only one of many steps. The outcome of the workflow is to create the list of publications that are included in a publicly-available bibliography at [https://myumi.ch/3qnOG](https://myumi.ch/3qnOG) (via Zotero).

Okay, let's get into text editors and regex!

## Using the text editor

![png](/assets/images/wrangling-humanities-data/vscode-csv-editing.png "An image showing the visual display of a CSV file in the VSCode interface.")

Displayed above is the CSV file, as it appears in VSCode, which I discuss in more detail below. Note that the rows are very easy to distinguish and individual fields as noted in the header row are color-coded, which makes the data somewhat easier to read in lower rows. Various extensions can be added to VSCode to aid in the processing of CSV files, which are available in the VSCode extension "marketplace" (note that most of the CSV extensions are free). In the above illustration, I am using the "[Rainbow CSV](https://marketplace.visualstudio.com/items?itemName=mechatroner.rainbow-csv)" extension to make the "cells" more visible to a human reader - this makes each field in the line a different color, which is much easier to see than a tiny comma.

### Power editing with keyboard commands

One of the first things I noted when viewing the file is that the first line is not one that is necessary for my project (it provides information about the file and the specific search string that was used to generate the result), and the second line contains the field headers. 

So, I deleted the first line. In VSCode, you can double click on the line to highlight it, then delete. Or, if you want to use the keyboard, position the cursor on the line you want to delete, then type `Ctrl + X` (`Cmd + X` on MacOS). 

VS Code is full of handy keyboard shortcuts, and you can even create new ones! If you use shortcuts frequently, there are lists of useful shortcuts, like [this one from VS Code developers for Windows users](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf), or [this one for Mac users](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf). Print one out and keep it beside your desk until you learn them all! Or, see the many lists generated by other shortucut users, like [this list from Deepak Gupta](https://medium.com/better-programming/20-vs-code-shortcuts-for-fast-coding-cheatsheet-10b0e72fd5d), for additional useful keyboard commands in VSCode. You can even create new shortcuts, which are called "key bindings," [within VSCode using built-in features](https://code.visualstudio.com/docs/getstarted/keybindings).

## Using regular expressions 

Regular expressions are a useful set of pattern-searching techniques, which allow you to find very specific patterns within text. For example, have you ever searched in multiple sites and noticed that you want to find things with both British and American spellings? For example, all of the times where the word digitize appears, but you know that it might be spelled digitize or _digitise_? Regular expressions can help! In this case, if you have a search tool that can search with regular expressions, you could input the string `digiti[sz]e`, and it would be able to match either spelling. The regular expression syntax is complicated and can be quite powerful, but I am only going to go into a few specific search expressions in this post. If you're interested to learn more about regular expressions, or "regex" as they are often called, check out the [introduction from Library Carpentry](https://librarycarpentry.org/lc-data-intro/01-regular-expressions/index.html), or search for one of the many cheatsheets available online, such as this [Regex cheat sheet from MIT](http://web.mit.edu/hackl/www/lab/turkshop/slides/regex-cheatsheet.pdf).

![Detail from the VS Code interface: the find and replace popup window.](/assets/images/wrangling-humanities-data/vscode-csv-find-replace.png "An image showing a detail from the VS Code interface: the find and replace popup window.")

Many advanced text editors support the use of regular expressions, which can be used to conduct advanced searches. In VSCode, you can bring up the window by typing `Ctrl + F` (`Cmd + F` on MacOS) or opening the `Edit` pull-down menu and selecting `Find`. The find and replace console (above) appears at the upper right corner of the VSCode window, and you can activate (or deactivate) the option to use regular expressions in searches by selecting the button at the right end of the search input prompt (above, note that `.*` is highlighted).

### Locating errors in a CSV using regex

If you start to see particular patterns that are beyond matching a word or phrase, you may want to consider using regular expressions. For example, are there many lines that begin with blank spaces or letters, when they should begin with numbers? Or are there some lines that can begin with letters, but only if they are followed by a line that begins with a number? If you have identified pattersn like these, regex may be a tool that you want to use. Regex also offers more advanced usage that supports selecting and changing certain patterns. (I will not discuss regex replacement features here, but you can learn more about that functionality in [Bohyun Kim's post at the ACRL TechConnect Blog](https://acrl.ala.org/techconnect/post/fear-no-longer-regular-expressions/).) 

In this post, I am using basic regex in this example to identify certain patterns that cause problems in a CSV document that I received. When I opened the file, I noticed that many lines included unescaped commas, non-alphanumeric characters, tabs, or other strange formatting that caused the CSV to be inaccurate. While fixing the CSV and preserving the data would require more refined regex work, I needed to get rid of the errant lines while preserving the lines that were correct, plus the first three columns (I need a list of the DOI entries, which are in the third column). Here's how I used regular expressions and VSCode to do that:

#### Identify lines that do not begin with a numerical index

The lines that were not "broken" all began with a number. Lines that did not begin with a number were causing problems in the file's format, which caused the file to be an invalid CSV. To identify these lines, I searched for any line that begins with an upper- or lower-case letter, which was not followed by a line beginning with a number:

```regex
^[A-Za-z].*\n(?!^[0-9])
```

Using VSCode, I selected each of these patterns by using the "multi-cursor" option in VSCode. To select all of the matching lines, type `Shift + Ctrl + L` (or `Shift + Cmd + L` on MacOS). I made sure the cursor was at the beginning of each of these lines, then deleted the selected text to remove the unwanted lines using `Ctrl + X` (`Cmd + X` on MacOS). 

#### Identify blank lines

Some lines were completely blank, or appeared to be. This expression matches all lines that are blank or include space characters. Once selected, again delete the selection in batch using the multiple select method above.

```regex
^\s*$
```

#### Identify lines that begin with blank spaces or tabs

Reviewing the remaining lines, I noticed that many of the lines were not blank, but they did begin with spaces or tab characters. The previous pattern did not match them since they were not blank lines. Most of the tabs (though not all) were converted to sequences of 6 or 8 spaces. To catch this case, I created a pattern to look for 6 spaces at the beginning of the line (this also catches the cases that have 8 spaces), plus any characters following these up to a line break (`\n`). Then, using the negative look ahead pattern (parentheses at the end of the line), the pattern checks the following line to see if it begins with a numeral; if the line does begin with a numeral, then the line is skipped and not matched. The reason for skipping the trailing line is to reduce the possibility of deleting needed information and fields. 

```regex
^[\s]{6}.*\n(?!^[0-9])
```

![png](/assets/images/wrangling-humanities-data/regex-6-blanks-not-preceeding-numeral-at-beginning-of-line.png "A visualization that provides a visual process diagram of how the regular expression is matched.") 

This is how [regexper visualizes](https://regexper.com/#%5E%5B%5Cs%5D%7B6%7D.*%5Cn%28%3F!%5E%5B0-9%5D%29) the match.

Instead of using the delete line method of removing the material, as previously, I used a different approach this time. This pattern selects the entire line, so using the multiple select selects the entirety of the undesired content. To remove this, use the multiple select (`Shift + Ctrl + L` or `Shift + Cmd + L`), then hit the delete key. Bye bye!

#### Identify lines that begin with tabs

There were still some lines that looked blank, which turned out to be special tab characters. Regex allows you to look for these with `\t`, so I searched for any cases where the character occured at the beginning of the line and paired this with the look ahead to avoid any lines with additional content.

```regex
^[\t].*\n(?![0-9])
```
Then select, delete, and buh bye!

#### Identify lines with odd characters

At this point, a few lines remained with "odd characters" at the beginning, which turned out to be bullet point characters. After a few tries, I realized serendipitously that the following regex will select these lines:

```regex
^.\n(?![0-9])
```

This is selecting any line that is not followed by a line that begins with a number. Then, select those remaining and delete! This time I used the remove line (`Ctrl + X`) again.

#### Identify and remove remaining unwanted content

Now, most of the blank or extra lines are gone, but there are still the remaining lines that begin in the middle of a cell Now that the other lines are gone, I can identify these lines by matching anything that _doesn't_ have a number at the beginning. Then, to prevent the deletion of content beyond the cell (delineated by a double quotation mark), the pattern stops when it finds a double quotation mark.

```regex
^[^0-9][^"]+
```

This is a good pattern for error-checking CSVs (if you have lines beginning with a numerical index): it matches lines not beginning with a digit, then matches up to a double quotation mark. will select text to delete and catches most of the abstracts that do not have multiple sentences. 

#### Stragglers

There are still a few lines that need help. I identified these by searching for lines that don't begin with a digit:

```regex
^[^0-9]
```

At this point, there are only a few (for February's CSV there were only 3!), and these can be corrected one by one.

Finally, I use the CSVLint extension of VSCode, which looks for errors. There were a few lines that were missing a cell, which means that some of these patterns were selecting and/or deleting too much. These lines still had the most important information - the DOI - that I wanted, so I fixed each line by adding an extra cell at the end by appending a comma to the end of the line. 

## Conclusion

Regular expressions are fussy and esoteric. It 
is not always clear why certain patterns match (obviously the computer 
knows, but it can take a while to decrypt what is happening even if you 
are the person who wrote the expression), and the patterns often don't do quite what you think they will (note that I still had about 6 lines to fix by hand because of uncaught errors). That said, there is something fun in thinking through how the pattern will work, and whether it is going to match exactly the content that you want. 

Perhaps that joy of finding and matching patterns is likely what 
some people find 
appealing about the game of "regex golf". That's basically a riddle game where you have two groups of strings (say, the titles of _Star Trek_ and _Star Wars_ movies), then you try to create a pattern that matches all of the items in one list, but none of the ones in the other list. I'm not sure I would play the game, but after working through some "real world" examples, I can see the appeal (but also frustration) in that work. To end, here is an XKCD about regex golf (I had to read [the explanation](https://www.explainxkcd.com/wiki/index.php/1313:_Regex_Golf)): 

![png](https://imgs.xkcd.com/comics/regex_golf.png "A comic strip from XKCD that illustrates the game of 'RegEx Golf'.")

I am not likely to choose this approach to cleaning up a CSV file in 
future. Instead, I would use a tool specialized for CSV as a format. 
Nonetheless, this was a good way to practice with regular expressions.

## Resources

If you are interested in working with regular expressions, you may find these resources, including a few helpful validators and visualizers, of use: 

* [Regex101](https://regex101.com/) allows you to test patterns agains text that you define  
* [Regexper](https://regexper.com/#%5E%5B%5E0-9%5D%2B%5C.%28%3F%3D%22%29) allows you to input expressions and then creates visualizations what the pattern matcher will do 
* Information about ["lookaround" patterns](http://www.rexegg.com/regex-lookarounds.html) from [rexegg](http://www.rexegg.com/)
* Regex cheatsheet from [MIT](http://web.mit.edu/hackl/www/lab/turkshop/slides/regex-cheatsheet.pdf)
* Bohyun Kim's post at the ACRL TechConnect Blog, "[Fear No Longer Regular Expressions](https://acrl.ala.org/techconnect/post/fear-no-longer-regular-expressions/)"
* [Library Carpentry](https://librarycarpentry.org/) offers a [great introduction to regular expressions](https://librarycarpentry.org/lc-data-intro/01-regular-expressions/index.html)
