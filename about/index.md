---
title: Poirot
class: clean
framework: thinny
highlighter: prettify
hitheme: twitter-bootstrap
article: true
url: {lib: ../libraries}
mode: selfcontained
image: "poirot.jpg"
description: >
  This is my blog, where I shall be posting all sorts of random ramblings, mainly related to data science and R.
---

> __Poirot__ is an elegant, blog aware, static site generator to create, manage and publish websites with reproducible content using [R Markdown](http://www.r-project.org).

---

**Installing Poirot**

Poirot is still under development and will be available on github shortly. You will also need to install [slidify](http://github.com/ramnathv/slidify) and [slidifyLibraries](http://github.com/ramnathv/slidifyLibraries).


Poirot uses **markdown** for text content, **knitr** for code chunks and **mustache** for templating. Poirot is highly extensible using widgets. As a publishing framework for data scientists, Poirot comes pre-equipped with support for math, visualization and tables.

**Reproducing this Blog**

Once you have installed Poirot, you can reproduce this blog by cloning this git repository, running `blogify`, opening a web server and navigating to `http://localhost:8080/posts` on your browser.

```bash
$ git clone git@github.com:ramnathv/poirotBlog
$ cd poirotBlog
$ python -m SimpleHTTPServer 8080
$ open localhost:8080/posts
```

You can generate your own posts by clearing out all files from the `posts` folder (except for `config.yml`), tweaking `site.yml` and `posts/config.yml` and running `blogify("posts")` from the root directory.

**How to contribute**

You can [fork the repository](https://github.com/ramnathv/blogify) on Github.

**Copyright**

All the content in this blog is licensed under [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/). This site is hosted on [GitHub](https://github.com) Pages and uses a modified version of [purus theme](https://github.com/mertemin/purus).

<style>  
  pre {margin-left: 0px;}
</style>
