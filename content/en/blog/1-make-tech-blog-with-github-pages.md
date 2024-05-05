---
author: "Loko"
title: "Make tech blog with Github Pages"
date: 2023-02-11
lastmod: 2023-02-11
description: "Hugo + Github Pages"
tags: ["tech", "hugo", "github"]
thumbnail: /thumbnail/hugo-logo.webp
toc: true
---

## 0. What is Github Pages?

Github Pages is a useful tool that allows us to create our own website for free without a server or DB.  
It is widely used not only for blogs but also for resumes, portfolios, official documents, galleries, and archives.  
(Especially, many developer create portfolio with Github Pages.)  
I also chose Github Pages because I wanted to create a new tech blog!

[Github Pages](https://pages.github.com)

## 1. Why Hugo?

In order to create Github Pages, we need to use **Static Site Generator**, which create static website.  
I checked the [ranking](https://ossinsight.io/collections/static-site-generator/) without hesitation.  
Hugo, which I chose from the ranking table, was ranked 6th as of February 2023, indicating that it is a very popular tool.  
In addition, there were many blog posts in Korean language that show how to deploy Github Pages using Hugo, and it looked really easy and simple.  
As Hugo supported various themes, I thought I just needed to choose a theme and change it to suit my taste.

On the other hand, I thought about using **Next.js**, which ranks second in the Static Site Generator rankings as of February 2023.  
I thought about studying Next.js, which is in the sensational as a front-end framework.  
However, Next.js did not seem to have many themes like Hugo, so it was expected that it would take a considerable amount of time to deploy the website.  
(There seem to be many people who make portfolios using Next.js with Github Pages.)  
I didn't want to spend that much time on making blog, so I chose Hugo.

## 2. Process

### Install a Hugo

To install a Hugo, we need to install [Git](https://git-scm.com/downloads) and [Go](https://go.dev/dl/).  
After installing two from the link above, I installed Hugo using [Homebrew](https://brew.sh/) in Mac OS.

```sh
brew install hugo
```

After installation, it is good to check the version to see if it is installed properly.

```sh
hugo version
```

### Creat two Github Repositories

One is to contain the entire blog source managed by Hugo.  
We can choose the name freely, but many people name it `blog`, so I use the same name.

The other contains files that have been built and will work in the production environment.  
This repository must be named as `<USERNAME>.github.io`.

### Create a directory that manage blog files

We need to create a directory where the blog files will be contained.  
We can simply create it by moving to the desired folder location and entering the command below.

```sh
hugo new site <NAME>
```

\<NAME> is the name of the folder we want to create.  
Many people named it 'blog', and so did I.

### Choose the theme you want

We can choose the theme we want from [Hugo Themes](https://themes.gohugo.io/).  
Click the theme you want and press the **Download** button to go to the Github Repository page, and we can clone the repository from the `blog/themes` directory.  
(If you used a different name in the `hugo new site` command, you should find the themes folder under the folder with that name.)  
And you can modify the `config.toml` file according to the guide organized on the theme page.

### Link with Github Repositories created

The `blog` directory works with the `blog` Github Repository.

```sh
git init
git remote add origin https://github.com/<USERNAME>/blog
```

In the `blog` directory, add the `blog/public` directory as submodule, and link the `<USERNAME>.github.io` repository.

```sh
git submodule add -b master https://github.com/<USERNAME>.github.io public
```

### Write a posting and deploy

Since each theme has a different directory structure in which the posts to be located, you should read the guide well on the theme description page and then write an md file in the correct location.  
After completing the writing content, build it through the command below in the `blog` directory.

```sh
hugo -t <THEME_NAME>
```

The built files will be placed in the `blog/pubic`, and all you need to do is push the code to the `github.io` repository for deployment.  
Push the `blog` repository also for code management.

## 3. TODO

- Make my own logo!
- Categorize posts by creating categories
- Complete the 'About' page
