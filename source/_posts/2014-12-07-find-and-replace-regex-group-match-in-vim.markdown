---
layout: post
title: "Find and replace regex group match in VIM"
date: 2014-12-07 01:57
comments: true
categories:
  - vim
  - regex
---

# Intro
Regular expression is a special text string for describing a search pattern and since VIM is a full-fledged editor you can use a regex string to find what you're looking for in a file.

If you are a VIM user you might already know that, same goes here! Until few days back I needed to do more than a usual search in my file.

I received a file with this content:

{% codeblock %}
1-2-13,3700,26840,0,mt,0,EUR,0,USD
10-21-13,3700,26807,0,mt,0,EUR,0,USD
...
{% endcodeblock %}

And I had to import into a table in database. I found a way to import CSV file to the database but after I imported all the records I noticed that the date column was wrong in the database. As you can see in the first column, date format isn't what usually databases accept, the format should be like '2013-1-2'. Well I had two ways to fix the problem, either write a small script to fix the date or use VIM's search and replace feature to fix the problem.

<!-- more -->

## Search a Regex string in VIM
The regex was simple it can be used in VIM's search like this:

{% codeblock %}
\d\{1,2\}-\d\{1,2\}-\d\d
{% endcodeblock %}

> In order to search in VIM, you need to go to Normal mode(press escape) and press / and start typing to find your match

The month and day length might be one or two, that's we why have \d\\{1,2\\} to make sure it matches with both cases. The first surprise that I had was this weird backslash before curly brackets. If you are familiar with regex, you don't need to escape the curlys!


## Replace matched Regex in VIM
So far we managed to match the date string but how can we reuse our matched string and change the order? The answer is Grouping feature in regex. All we need to do is put the part that we want to group in parentheses and then use it. What does it mean?! Let's try it out:

{% codeblock %}
\(\d\{1,2\}\)-\(\d\{1,2\}\)-\(\d\d\)
{% endcodeblock %}

This thing might be bit scary but if you take look closer, I just added parentheses around month, day and year regex.

To reuses the matched string and replace it with the correct order you can use VIM's replace feature:

{% codeblock %}
%s//21\3-\1-\2/g
{% endcodeblock %}

> In order to replace a string in VIM, you need to go to Normal mode(press escape) and press : and start typing the above command

By this replace command we replace the date with 21(third match group)-(first match group)-(second match group).
