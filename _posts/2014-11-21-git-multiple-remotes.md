---
layout: post
title:  "Using git with multiple remote locations"
date:   2014-11-21 16:05:00
categories: 
---

Sometimes I use git to push code from a single project into multiple remote repositories. 

Git can be setup to use multiple equivalent authoritative upstreams, thus allowing both push and pull between multiple remote locations. This can be handy if you want to have some projects in both a private repository and a public one like GitHub. I don't like having to push changes to each remote individually, so I setup the following config that allows sending updates with a single push.

First, I did `git remote add` for each of my repositories. The `deploy` repository is my private one that receives updates for this website and runs an post-receive hook that deploys content to my web server. The `github` repository a my public copy.

So I have a git config such as this:

    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
    [remote "deploy"]
        url = deployer@cornellio.net:Sites/blog.git
        fetch = +refs/heads/*:refs/remotes/deploy/*
    [remote "github"]
        url = https://github.com:/Cornellio/jekyll-blog.git
        fetch = +refs/heads/*:refs/remotes/github/*

To create a merged‐remote I did a `git config -e` and added this combination:

    [remote "all"]
        url = deployer@cornellio.net:Sites/blog.git
        url = https://github.com:/Cornellio/jekyll-blog.git

Now I can do `git push all master` and update both repositories with a single command.