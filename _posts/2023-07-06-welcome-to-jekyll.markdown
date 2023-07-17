---
layout: post
title:  "How I Created This Blog Using GitHub Pages & Jekyll"
date:   2023-07-06 22:11:30 -0400
---

GitHub Pages' support for Jekyll has been around for a while, for good reason, as it allows for one of the simplest and quickest ways to set up a static website or blog. The seamless integration with GitHub, complimentary hosting, custom domain support, simplicity of Jekyll, and the ease of content creation with Markdown has made the process of creating and maintaining this blog a breeze. By leveraging GitHub's editor and direct commit functionality, you can conveniently manage your Jekyll blog directly on the GitHub platform itself. However, I decided to setup a local environment with Jekyll to understand the entire stack, which is documented below.

## Environment
- Windows Subsystem for Linux (WSL2) with Ubuntu. Installing WSL2 is effortless these days, just open a windows command prompt and issue `wsl --install`. This will install all the WSL2 components and Ubuntu Linux distro. [Install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install){:target="_blank"}

- Visual Studios Code (VSCode) connected to WSL2. This is a great way to get the best of both worlds as VSCode will hook into the WSL2 filesystem allowing you to open projects that live in the Linux subsystem. To enable this feature in VSCode you basically install the extension `Remote Development`. [Working with VSCode on Ubuntu on WSL2](https://ubuntu.com/tutorials/working-with-visual-studio-code-on-ubuntu-on-wsl2#1-overview){:target="_blank"}

- Git. Install within WSL2 on Ubuntu via `sudo apt-get install git`. Your WSL2 installation may already contain git. [Installing Git](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git){:target="_blank"} There are some options to install & use Git for Windows, personally I prefer the command line terminal.

- Ruby & Jekyll. We can install ruby and all prerequisites directly through WSL2. [Jekyll Requirements](https://jekyllrb.com/docs/installation/){:target="_blank"}
    - ruby - `sudo apt-get install ruby-full`
    - rubygems - `sudo apt-get install rubygems`
    - gcc, g++, make - `sudo apt-get install build-essential`
    - jekyll - `gem install jekyll bundler`

## GitHub Pages
The official documentation is an incredible source at this point for initial setup of GitHub pages, and customizing Jekyll themes. [Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart){:target="_blank"}.

1. Create a new repository with an empty README on GitHub and follow the naming convention for the repo name `username.github.io`
2. Locally create a new Jekyll blog `jekyll new myblog`
3. Navigate to the newly created blog directory and clone your GitHub repo via `git clone`. Setup your SSH key for ease of use. Essentially you generate a new key via `ssh-keygen -t ed25519 -C "your_email@example.com"`, then copy the contents of file: `~/.ssh/id_ed25519.pub`, and add it within GitHub on Settings --> SSH and GPG keys. [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account){:target="_blank"}
4. use `git` to `add`, `commit`, and `push` your newly created Jekyll blog
5. After a few minutes your blog should be live at `https://username.github.io`

For a list of dependency versions for the entire GitHub Pages & Jekyll stack, check out [GitHub Pages Dependency Versions](https://pages.github.com/versions/){:target="_blank"}

## Jekyll - Adding Content
To start learning how you can add content to Jekyll, such as static pages or new blog posts, visit the official documentation.
- [Pages](https://jekyllrb.com/docs/pages/){:target="_blank"}
- [Posts](https://jekyllrb.com/docs/posts/){:target="_blank"}
- [Front Matter](https://jekyllrb.com/docs/front-matter/){:target="_blank"}


## Jekyll - Customizing Themes
GitHub pages comes with quite a bit of supported themes to alter the entire look of the blog. This is easily configurable through Jekyll's `_config.yml` `theme: theme-name` property. [GitHub Pages Supported Themes](https://pages.github.com/themes/){:target="_blank"}

In my case I wanted a dark version of the default `minima` theme, and also wanted to customize some `css` myself. Jekyll's `minima` skin can be easily swapped out in `_config.yml` with...
```
minima:
  skin: dark
```
However, this will not work directly with GitHub Pages because of the version of `minima` that GitHub Pages supports by default. What is interesting is that the dependency page [GitHub Pages Dependency Versions](https://pages.github.com/versions/){:target="_blank"} will state `minima 2.5.1` yet unless told to do so by `remote_theme` in `_config.yml` it will generate from an older version which does not support the `skin: dark` option.

Therefore, we will directly target a certain commit id that represents `minima 2.5.1`, which does support different skins. `_config.yml` should now be...
```
remote_theme: jekyll/minima@3cdd14d
minima:
  skin: dark
```
For reference [Jekyll minima 2.5.1 GitHub repo](https://github.com/jekyll/minima/tree/3cdd14dff1216f561c68329e0b7420c2dc9b796a){:target="_blank"}

## Jekyll - Customizing css
We can browse the GitHub repo for this version of `minima` to understand the structure - as overwriting any file in our local repo will take precedence when building. The best way to customize css is to add the `_sass/minima/custom-styles.scss` path and file to our local repo. Now we can use the browser's inspect tools to find html elements or ids to customize our css.

In my case, I decided to increase the width of overall wrapper of the default `minima` theme with...
```
.wrapper {
    max-width: 60%;
}
```