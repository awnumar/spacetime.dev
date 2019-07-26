---
layout: post
title: working with forks in git
---

Working with forks entails having a single codebase but being able to push and pull from multiple remote endpoints. This can get fairly annoying, especially for Go projects which have a specified import path which must be reflected in the filesystem path (at least before [modules](https://github.com/golang/go/wiki/Modules)).

Luckily git handles this use-case pretty well.

```shell
$ # Working with a hypothetical project hosted on GitHub.
$ cd $GOPATH/src/github.com
$ 
$ # Create a directory for the source owner, to preserve import paths.
$ mkdir source && cd source
$ 
$ # Clone your own fork into this directory.
$ git clone git@github.com:me/repo.git && cd repo
$ 
$ # Add the original repository URL as a new remote.
$ git remote add upstream git@github.com:source/repo.git
$ 
$ git pull upstream master # Pull from original repository
$ git push origin master   # Push to forked repository
```
