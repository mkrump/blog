---
title:  Things I learned on my pairing tour
date:   2017-09-08 08:30:00 -0500
categories: 
  - apprenticeship
---

I just finished my first week of the pairing tour. Initially, I wasn't exactly
sure what to expect. It seemed like it would be difficult to contribute to a project
that you didn't regularly work on, especially if the team was using a language
or framework you weren't familiar with. However, overall it's been a really good
experience. One of the things that I've enjoyed is getting a chance to see to
see other developers' workflows. By watching others work, I learned a ton of
neat little productively hacks. I thought I'd write a blog post to document some
of the things that I learned, so that I don't forget and can hopefully
integrate into my own workflow.

### The Silver Searcher (ag) 

I like the JetBrains IDEs (specifically PyCharm). The various search
features like `Cmd-F` (fuzzy search in path), and `Shift-Shift` (fuzzy search
everywhere), are excellent and I use these all of the time. But sometimes you're
not working with an IDE and just want to quickly find where that darn `print`
statement that you accidentally left behind is. In this case the [Silver
Searcher](https://github.com/ggreer/the_silver_searcher) or `ag` is what you are
looking for. I saw multiple people on my pairing tour using `ag` to quickly
search through large codebases. 

{{<figure src="/assets/2017-09-08-ag.gif" class="center-figure-image">}}

### Sorting with vim

Most of the people that I paired with either used vim as their editor or used the
vim key bindings within their IDE. One neat trick I saw multiple people use was
vim's [sort lines](http://vim.wikia.com/wiki/Sort_lines) feature. This provides
a nice way to order things like multi-line arguments. Also, it can sort dates,
remove duplicates, and several other data types.

{{<figure src="/assets/2017-09-08-sort.gif" class="center-figure-image">}}

### Search shell history
In the past when I needed to retrieve a previous shell command, I just paged
through my history using the up/down arrows, or very occasionally the `history`
command. Those days are definitely over. While pairing I learned that both bash
and zsh support searching your command history. To use the shell's search, simply hit
`Ctrl-R` and start typing. Matches are retrieved in reverse order.
 
 
{{<figure src="/assets/2017-09-08-search.gif" class="center-figure-image">}}

### Git interactive staging

As with everything git there are a million options here, but this came up for me
when I had accidentally added whitespace to multiple spots in a file that I
wanted to commit. Normally, I just stage the entire file, but in this case I
didn't want to include the whitespace related changes. Interactive staging via
`git add -p` gives you very fine-grained control as to exactly what changes
within a file will be staged. 

{{<figure src="/assets/2017-09-08-git-add-p.gif" class="center-figure-image">}}

