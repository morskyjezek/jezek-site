---
title: 'Find and replace data in the shell'
date: 2022-09-11
excerpt: 'Using shell tools to quickly find and replace text data. This demonstration uses regular expressions and find, grep, and sed tools.'
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
  - humanities data
  - data wrangling
  - tools
  - regex
  - shell
  - data
---

The following demo was created for a specific task, but it might be a useful model for similar tasks: searching out specific patterns of text in a set of files and making replacements or edits according to a specific pattern. In this, case, I had a specific problem that I wanted to fix while updating this site: in a series of posts, I had used a particular text string, but I had to update and change the text string. This site runs on a static site generator platform called [Jekyll](http://jekyllrb.com), which reads through a series of plain text files formatted in markdown and generates the html to display each page on this site. Over the summer, I had created a template for introductory text on certain posts (first called `toc_mapping_humanities_data.md`). Jekyll provides a capability that allowed me to include and this boilerplate text so that it appears in any page on the site where I put the code to reference it. I had referenced this in a handful of posts, but I soon realized that in fact "mapping humanities" wasn't really correct; the main point, in fact, was humanities data curation, so I renamed that template `intro_humanities_data_curation.md`. Now, I needed to update each post where I had included the template with the new template title.

It would be possible to make these replacements by hand, but it seemed like it would be quicker if I could ask the computer to do this for me. I also thought that it might be useful to show how to use the command shell to make this update in a quick and consistent way. The basic steps would be:

1. create a way to match the text that I wanted to change
1. identify the files with the text to change
1. look through the files and replace the old text with the new text.

Each of these steps can be done with a command shell tool: the first using a regular expression, the second using [grep](https://www.gnu.org/software/grep/manual/grep.html) and [find](https://linux.die.net/man/1/find), and the third using a file editor like [sed](https://www.gnu.org/software/sed/manual/sed.html), which can use the regular expression to make the changes.

## Match the text to change

Each of the files where I used the introductory template included the following line:

{% raw %}
`{% include toc_mapping_humanities_data.md %}`
{% endraw %}

This line told the Jeykll site generator  to insert the template at this point. To include the new template, I needed to replace the above with the following line:

{% raw %}
`{% include intro_humanities_data_curation.md %}`
{% endraw %}

To do this, I created the following regular expression, which would match the portion of the line that I wanted to match. Using grouping, I could tell the command which parts I wanted to change:

```regex
(include )[a-z_]*(.md)
```

## Identify the files and make the changes

Now, `grep` and `sed` come into the picture. I used `grep` to identify all of the lines where the include command occurs:

```bash
grep 'include [a-z_]*.md' _posts/202[012]-[01]*.md
```

To make the replacement, the `sed` command. `sed` is a "stream editor" designed to replace matches in input based on pattern matching like regular expression.

To test `sed`, for example, we can pipe in a specific input:

{% highlight bash %}{% raw %}
echo "{% include toc_mapping_humanities_data.md %}" | sed -E 's/(include ).*(\.md)/\1intro_humanities_data_curation\2/'
{% endraw %}{% endhighlight %}

This is useful for testing the regular expression patterns, too. The `-E` option specifies using extended regular expressions, which are necessary to use things like group matching. In this case, the parentheses in the first pattern set groups, they are then references with numbers starting with `1` in the replacement string (i.e., `\1` and `\2`).

So the sed command may be:

{% highlight bash %}{% raw %}
sed -E 's/({% include ).*(.md %})/\1intro_humanities_data_curation\2' _posts/202*.md
{% endraw %}{% endhighlight %}

This will publish all of the modified stream (the file contents) into the terminal window or display. If you review the contents, the change has worked. Now, it needs to be rerouted into the files, if you want to make the actual changes.

## Find the files, make the changes

To do this, we can use the `find` command to make a specific search for the files, then run the sed command on each of these:

```bash
find _posts -type f -name '202[012]-[01]*.md' -exec sed -i '' -E 's/(include ).*(\.md)/\1intro_humanities_data_curation\2/' {} \;
```

The above illustrates some of the power of the `find` command. Here it searches for each of the markdown files with a name meeting the specific search parameters, then the `-exec` option runs the sed command with the regular expression created above.

> Aside: note the `find` uses what looks like regex to search for certain files. While similar, this is an example of a "file expansion" search also konwn as "globbing," which approach pattern matching similar to regex but specific to searching file paths (see [Globbing](https://tldp.org/LDP/abs/html/globbingref.html)). To find posts that don't inlude ones posted in August, for example, try the `find` command:
> 
> ```
> find _posts -type f -name '202[012]-[01][01234567]*'
> ```

So to sum up: Using this one line of commands, developed piece by piece, I quickly updated all of the include statements for all of the posts on the site.