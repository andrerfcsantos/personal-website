---
title: "Git Tricks: Global .gitignore"
date: 2024-10-29T20:10:29Z
draft: false
---

Most git repositories have a `.gitignore` file in it. If you are new to Git, this file controls the files that should be ignored by git in that specific repository. It's useful to prevent accidentally committing unwanted files to the repository.

But do you know that you can have a global `.gitignore` file that can be applied to all your repositories?

This is possible because one of the configurations Git has is `core.excludesFile`. With this option, you can pass a path to a file, and Git wil always use that file when considering files to ignore.

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

The main use of the global gitignore file for me is to have Git ignore those hidden folders IDEs sometimes create. I'm taking about folders like `~/.idea` for Jetbrains IDEs, and `.vscode` for Visual Studio Code. My `~/.gitignore` file looks like this currently:

```gitignore
.idea/
.vscode/
```

I want Git to ignore these folders across all my repositories, so I don't accidentally commit and push my specific IDE configurations to any repository.

One alternative to having these folders in the global gitignore would be to add them to each repo's `.gitignore` file instead. However, if you contribute to several projects, you'll find that not all maintainers want all their `.gitignore` files to support every IDE folder, or every specific file/folder you might want to ignore. Having them ignored globally on all your projects works really well in these scenarios.
