---
layout: post
title: "Making a Title Column in CSS"
date: 2012-12-09 14:23
comments: true
categories: 
---

So, if I *were* to actually recode my blog, I'd need to decide on a layout that's actually manageable.  The current design is a modification on top of the default Octopress theme which involves much CSS and Javascript wizardry beyond my comprehension.

One possible direction is to go for vast simplification and build the styling from scratch.  I styled my own CV with inspiration from <a href="http://kieranhealy.org/vita.pdf">the CV of Kieran Healy</a> and have liked the idea of pushing titles into the left column.  I'd actually like to push other content like photos into the left column as well so that it's kind of an alternate universe for items other than body text.  Here's the working mock-up I've put together for the web:

<iframe style="width: 100%; height: 300px" src="http://jsfiddle.net/jklukas/DvrTb/2/embedded/result,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
It keeps things in sync with the text.  I'm working to keep the vertical spacing in rhythm, so that the items in the left column fall on the same line boundaries as the main text.  The line height for the title is 1.5 times that for the body text, so the two lines of the title equal three lines of text.  I'm not sure I've got the image positioned correctly yet.  

Is this a reasonable concept to start from?
