---
title: "Git Tricks: Global .gitignore"
date: 2024-10-29T20:10:29Z
draft: false
---

Most git repositories have a `.gitignore` file in it. If you are new to Git, this file controls the files that should be ignored by git in that specific repository. It's useful to prevent accidentally committing unwanted files to the repository.

But do you know that you can have a global `.gitignore` file that can be applied to all your repositories?

This can be very useful, mainly for two reasons:

1. **Ignore files once for all your projects**
   
   For files that you know you always want to ignore, you can setup git to ignore that file once, instead of updating every `.gitignore` of every project where you want to ignore those files.

2. **Less noise in a project's `.gitignore`**
   
   If every contributor to a project adds a specific directory or file they want to ignore, the `.gitignore` file for the project can become cluttered very quickly. Not only because a lot of things will be added to it, but also because of the number of changes to the `.gitignore` file the maintainers of a project will have to understand and approve. Having this global gitignore ensures that the project's `.gitignore` is as clean as possible, and that you have the files you want to be ignored, even if maintainers don't agree to add it to the project's gitignore file.

Despite the convenience of having a global gitignore, most people I talk to don't know this is something they can do. And some of them have been using git for several years now. In this post my goal is to show how to setup one. 

## How to setup your global gitignore

Setting up a global gitignore is possible because one of the configurations Git has is `core.excludesFile`. With this option, you can pass a path to a file, and Git wil always use that file when considering files to ignore.

Here is how I setup my global gitignore:

1. **Create a file `~/.gitignore`.**

   This file doesn't have to be in the home directory, and it doesn't even need to be called `.gitignore`. But I like to see my home directory as a place for global stuff related with my user. So having global gitignore called `.gitignore` in my home directory makes sense. If you already have a folder for global configs, or a git specific folder, feel free to put it there instead.

2. **Update your `~/.gitconfig`:**
   
   Add the `core.excludesFile` to your `~/.gitconfig` with the path to the file you have created in the previous step:

   
   ```txt
   [core]
	   excludesFile = ~/.gitignore
   ```

This is a configuration I always do when I setup a new computer where I will be using Git extensively. If you want to know more about gitignore read [Git Reference: gitignore](https://git-scm.com/docs/gitignore). It explains in more detail the multiple sources gitignore uses to ignore files and its precedence, the defaults for `core.excludesFile` and other ways to have git ignore files, but not share those ignored files.

## What to put in the global gitignore file?

The main use of the global gitignore file for me is to have Git ignore those hidden folders IDEs and other tools sometimes create. I'm taking about folders like `~/.idea` for Jetbrains IDEs, and `.vscode` for Visual Studio Code. My `~/.gitignore` file looks like this currently:

```gitignore
.idea/
.vscode/
```

The reason I like to keep these directories ignored everywhere is twofold. Firstly, I don't accidentally commit and push my IDE configurations to the repositories I contribute to. Secondly, I understand that I want to ignore these folders because I use those IDEs. But the IDE choice is a matter of personal preference, and not everyone in a project will want to ignore the same files as me. And I understand a project's maintainer to not wanting to have to maintain a list of ignored files for every specific IDE or tool their contributors want to use. 

Other than IDEs, there are other tools that create files in a project's directory, which means that files created by those tools are good candidates to be in your global gitignore file. If the tool is very specific to your needs, it might make sense to make the files that tool creates in your global gitignore. On the other hand, if that tool is something that a lot of people that use the repo are expected to use, it might make more sense to add it to the project's `.gitignore`.

Always use your best judgement when ignoring git files globally as opposed to locally in each project, but now you know you have that choice!

Let me know if you are currently using a global gitignore file, or are considering using one. And if so, what files will you ignore globally?
