---
title: The Ultimate Git Log Format
date: 2011-01-17 15:00:00 -0700
---

I was recently working my way through [Git Immersion](http://gitimmersion.com/index.html), a great guided tour of git.  We use git almost exclusively at work and I'm always looking for tricks to improve my git workflow.  One of the first ones I found was [the Ultimate Log Format](http://gitimmersion.com/lab_10.html) which is a custom log format output that lets you quickly see the latest commits.

```
git log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short
```

![Custom git log](/assets/images/git-hist.png)

For comparison, here is the standard `git log`:

![Normal git log](/assets/images/git-log.png)

As the guide says, typing/memorizing that command will be a little difficult, so put it in your [.gitconfig](https://github.com/tjwallace/dotfiles/blob/master/gitconfig.erb#L12) as an alias to make your life easier.

```
[alias]
    hist = log --pretty=format:\"%h | %ad | %s%Cred%d%Creset %Cgreen[%an]%Creset\" --graph --date=short
```
