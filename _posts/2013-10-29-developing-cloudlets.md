---
layout: post
title: "Developing Cloudlets"
description: "A Git workflow for developing cloudlets"
category: "hacking"
tags: [ "cloudlets" ]
---

So, in our attempts to develop cloudlets (packages for [Nepho](https://github.com/huit/nepho)) in some sort of reasonable fashion, we recommend a Git workflow that looks something like this:

1. Fork the [example cloudlet](https://github.com/cloudlets/nepho-example).

2. Rename your fork to something less generic, and check it out.
        
        $ git clone https://github.com/cloudlets/nepho-railsapp.git
        Cloning into 'nepho-railsapp'...
        remote: Counting objects: 78, done.
        remote: Compressing objects: 100% (46/46), done.
        remote: Total 78 (delta 16), reused 74 (delta 12)
        Unpacking objects: 100% (78/78), done.
        Checking connectivity... done

3. Add a `git remote` called "example" that points to the original example.
        
        $ git remote add example https://github.com/cloudlets/nepho-example.git
        $ git remote -v
        example	https://github.com/cloudlets/nepho-example.git (fetch)
        example	https://github.com/cloudlets/nepho-example.git (push)
        origin	https://github.com/cloudlets/nepho-railsapp.git (fetch)
        origin	https://github.com/cloudlets/nepho-railsapp.git (push)

4. Start developing your cloudlet.

5. Periodically pull in updates from the example cloudlet as we make improvements.
        
        $ git pull example master
        From github.com:cloudlets/nepho-example
         * branch            master     -> FETCH_HEAD
        Updating 2bb727d..729e139
        Fast-forward
         README.md            |  26 ++++++++++++
         data/.gitignore      |   2 +-
         hooks/bootstrap      | 138 +++++++++++++++++++++++++++++++++++++-------------------------
         hooks/configure      |  18 +++++---
         hooks/deploy         |  15 ++++++-
         6 files changed, 136 insertions(+), 314 deletions(-)
         create mode 100644 README.md
         mode change 100644 => 100755 hooks/bootstrap
         mode change 100644 => 100755 hooks/configure
         mode change 100644 => 100755 hooks/deploy

If you're concerned about our updates breaking your code, I recommend that instead of pulling directly into your `master` branch, every time you pull you make a topic branch and pull into that first, then test.

Good resources for learning Git include [Pro Git](http://git-scm.com/book) and [Git Ready](http://gitready.com/).

{% include JB/setup %}
