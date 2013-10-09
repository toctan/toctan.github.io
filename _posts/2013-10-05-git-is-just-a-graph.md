---
layout: post
title: Git is just a graph
tags: git graph
---

Get started with git is quite easy, but after several months with git,
I was still not comfortable with it. _I'm not sure what will happen
when I issue the next command. I google I stackoverflow before every
danger action, maybe it will blow up the next minute._

Several days ago, I came across the amazing
[Think Like (a) Git](http://think-like-a-git.net/) by
[Sam Livingston-Gray](https://twitter.com/geeksam). It gives a really
**intuitive** overview with git:

> Git repo is just one giant graph, commits are nodes of the graph,
> and branches, tags are just labels attached to one of the nodes.

![Directed graph](http://think-like-a-git.net/assets/images2/reachability-example.png)

Technically speaking, it's a directed acyclic graph, every node or
commit has at least one parent(except the initial commit) and any
number of childs. `commit` just adds a new child node of the current
one, `branch` just attaches a new label, `reset` just attaches the
label to another node(with some side effects).

With this in mind, I headed over to the awesome
[Visual git guide](marklodato.github.io/visual-git-guide/) by
[Mark Lodato](http://marklodato.github.io/). Everything started to
make more sense. I knew what's going on with every command, I seemed
to undersand git.

Wow, I am comfortable with git.

## Some Resources

[Try git](http://try.github.io) and
[Git real](https://www.codeschool.com/courses/git-real)
to get started, then
[Think Like (a) Git](http://think-like-a-git.net/) and
[Visual git guide](marklodato.github.io/visual-git-guide/).

Don't just read it, think about it, clone a dummy repo, experiment
with it. Predict how the graph will change before every command, use a
visualizer like [GitX](http://gitx.frim.nl) to verify the result.

By the way, [Git Internals
PDF](https://github.com/pluralsight/git-internals-pdf/releases)
has just been open sourced if you want to learn more about git
_"than any sane human should ever know."_
