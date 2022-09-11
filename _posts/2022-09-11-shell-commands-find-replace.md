---
title: 'Find and replace data in the shell'
date: 2022-09-11
excerpt: 'This post demonstrates command shell tools for finding and replacing data. It uses regex, grep, and sed.'
header:
  teaser: '/assets/images/tape-red-th.png'
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

The following demo was created for a specific use: searching out specific patterns in a series of posts that I had written for this site, and making specific changes depending on the contents of the files. This site runs on a static site generator platform called [Jekyll](http://jekyllrb.com), which reads through a series of plain text files formatted in markdown and generates the html to display each page on this site. Over the summer, I had created a template for introductory text on certain posts (first called `toc_mapping_humanities_data.md`). Jekyll provides a capability that allowed me to include and this boilerplate text so that it appears in any page on the site where I put the code to reference it. I had referenced this in a handful of posts, but I soon realized that in fact "mapping humanities" wasn't really correct; the main point, in fact, was humanities data curation, so I renamed that template `intro_humanities_data_curation.md`. Now, I needed to update each post where I had included the template with the new template title.

It would be possible to make these replacements by hand, but it seemed like it would be quicker if I could ask the computer to do this for me. I also thought that it might be useful to show how to use the command shell to make this update in a quick and consistent way. The basic steps would be:

1. create a way to match the text that I wanted to change
1. identify the files with the text to change
1. look through the files and replace the old text with the new text.

Each of these steps can be done with a command shell tool: the first using a regular expression, the second using [grep](https://www.gnu.org/software/grep/manual/grep.html) and [find](https://linux.die.net/man/1/find), and the third using a file editor like [sed](https://www.gnu.org/software/sed/manual/sed.html), which can use the regular expression to make the changes.

## Match the text to change

Each of the files where I used the introductory template included the following line:

```
{% include toc_mapping_humanities_data.md %}
```

This line told the site publisher (Jekyll) to insert the template at this point. To include the new template, I needed to replace the above with the following line: 

```
{% include intro_humanities_data_curation.md %}
```

To do this, I created the following regular expression, which would match the portion of the line that I wanted to match. Using grouping, I could tell the command which parts I wanted to change.

```regex
(include )[a-z_]*(.md)
```

> Aside: note in the grep I used what looks like regex to search for certain files. While similar, this is an example of a "file expansion" search also konwn as "globbing," which approach pattern matching similar to regex but specific to searching file paths (see [Globbing](https://tldp.org/LDP/abs/html/globbingref.html)). To find posts that don't inlude ones posted in August, for example, try the `find` command: 
> 
> ```
> find _posts -type f -name '202[012]-[01][01234567]*'
> ```

## Identify the files and make the changes

Now, `grep` and `sed` come into the picture. I used `grep` to identify all of the lines where the include command occurs:

```bash
grep 'include [a-z_]*.md' _posts/202[012]-[01]*.md
```

To make the replacement, the `sed` command. `sed` is a "stream editor" designed to replace matches in input based on pattern matching like regular expression.

To test `sed`, for example, we can pipe in a specific input:

```bash
echo "{% include toc_mapping_humanities_data.md %}" | sed -E 's/(include ).*(\.md)/\1intro_humanities_data_curation\2/'
```

This is useful for testing the regular expression patterns, too. The `-E` option specifies using extended regular expressions, which are necessary to use things like group matching. In this case, the parentheses in the first pattern set groups, they are then references with numbers starting with `1` in the replacement string (i.e., `\1` and `\2`). 

So the sed command may be:

```bash
sed -E 's/({% include ).*(.md %})/\1intro_humanities_data_curation\2' _posts/202*.md
```

This will publish all of the modified stream (the file contents) into the terminal window or display. If you review the contents, the change has worked. Now, it needs to be rerouted into the files, if you want to make the actual changes.

## Find the files, make the changes

To do this, we can use the `find` command to make a specific search for the files, then run the sed command on each of these: 

```bash
find _posts -type f -name '202[012]-[01]*.md' -exec sed -i '' -E 's/(include ).*(\.md)/\1intro_humanities_data_curation\2/' {} \;
```

The above illustrates some of the power of the `find` command. Here it searches for each of the markdown files with a name meeting the specific search parameters, then the `-exec` option runs the sed command with the regular expression created above. 

Using this one command, all of the include statements were udpated. 