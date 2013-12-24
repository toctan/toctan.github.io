---
layout: post
title: GNU Parallel
tags: bash shell script subshells GNU parallel
---


A while ago, I wrote this little shell script to download YouTube
videos in parallel:

{% gist 5604384 %}

It takes advantage of _subshells_ to do concurrent processing, in
effect downloading multiple videos simultaneously. A _subshell_ is a
child process launched by a parent shell, when a list of commands are
wrapped in _parentheses_, they run as a subshell. `wait` prevents the
next command to be executed until all subshells exit, thus with a
counter it can be used to limit the amount of concurrent tasks.

Now imagine that you have a network authentication client which will
dramatically stop the `network-manager` daemon and somehow it will
never exit until the user hit RET. You might wonder there is no such
thing, it does not make any sense. Well, it's the crappy
`rjsupplicant` under Linux, which is used in most universities in
China for education network authentication. I'd like to add it to the
start up jobs so I don't have to run it every time I login.

```bash
$ sudo rjsupplicant && sudo service network-manager start
```

This will not work because `rjsupplicant` will never exit and the last
half of the command will never get run. I finally get it working with
_subshell_ and a little hack:

```bash
$ (sudo rjsupplicant) &; sleep 5 && sudo service network-manager start
```


## GNU parallel

Now back to our little script, it works great except that the output
is a little messed because the three _subshells_ will tried to write
to STDOUT simultaneously. But I don't care as long as the videos get
downloaded. Now what if we have a text file contain a long list of
hosts and we want to use `ping` to find the fastest one in the
shortest time. This time the output order is important, _subshells_
will mess up the output so what can we do? [GNU parallel][] to the
rescue.

GNU parallel is not pre installed in most systems, so your have to
install it first:

```bash
# OS X with homebrew
$ brew install parallel

# Arch Linux
$ sudo pacman -S parallel

# *nix
$ (wget -O - pi.dk/3 || curl pi.dk/3/) | bash
```
_parallel_ is used to build and execute shell command in parallel
locally or remotely. With parallel, the previous youtube script can
rewritten into a one-liner:

```bash
$ cat videos.txt | parallel -j3 youtube-dl -o "%(title)s.%(ext)s"
```

The amazing thing about parallel is that __it does not only run the
jobs in parallel but also preserve the output of each job so that they
will not interfere with each other__. So our parallel pings can be
achieved easily with:

```bash
$ cat hosts.txt | parallel -k -j10 ping -c10
```

Normally, the output order of `parallel` is not granted due to its
parallel processing, with the option `-k`, the sequence of output
will be the same as the input.

### A crash course

The input can be read from the command line, STDIN, a file or all of
the three at the same time.

```bash
# use ::: to specify arguments from the command line
$ parallel echo ::: A B C

# use :::: or -a to specify input from a file
$ parallel -a hosts.txt ping -c10
$ parallel ping -c10 :::: hosts.txt

# from STDIN
$ cat hosts.txt | parallel ping -c10

# all three
$ cat hosts1.txt | parallel ping -c10 ::: A B C -a hosts.txt
```

If no command is given, the arguments themselves are treated as
commands:

```bash
$ parallel ::: date uptime whoami
# Mon Dec  9 23:36:42 CST 2013
# 23:36  up 6 days,  7:47, 2 users, load averages: 1.45 1.21 1.19
# toctan
```

Parallel will normally treat a full line as a single argument, this
can be changed with `-d` to specify the delimiter:

```bash
$ echo 'A,B,C' | parallel -d, echo
```

To skip empty lines use `--no-run-if-empty` option:

```bash
$ cat hosts.txt | parallel --no-run-if-empty ping -c10
```

If no _replacement strings_ are specified, the default is `{}`, which
refers to the original argument.

```bash
$ parallel echo {} ::: /path/to/README.md
# /path/to/README.md

# to remove the extension:
$ parallel echo {.} ::: /path/to/README.md
# /path/to/README

# to remove the path:
$ parallel echo {/} ::: /path/to/README.md
# README.md

# to keep only the path:
$ parallel echo {//} ::: /path/to/README.md
# /path/to/

# to remove path and extension at the same time:
$ parallel echo {/.} ::: /path/to/README.md
# README
```

## Some useful examples

```bash
# to compress all pdf files using gzip:
$ parallel gzip ::: *.pdf

# generate a thumbnail for all the jpg files in current directory:
$ ls *.jpg | parallel convert -geometry 120 {} {.}_thumb.jpg

# repeat the same command 10 times
$ seq 10 | parallel -n0 ping -c1 google.com

# parallelizing rsync
$ ls src/ | parallel -v ssh server rsync -Havessh {} server:/deist/{}
```

## What next?

To learn more about Parallel, such as running jobs in parallel
remotely, checkout the official [manual][], [tutorial][] and
[youtube videos][] by the author.

[GNU parallel]: http://www.gnu.org/software/parallel/
[tutorial]: http://www.gnu.org/software/parallel/parallel_tutorial.html
[manual]: http://www.gnu.org/software/parallel/man.html
[youtube videos]: http://www.gnu.org/software/parallel/man.html
